## unit 2: a stateful service

This unit uses `elasticsearch` as base technology to showcase how to develop a stateful framework.

### [params.yaml](operator/params.yaml)

The sample has one configuration parameter named `NODE_COUNT`, the number of nodes in the elasticsearch cluster, it defaults to `3`.

```yaml
NODE_COUNT:
  default: "3"
```

### [templates](operator/templates)

The sample has two resource templates, one is of type `Service` the other of type `StatefulSet`.

StatefulSets currently require a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods.

The following shows the resource template for the `Headless Service`.

[service.yaml](operator/service.yaml)
```yaml
kind: Service
apiVersion: v1
metadata:
  name: hs
  namespace: {{ .Namespace }}
spec:
  selector:
    app: elastic
  ports:
    - protocol: TCP
      port: 9200
  clusterIP: None
    clusterIP: None
```

A [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) is used to manage a stateful service. It manages the deployment and scaling of a set of Pods , and provides guarantees about the ordering and uniqueness of these Pods. StatefulSet manages Pods that are based on an identical container spec, and maintains a sticky identity for each of the Pods. These pods are created from the same spec, but are not interchangeable. Each has a persistent identifier that it maintains across any rescheduling. A StatefulSet operates under the same pattern as any other Kubernetes Controller. You define your desired state in a StatefulSet object, and the StatefulSet controller makes any necessary updates to get there from the current state.

The following shows the resource template for the `StaefulSet`.

[node.yaml](operator/node.yaml)
```yaml

node.yaml: |
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: node
  namespace: {{ .Namespace }}
spec:
  selector:
    matchLabels:
      app: elastic # has to match .spec.template.metadata.labels
  serviceName: {{ .Name }}-hs
  replicas: {{ .Params.NODE_COUNT }}
  template:
    metadata:
      labels:
        app: elastic # has to match .spec.selector.matchLabels
    spec:
      initContainers:
        - name: init-sysctl
          image: busybox
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
          securityContext:
            privileged: true
        - name: volume-permissions
          image: busybox
          command: ['sh', '-c', 'chown -R 1000:1000 /usr/share/elasticsearch/data']
          volumeMounts:
            - name: data
              mountPath: /usr/share/elasticsearch/data
      terminationGracePeriodSeconds: 10
      containers:
        - name: elastic
          image: elasticsearch:7.0.0
          ports:
            - containerPort: 9200
              name: api
            - containerPort: 9300
              name:
          env:
            - name: cluster.name
              value: {{ .Name }}-cluster
            - name: discovery.seed_hosts
              value: {{ .Name }}-node-0.{{ .Name }}-hs,{{ .Name }}-node-1.{{ .Name }}-hs,{{ .Name }}-node-2.{{ .Name }}-hs
            - name: cluster.initial_master_nodes
              value: {{ .Name }}-node-0,{{ .Name }}-node-1,{{ .Name }}-node-2
          volumeMounts:
            - name: data
              mountPath: /usr/share/elasticsearch/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
```

The StatefulSet has a `template` section that holds the specification of each pod in the set. It also has a `volumeClaimTemplate` for a persistent volume claim, this for a volume required by each pod in the set.

The `template` section has a `spec` section which has the `initContainers` and `containers` specification of the pod.

`containers` shows how in our sample the elasticsearch base technology is used. The container configuration is done via environment variables (alternatively could also be done via a config map as shown in unit 1). It also shows where the claimed volume is to be mounted in the container.

`initContainers` come handy if the base technology requires some goofy stuff, like elasticsearch needs the `vm.max_map_count` setting, and also allow access to the mounted volume by the elasticsearch user that the container runs under (UID=1000, AND GID=1000).

### [operator.yaml](operator/operator.yaml)

#### tasks

The sample `deploy-task` applies the service and node template.

```yaml
tasks:
  deploy-task:
    resources:
      - service.yaml
      - node.yaml
```

#### plans

The sample has a `deploy` plan with a `deploy-phase` and a `deploy-step`. From the `deploy-step` the `deploy-task` is referenced. This task gets executed when an instance is created using the operator.

```yaml
plans:
  deploy:
    strategy: serial
    phases:
      - name: deploy-phase
        strategy: parallel
        steps:
          - name: deploy-step
            tasks:
              - deploy-task
```


### Run the framework instance

If you haven't already then clone the `kudo-tutorial` repository.

```
git clone https://github.com/realmbgl/kudo-tutorial.git
```

From the `unit2/operator` folder use the following command to install the operator and create an instance

```
kubectl kudo install . --instance myes
```

Once the install is finished we should see the following pods.
```
kubectl get pods

NAME          READY   STATUS    RESTARTS   AGE
myes-node-0   1/1     Running   0          121m
myes-node-1   1/1     Running   0          121m
myes-node-2   1/1     Running   0          121m
```

Lets check on the elasticsearch clusters health.

Exec into one of the POD's.

```
kubectl exec -ti myes-node-2 bash
```

Use the following curl command to check the health of the cluster.

```
curl myes-node-0.myes-hs:9200/_cluster/health?pretty
```

You should see the following output, showing the cluster status `green`.

```
{
  "cluster_name" : "myes-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```


### Update the framework instance to scale to 4 nodes

Lets increase the `NODE_COUNT` to `4` using the following command.

```
kubectl kudo update --instance myes -p NODE_COUNT=4
```

Once the update is finished we should see an additional pod `myes-node-3`.

```
kubectl get pods

NAME          READY   STATUS    RESTARTS   AGE
myes-node-0   1/1     Running   0          128m
myes-node-1   1/1     Running   0          128m
myes-node-2   1/1     Running   0          127m
myes-node-3   1/1     Running   0          29s
```

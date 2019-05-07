## unit 2: a stateful service

This unit uses `elasticsearch` as base technology to showcase how to develop a stateful service.

In this unit we use separate YAML files for framework type, implementation, and instance. The files are the following.

* [elastic-type.yaml](elastic-type.yaml) - the framework type
* [elastic-impl.yaml](elastic-impl.yaml) - the framework implementation
* [elastic.yaml](elastic.yaml) - the framework instance


### framework implementation

#### templates

The sample has two resource templates, one is of type `Service` the other of type `StatefulSet`.

StatefulSets currently require a [Headless Service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to be responsible for the network identity of the Pods.

The following shows the resource template for the `Headless Service`.

```yaml
service.yaml: |
  kind: Service
  apiVersion: v1
  metadata:
    name: hs
    namespace: {{NAMESPACE}}
  spec:
    selector:
      app: elastic
    ports:
      - protocol: TCP
        port: 9200
    clusterIP: None
```

A [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) is used to manage a stateful service. It manages the deployment and scaling of a set of Pods , and provides guarantees about the ordering and uniqueness of these Pods. StatefulSet manages Pods that are based on an identical container spec, and maintains a sticky identity for each of the Pods. These pods are created from the same spec, but are not interchangeable. Each has a persistent identifier that it maintains across any rescheduling. A StatefulSet operates under the same pattern as any other Kubernetes Controller. You define your desired state in a StatefulSet object, and the StatefulSet controller makes any necessary updates to get there from the current state.

The following shows the resource template for the `StaefulSet`.

```yaml

node.yaml: |
  kind: StatefulSet
  apiVersion: apps/v1
  metadata:
    name: node
    namespace: {{NAMESPACE}}
  spec:
    selector:
      matchLabels:
        app: elastic # has to match .spec.template.metadata.labels
    serviceName: {{NAME}}-hs
    replicas: {{NODE_COUNT}}
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
                value: {{NAME}}-cluster
              - name: discovery.seed_hosts
                value: {{NAME}}-node-0.{{NAME}}-hs,{{NAME}}-node-1.{{NAME}}-hs,{{NAME}}-node-2.{{NAME}}-hs
              - name: cluster.initial_master_nodes
                value: {{NAME}}-node-0,{{NAME}}-node-1,{{NAME}}-node-2
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
```

The StatefulSet has a `template` section that holds the specification of each pod in the set. It also has a `volumeClaimTemplate` for a persistent volume claim, this for a volume required by each pod in the set.

The `template` section has a `spec` section which has the `initContainers` and `containers` specification of the pod.

`containers` shows how in our sample the elasticsearch base technology is used. The container configuration is done via environment variables (alternatively could also be done via a config map). It also shows where the claimed volume is to be mounted in the container.

`initContainers` come handy if the base technology requires goofy stuff, like elasticsearch needs the `vm.max_map_count` setting, and also allow access to the mounted volume by the elasticsearch user that the container runs under (UID=1000, AND GID=1000).

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

The sample has a `deploy` plan with a `deploy-phase` and a `deploy-step`. From the `deploy-step` the `deploy-task` is referenced. This task gets executed when a framework instance is applied.

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


### framework instance

The instance name is specified in the metadata section.
```yaml
metadata:
  name: myes
```

The instance spec references the framework implementation to be used. The sample sets the `NODE_COUNT` parameter to the value `3`.
```yaml
spec:
  frameworkVersion:
    name: elastic-v1
    namespace: default
  parameters:
    NODE_COUNT: "3"
```


### Run the framework instance

If you haven't already then clone the `kudo-tutorial` repository.

```
git clone https://github.com/realmbgl/kudo-tutorial.git
```

From the `unit2` folder use the following command to run the instance.

```
kubectl apply -f .
```

Once the install finished we should see the following pods.
```
kubectl get pods

NAME          READY   STATUS    RESTARTS   AGE
myes-node-0   1/1     Running   0          121m
myes-node-1   1/1     Running   0          121m
myes-node-2   1/1     Running   0          121m
```


### Update the framework instance to scale to 4 nodes

Update the `NODE_COUNT` parameter in `elastic.yaml` to `4`.

```
parameters:
  NODE_COUNT: "4"
```

From the `unit2` folder use the following command to update the instance.

```
kubectl apply -f elastic.yaml
```

Once the update finished we should see an additional pod `myes-node-3`.

```
kubectl get pods

NAME          READY   STATUS    RESTARTS   AGE
myes-node-0   1/1     Running   0          128m
myes-node-1   1/1     Running   0          128m
myes-node-2   1/1     Running   0          127m
myes-node-3   1/1     Running   0          29s
```
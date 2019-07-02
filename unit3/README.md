## unit 3: update and upgrade plans

This unit builds on `unit 2` showcasing how `update`, and `upgrade` plans get triggered. It also showcases how `controlled parameter update` can be triggered.


### [params.yaml](operator/params.yaml)

The sample has an additional `TEST` parameter, to showcase `controlled parameter update`. The `trigger` key of the `TEST` parameter points to the `super` plan to be executed on update.

```yaml
NODE_COUNT:
  default: "3"
TEST:
  default: "0"
  trigger: super
```

### [templates](operator/templates)

No changes here.

### [templates](operator/templates)

#### tasks

Besides the `deploy-task` there is now also an `update-task`, `upgrade-task`, and `super-task`. At this point they all apply the same templates. This will change on further iteration, for example the `upgrade-task` will take care of the [elasticsearch rolling upgrade](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html) steps.

```yaml
deploy-task:
  resources:
    - service.yaml
    - node.yaml
update-task:
  resources:
    - service.yaml
    - node.yaml
upgrade-task:
  resources:
    - service.yaml
    - node.yaml
super-task:
  resources:
    - service.yaml
    - node.yaml
```

#### plans

The sample has not only the `deploy` plan, but also `update`, `upgrade`, and a `super` plans.

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
  update:
    strategy: serial
    phases:
      - name: update-phase
        strategy: parallel
        steps:
          - name: update-step
            tasks:
              - update-task
  upgrade:
    strategy: serial
    phases:
      - name: upgrade-phase
        strategy: parallel
        steps:
          - name: upgrade-step
            tasks:
              - upgrade-task
  super:
    strategy: serial
    phases:
      - name: super-phase
        strategy: parallel
        steps:
          - name: super-step
            tasks:
              - super-task
```


### Run the framework instance

If you haven't already then clone the `kudo-tutorial` repository.

```
git clone https://github.com/realmbgl/kudo-tutorial.git
```

From the `unit3/operator` folder use the following command to run the instance.

```
kubectl kudo install . --instance myes
```

Once the install is finished we should see the following pods.
```
kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
myes-node-0   1/1     Running   0          42m
myes-node-1   1/1     Running   0          41m
myes-node-2   1/1     Running   0          41m
```

Lets check on the plan execution history using the kudo cli, we see the `deploy plan` has been executed.

```
kubectl kudo plan history --instance=myes
History of all plan-executions for instance "myes" in namespace "default":
.
└── myes-deploy-206753060 (created 4h18m37s ago)
```


### Update the framework instance to scale to 4 nodes

Update the `NODE_COUNT` parameter in `elastic.yaml` to `4`.

```
parameters:
  NODE_COUNT: "4"
```

From the `unit3` folder use the following command to update the instance.

```
kubectl apply -f elastic.yaml
```

Once the update is finished we should see an additional pod `myes-node-3`.

```
kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
myes-node-0   1/1     Running   0          46m
myes-node-1   1/1     Running   0          46m
myes-node-2   1/1     Running   0          45m
myes-node-3   1/1     Running   0          70s

```

Lets check on the plan execution history using the kudo cli, we see the `update plan` has been executed.

```
kubectl kudo plan history --instance=myes
History of all plan-executions for instance "myes" in namespace "default":
.
├── myes-deploy-206753060 (created 4h20m36s ago)
└── myes-update-207626186 (created 39s ago)
```


### Upgrade the framework instance to use a newer framework implementation

In the `elastic.yaml` upgrade the frameworkVersion name to `elastic-v2`. v2 uses a newer elasticsearch docker image.

```
spec:
  frameworkVersion:
    name: elastic-v2
    namespace: default
```

From the `unit3` folder use the following command to update the instance.

```
kubectl apply -f elastic.yaml
```

Lets check on the plan execution history using the kudo cli, we see the `upgrade plan` has been executed.

```
kubectl kudo plan history --instance=myes
History of all plan-executions for instance "myes" in namespace "default":
.
├── myes-deploy-206753060 (created 4h22m0s ago)
├── myes-update-207626186 (created 2m3s ago)
└── myes-upgrade-521226614 (created 17s ago)
```

Note: Currently not working [issue 208](https://github.com/kudobuilder/kudo/issues/208).


### Controlled parameter update

Update the `TEST` parameter in `elastic.yaml` to like `100`.

```
parameters:
  NODE_COUNT: "4"
  TEST: "100"
```

From the `unit3` folder use the following command to update the instance.

```
kubectl apply -f elastic.yaml
```

Lets check on the plan execution history using the kudo cli, we see the `super plan` has been executed.

```
kubectl kudo plan history --instance=myes
History of all plan-executions for instance "myes" in namespace "default":
.
├── myes-deploy-206753060 (created 4h23m46s ago)
├── myes-super-133145198 (created 4s ago)
├── myes-update-207626186 (created 3m49s ago)
└── myes-upgrade-521226614 (created 2m3s ago)
```

## unit 1: parameters, templates, tasks, and plans in action

This unit uses the sample [myservice.yaml](myservice.yaml) to showcase the key framework implementation concepts parameters, templates, tasks, and plans.

### framework implementation

#### parameters

Parameters allow for the configuration of you framework implementation on instantiation.

The sample has one configuration parameter named `WHO` with a default value of `Ravi`.

```yaml
parameters:
  - name: WHO
    default: "Ravi"
```

Besides the keys shown in the sample a parameter can have the following additional keys `description`, `displayName`, and `required`.


#### templates

Templates define the resources that can be applied by this framework implementation.

The sample has two resource templates, one is of type `ConfigMap` the other of type `POD`. The pod runs an nginx container into which the config map which holds an index.html file is mounted.

In the config map you see how parameters get templated in, here the parameter `{{WHO}}`. There is also `{{NAMESPACE}}` and `{{NAME}}` which get provided by the framework instantiation.


```yaml
templates:

  config.yaml: |
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: config
      namespace: {{NAMESPACE}}
    data:
      index.html: |
        {{WHO}}, hello from {{NAME}} !!!

  pod.yaml: |
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod
      namespace: {{NAMESPACE}}
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html
      volumes:
        - name: config
          configMap:
            name: {{NAME}}-config
```

#### tasks

Tasks list the resource templates that get applied together.

The sample `deploy-task` applies the config and pod template.

```yaml
tasks:
  deploy-task:
    resources:
      - config.yaml
      - pod.yaml
```

#### plans

Plans orchestrate tasks through phases and steps.

`Plans` consists of one or more `phases`. `Phases` consists of one or more `steps`. `Steps` contain one or more `tasks`. Both phases and also steps can be configured with an execution `strategy`, either serial or parallel.

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

The framework instance defines a configured instance of the framework implementation.

The instance name is specified in the metadata section.
```yaml
metadata:
  name: myservice
```

The instance spec references the framework implementation to be used. It is here where you can also provide parameter values. The sample sets the `WHO` parameter to the value `Matt`.
```yaml
spec:
  frameworkVersion:
    name: myservice-impl-v1
    namespace: default
  parameters:
    WHO: "Matt"
```

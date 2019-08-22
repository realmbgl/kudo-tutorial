## unit 1: parameters, templates, tasks, and plans in action

This unit uses an [operator sample](operator) to showcase the key kudo operator concepts parameters, templates, tasks, and plans.

### [params.yaml](operator/params.yaml)

Parameters allow for the configuration of instances created by the operator

The sample has one configuration parameter named `WHO` with a default value of `Ravi`.

```yaml
WHO:
  default: "Ravi"
```

Besides the keys shown in the sample a parameter can have the following additional keys `description`, `displayName`, `required`, and `trigger`.

### [templates](operator/templates)

Templates define the resources that can be applied by this operator.

The sample has two resource templates, one is of type `ConfigMap` the other of type `POD`. The pod runs an nginx container into which the config map which holds an index.html file is mounted.

In the config map you see how parameters get templated in, here the parameter `{{ .Params.WHO }}`. There is also `{{ .NAMESPACE }}` and `{{ .NAME }}` which get provided by the operator on instantiation.

[config.yaml](operator/config.yaml)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
  namespace: {{ .Namespace }}
data:
  index.html: |
    {{ .Params.WHO }}, hello from {{ .Name }} !!!
```

[pod.yaml](operator/pod.yaml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod
  namespace: {{ .Namespace }}
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
        name: {{ .Name }}-config
```

### [operator.yaml](operator/operator.yaml)

#### general

In the general section the `name` of the operator and its `version` is defined. It also lists the `kudo version` the operator works with.

```
name: "myservice"
version: "0.1.0"
kudoVersion: 0.5.0
```

#### tasks

Tasks list the resource templates that get applied together.

The sample `deploy-task` applies the `config` and `pod` template.

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

From the `unit1/operator` folder use the following command to install the operator and create an instance.

```
kubectl kudo install . --instance myservice --parameter WHO=Matt
```

Next enable localhost access to the instance.

```
kubectl port-forward myservice-pod 8080:80
```

[Click](http://localhost:8080/) to access the instance from your browser.

You should see the following

```
Matt, hello from myservice !!!
```

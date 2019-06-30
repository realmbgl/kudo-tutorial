# kudo tutorial

<img src="https://kudo.dev/images/kudo_horizontal_color@2x.png" srcset="https://kudo.dev/images/kudo_horizontal_color@2x.png 2x" width="256">

Developing `Kubernetes` operators using `KUDO`, the Kubernetes Universal Declarative Operator. KUDO provides a declarative approach to building production-grade Kubernetes Operators covering the entire application lifecycle.

## install kudo

Run the following three commands to `install kudo`.
```
kubectl create -f https://raw.githubusercontent.com/kudobuilder/kudo/v0.3.0/docs/deployment/00-prereqs.yaml
kubectl create -f https://raw.githubusercontent.com/kudobuilder/kudo/v0.3.0/docs/deployment/10-crds.yaml
kubectl create -f https://raw.githubusercontent.com/kudobuilder/kudo/99e0fe7980567ba94503f6385cd54dc5e7e6e90c/docs/deployment/20-deployment.yaml
```

More on the kudo install at [kudo.dev](https://kudo.dev/docs/getting-started/) .


Kudo comes with its own `kubectl CLI plugin`. On `Mac OS X` you can install it using `brew`.

```
brew tap kudobuilder/tap
brew install kudo-cli
```

More on the kudo CLI install at [kudo.dev](https://kudo.dev/docs/cli/) .


## install and operate kudo operators

Install and operate your first kudo operator using [kafka](https://kudo.dev/docs/examples/apache-kafka/) as the sample.


## develop kudo operators

### kudo operator programming model

As a developer of a kudo operator you have to author three types of `YAML` artifacts in the following file folder structure.

```
operator.yaml
params.yaml
templates/
     <template>.yaml
     ...
```

In the following we describe the roles that each of the YAML artifacts plays and what they capture.

#### operator.yaml
The operator file defines the operator behavior.

It has a `name` (e.g kafka), and a `version`.

`Tasks` list the resource templates that get applied together.

`Plans` orchestrate tasks through phases and steps. `Plans` consists of one or more `phases`. `Phases` consists of one or more `steps`. `Steps` contain one or more `tasks`. Both phases and also steps can be configured with an `execution strategy`, either `serial` or `parallel`.

#### params.yaml
The params file defines the parameters that can be used to configure an instance created by the operator. A parameter definition has a `name`, `default value`, `display name`, and `description`.

#### template.yaml
A template file defines the resources that can be applied by operator tasks. Samples are `config maps`, `service`, `deployment`, `stateful set`, ... .


### [unit 1](unit1): parameters, templates, tasks, and plans in action
* showcasing the core operator concepts

### [unit 2](unit2): a stateful service
* showcasing stateful set, and headless service templates
* showcasing containers, and init containers
* showcasing persistent volumes
* showcasing instance update, showing scaling

### [unit 3](unit3): update and upgrade plans
* showcasing update and upgrade Plans
* showcasing controlled parameter updates

### [unit 4](unit4): custom plans
* showcasing backup and restore

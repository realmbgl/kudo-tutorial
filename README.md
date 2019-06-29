# kudo tutorial

<img src="https://kudo.dev/images/kudo_horizontal_color@2x.png" srcset="https://kudo.dev/images/kudo_horizontal_color@2x.png 2x" width="256">

Developing `Kubernetes` operators using `KUDO`, the Kubernetes Universal Declarative Operator. KUDO provides a declarative approach to building production-grade Kubernetes Operators covering the entire application lifecycle.

## install kudo

Run the following three commands to `install the kudo operator`.
```
kubectl create -f https://raw.githubusercontent.com/kudobuilder/kudo/b49ff7d48547bff8c0d2a5fbc01d9ffbae49adbd/docs/deployment/00-prereqs.yaml
kubectl create -f https://raw.githubusercontent.com/kudobuilder/kudo/75603761df510597376f5b1cba01b7e7b8670cc1/docs/deployment/10-crds.yaml
kubectl create -f https://raw.githubusercontent.com/kudobuilder/kudo/75603761df510597376f5b1cba01b7e7b8670cc1/docs/deployment/20-deployment.yaml
```

More on the kudo operator install at [kudo.dev](https://kudo.dev/docs/getting-started/) .


Kudo comes with its own `kubectl CLI plugin`. On `Mac OS X` you can install it using `brew`.

```
brew tap kudobuilder/tap
brew install kudo-cli
```

More on the kudo CLI install at [kudo.dev](https://kudo.dev/docs/cli/) .


## install and operate kudo frameworks

Install and operate your first kudo framework using [kafka](https://kudo.dev/docs/examples/apache-kafka/) as the sample.


## develop kudo frameworks

### artifacts that make a kudo framework

As a developer of a kudo framework you have to author three `YAML` artifacts. The `framework-type`, the `framework-implementation` (aka framework-version), and the `framework-instance`. The artifacts are usually placed in individual YAML files respectively, but you can also just place them in one YAML file which is often handy when you prototype something.

In the following we describe the roles that each of the YAML artifacts plays and what they capture.

#### framework-type
The framework type defines a type of service, e.g. elastic, kafka, ... .

#### framework-implementation
The framework implementation defines the implementation of a framework type, e.g. an elastic, kafka, ... implementation. A framework implementation has a version.

A framework implementation consists of the following.
* `parameters`
  * Parameters allow for the configuration of you framework implementation templates on instantiation.
* `templates`
  * Templates define the resources that can be applied by this framework implementation.
  * Templates are config maps, service, deployment, stateful set, ...
* `tasks`
  * Tasks list the resource templates that get applied together.
* `plans`
  * Plans orchestrate tasks through phases and steps.
  * Plans consists of one or more phases.
  * Phases consists of one or more steps.
  * Steps contain one or more tasks.
  * Both phases and also steps can be configured with an execution strategy, either serial or parallel.

#### framework-instance
The framework instance defines a configured instance of a framework implementation, e.g. an elastic, kafka, ... instance.

### [unit 1](unit1): parameters, templates, tasks, and plans in action
* showcasing the core framework implementation spec concepts

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

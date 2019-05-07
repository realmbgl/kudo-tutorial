# kudo-tutorial

## install kudo

See [here](https://kudo.dev/docs/getting-started/.) and [here](https://github.com/kudobuilder/kudo).


## install and operate kudo frameworks

See [here](https://github.com/kudobuilder/kudo#deploy-your-first-application).


## develop kudo frameworks

### artifacts that make a kudo framework

As a developer of a kudo framework you have to author three artifacts. The framework-type, the framework-implementation (aka framework-version), and the framework-instance. The artifacts are usually placed in individual files respectively, but you can also just place them in one file which is often handy when you prototype something.

In the following we describe the roles that each of the artifacts plays and what they capture.

#### framework-type
The framework type defines a type of service, e.g. elastic.

#### framework-implementation
The framework implementation defines the implementation of a framework type, e.g. an elastic implementation. A framework implementation has a version.

A framework implementation consists of the following.
* `parameters`
  * Parameters allow for the configuration of you framework implementation templates on instantiation.
* `templates`
  * Templates define the resources that can be applied by this framework implementation.
  * Templates like config maps, service, deployment, stateful set, ...
* `tasks`
  * Tasks list the resource templates that get applied together.
* `plans`
  * Plans orchestrate tasks through phases and steps.

  * Plans consists of one or more phases.
  * Phases consists of one or more steps.
  * Steps contain one or more tasks.
  * Both phases and also steps can be configured with an execution strategy, either serial or parallel.

#### framework-instance
The framework instance defines a configured instance of a framework implementation, e.g. an elastic instance.

### [unit 1](unit1/README.md): parameters, templates, tasks, and plans in action
* getting an intuitive understanding of the framework implementation spec concepts

### unit 2: a stateful service
* using a stateful set template
* showcasing persistent volumes
* showcasing instance update, showing scaling

### unit 3: upgrade plans
* showing that upgrade req special consideration for example in the es case not to panic

### unit 4: custom plans
* eg for backup and restore

### unit 5: controlled parameter updates
...

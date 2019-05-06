# kudo-tutorial

## install kudo

## installing and operating kudo frameworks

## developing kudo frameworks

### authoring artifacts that make a kudo framework

#### framework-type
#### framework-implementation (aka framework-version)
framework implementation spec
* parameters
  * provide for configuration of templates
* templates
  * things that can be applied, a template can be … list the most common ones used also that it can be just a config file
* tasks
  * groups templates that are applied together
* plans
  * plans have 1 or more phases (exec parallel or serial)
  * phases have 1 or more steps (exec parallel or serial)
  * steps have 1 or more tasks
#### framework-instance

### excersise 1: parameters, templates, tasks, and plans in action
* getting a 1st intuitive understanding of the framework implementation spec concepts

### excersise 2: a stateful service
* using a stateful set template
* showcasing persistent volumes
* update plan, showing scaling

### excersise 3: upgrade plans
* showing that upgrade req special consideration for example in the es case not to panic 

### excersise 4: custom plans
* eg for backup and restore

### excersise 5: controlled parameter updates

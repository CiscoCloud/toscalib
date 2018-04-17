# Abstract

This library is an implementation of the TOSCA definition as described in the document written in pure GO
[TOSCA Simple Profile in YAML Version 1.1](http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.1/TOSCA-Simple-Profile-YAML-v1.1.html)

## Status

[![GoDoc][1]][2]
[![GoCard][3]][4]
[![coverage][5]][6]
[![Build Status][7]][8]

[1]: https://godoc.org/github.com/CiscoCloud/toscalib?status.svg
[2]: https://godoc.org/github.com/CiscoCloud/toscalib
[3]: https://goreportcard.com/badge/CiscoCloud/toscalib
[4]: https://goreportcard.com/report/github.com/CiscoCloud/toscalib
[5]: http://gocover.io/_badge/github.com/CiscoCloud/toscalib
[6]: http://gocover.io/github.com/CiscoCloud/toscalib
[7]: https://travis-ci.org/CiscoCloud/toscalib.svg?branch=master
[8]: https://travis-ci.org/CiscoCloud/toscalib

## Plans

[ToDo Tasks](TODO.md)

## Normative Types
The normative types definitions are included de facto. The files are embeded using go-bindata.

## Dynamic targets for workflow steps
We made a few tweaks to this library that we thought would be useful. When defining workflow steps, you can now set the step target dynamically using the get_input property function. The library will first look in  workflow inputs for the value and if it is not present in the workflow inputs, will check the template inputs for the same.

Please note that this functionationlity is not as per the specification. This is an additional feature on top of the spec. However, this feature is fully backward compatible with the specification, meaning that you can go ahead and use a plain string as the step target, as per the TOSCA specification. 

### But why?
While working with the TOSCA spec for our applications, we realized that since the workflow step target is a static value, we had to define multiple imperative workflows in order to essentially do the same steps for different nodes. Basically, workflows were not reusable across nodes. With small applications with 2 or 3 nodes this wouldn't be much of an issue, but with large applications with many more nodes, this would lead to extremely verbose profiles with a lot of redundancy.

Lets take an example to make things clearer. 

Assume we define our application as per the below node templates:
```yaml
  node_templates:
    ui-ctnr:
      type: tosca.nodes.ScalableContainer
      properties:
        username: { get_input: uname }
        namespace: { get_input: name_space }
        password: { get_input: pword }
        image_name: dummy/path/to/image
        image_tag: latest
        run_as_root: true
        ports:
          -
            name: web
            port: 80
            protocol: TCP
      requirements:
        - endpoint:
            node: api-svc-1
            capability: tosca.capabilities.Endpoint
            relationship: tosca.relationships.ConnectsTo
        - endpoint:
            node: api-svc-2
            capability: tosca.capabilities.Endpoint
            relationship: tosca.relationships.ConnectsTo

    api-ctnr:
      type: tosca.nodes.ScalableContainer
      properties:
        replicas: 1
        username: { get_input: uname }
        namespace: { get_input: name_space }
        password: { get_input: pword }
        image_name: dummy/path/to/image
        image_tag: latest
        ports:
          -
            name: web
            port: 8080
            protocol: TCP

```		  
This would result in 2 nodes being deployed for our application. Now we expect that the ui-ctnr will see a lot of load, so we define a scale-up workflow so that we can bump up the number of replicas if we need to.
```yaml
workflows:
    scaleup-app-ui:
      inputs:
        increment:
          type: integer
        min_instances:
          type: integer
        max_instances:
          type: integer
      steps:
        scale:
          target: ui-ctnr
          activities:
            - set_state: scaling
            - call_operation: Scale.scaleup
            - set_state: scaled
```
Now we also know that the api-ctnr might also get subjected to a high load during peak hours, so we would want to have a scale-up workflow for the api-ctnr as well. But as per the current TOSCA spec, we will need to define a seperate workflow for api-ctnr.
```yaml
workflows:
    scaleup-app-ui:
      inputs:
        increment:
          type: integer
        min_instances:
          type: integer
        max_instances:
          type: integer
      steps:
        scale:
          target: ui-ctnr
          activities:
            - set_state: scaling
            - call_operation: Scale.scaleup
            - set_state: scaled
    scaleup-app-api:
      inputs:
        increment:
          type: integer
        min_instances:
          type: integer
        max_instances:
          type: integer
      steps:
        scale:
          target: api-ctnr
          activities:
            - set_state: scaling
            - call_operation: Scale.scaleup
            - set_state: scaled
```
Even though, both workflows are essentially the same, we are still forced to define one workflow of each target. In a profile for a complex application with perhaps 10 or 20 nodes, this would result in a large number of workflow definitions.

By changing the target type from string to a PropertyAssignment, we can use a static string value as the target, same as before, or if needed we can assign the target value from a workflow/topology input via the get_input function. This provides flexibility and can simplify the above workflow definitions to:
```yaml
workflows:
    scaleup-app:
      inputs:
        increment:
          type: integer
        min_instances:
          type: integer
        max_instances:
          type: integer
        target:
          type: string
      steps:
        scale:
          target: { get_input: target }
          activities:
            - set_state: scaling
            - call_operation: Scale.scaleup
            - set_state: scaled
```
Now any node that supports the Scale.scaleup call_operation can be scaled up using the scaleup-app workflow. All that would need to be done is to pass in the target you want to scale up as an input to the workflow.

##Do we want to make this part of the spec?

We do believe this feature will improve reusability with regard to TOSCA workflows. We intend to get in touch with the folks who defined the TOSCA specification to both understand the reasoning behind their current specfications for step definition targets, as well as to discuss whether the above feature should/could be included in the specficiations.

# Howto

Create a `ServiceTemplateDefinition` and call `Parse(r io.Reader)` of `ParseCsar(c string)` to fill it with a YAML definition.

## Example

```go
var t toscalib.ServiceTemplateDefinition
err := t.Parse(os.Stdin)
if err != nil {
    log.Fatal(err)
}
```

```go
var t toscalib.ServiceTemplateDefinition
err := t.ParseCsar("tests/tosca_elk.zip")
if err != nil {
    log.Fatal(err)
}
```


## Origins

Original implementation provided by [Olivier Wulveryck](https://github.com/owulveryck) at [github.com/owulveryck/toscalib](https://github.com/owulveryck/toscalib).

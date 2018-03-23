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
When defining workflow steps, you can set the step target dynamically using the get_input property function. The library will first look in  workflow inputs for the value and if it not present in the workflow inputs, will check the template inputs for the same.

This is an additional feature on top of the spec. It does not break the standard spec functionality in any way. So you can set set target as a string as well.

### Example
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
      steps:
        scale:
          target: { get_input: target }
          activities:
            - set_state: scaling
            - call_operation: Scale.scaleup
            - set_state: scaled
```

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

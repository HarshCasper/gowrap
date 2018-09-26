# gowrap [![License](https://img.shields.io/badge/license-mit-green.svg)](https://github.com/hexdigest/gowrap/blob/master/LICENSE) [![Build Status](https://travis-ci.org/hexdigest/gowrap.svg?branch=master)](https://travis-ci.org/hexdigest/gowrap) [![Coverage Status](https://coveralls.io/repos/github/hexdigest/gowrap/badge.svg?branch=master)](https://coveralls.io/github/hexdigest/gowrap?branch=master) [![Go Report Card](https://goreportcard.com/badge/github.com/hexdigest/gowrap)](https://goreportcard.com/report/github.com/hexdigest/gowrap) [![GoDoc](https://godoc.org/github.com/hexdigest/gowrap?status.svg)](http://godoc.org/github.com/hexdigest/gowrap)

GoWrap is a command line tool that generates decorators for Go interface types using simple templates.
With GoWrap you can easily add metrics, tracing, fallbacks, pools and many other features into your existing code in a few seconds.


## Demo

![demo](https://github.com/hexdigest/gowrap/blob/master/gowrap.gif)

## Installation

```
go get -u github.com/hexdigest/gowrap/cmd/gowrap
```

## Usage of gowrap

```
Usage: gowrap gen -p package -i interfaceName -t template -o output_file.go
  -d string
    	source package dir where to look for the interface declaration,
    	default is a current directory
  -g	don't put //go:generate instruction to the generated code
  -i string
    	interface name, i.e. "Reader"
  -o string
    	output file name
  -p string
    	source package import path, i.e. "io" or "github.com/hexdigest/gowrap"
  -t gowrap template list
    	template to use, it can be an HTTPS URL, local file or a
    	reference to one of the templates in the gowrap repository
```

This will generate an implementation of the io.Reader interface wrapped with prometheus metrics

```
  $ gowrap gen -p io -i Reader -t prometheus -o reader_with_metrics.go
```

This will generate a fallback decorator for the Connector interface that can be found in the ./connector directory:

```
  $ gowrap gen -d ./connector -i Connector -t fallback -o ./connector/with_metrics.go
```

Run `gowrap help` for more options

## Hosted templates

When you specify a template with the "-t" flag, gowrap will first search for and use the local file with this name.
If the file is not found, gowrap will look for the template [here](https://github.com/hexdigest/gowrap/tree/master/templates) and use it if found.

List of available templates:
  - [circuitbreaker](https://github.com/hexdigest/gowrap/tree/master/templates/circuitbreaker) stops executing methods of the wrapped interface after specified number of consecutive errors and resumes execution after specified delay
  - [fallback](https://github.com/hexdigest/gowrap/tree/master/templates/fallback) takes several implementations of the source interface and concurrently runs each implementation if previous attempt didn't return result in a specified period, returns first non-error result
  - [opentracing](https://github.com/hexdigest/gowrap/tree/master/templates/opentracing) instruments source interface with opentracing spans
  - [prometheus](https://github.com/hexdigest/gowrap/tree/master/templates/prometheus) instruments source interface with prometheus metrics
  - [retry](https://github.com/hexdigest/gowrap/tree/master/templates/retry) instruments source interface with retries
  - [robinpool](https://github.com/hexdigest/gowrap/tree/master/templates/retry) puts several implementations of the source interface to the slice, for every method call picks one implementation from the slice using Round-robin algorithm
  - [syncpool](https://github.com/hexdigest/gowrap/tree/master/templates/syncpool) puts several implementations of the source interface to the sync.Pool, for every method call gets one implementation from the pool and puts it back once finished

By default GoWrap places the `//go:generate` instruction into the generated code. 
This allows you to regenerate decorators' code just by typing `go generate ./...` when you change the source interface type declaration.
However if you used a remote template, the `//go:generate` instruction will contain the HTTPS URL of the template and therefore
you will need to have internet connection in order to regenerate decorators. In order to avoid this, you can copy templates from the GoWrap repository 
to local files and add them to your version control system:
```
$ gowrap template copy fallback templates/fallback
```

The above command will fetch the fallback template and copy it to the templates/fallback local file.
After template is copied, you can generate decorators using this local template:

```
$ gowrap gen -p io -i Reader -t templates/fallback reader_with_fallback.go
```

## Custom templates

You can always write your own template that will provide the desired functionality to your interfaces.
If you think that your template might be useful to others, please consider adding it to our [template repository](https://github.com/hexdigest/gowrap/tree/master/templates).

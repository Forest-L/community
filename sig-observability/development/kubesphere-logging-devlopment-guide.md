# Logging

This documentation contains backend development guide for interaction with key components behind KubeSphere logging system. Logging backend provides the capabilities of:

- Log search
- Log export
- Log output configuration
- Multi-tenant isolation

## File Tree

The listing below covers all folders related to the logging backend.

```yaml
/pkg
  ├─api
  │  └─logging            # declare structs for api responses
  │     └─v1alpha2
  ├─apiserver             # implement handler for http requests
  │  ├─logging
  │  └─tenant
  ├─kapis                 # register APIs and routing
  │  ├─logging
  │  │  ├─install
  │  │  └─v1alpha2
  │  ├─tenant
  │  │  ├─install
  │  │  └─v1alpha2
  ├─models
  │  ├─log                # constants, utils and fluent-bit CRD operation
  │  │  ├─constants.go
  │  │  ├─logcollector.go # some utils
  │  │  ├─logcrd.go       # interact with fluent-bit CRD
  │  │  └─types.go
  │  └─tenant
  └─simple
     ├─factory.go         # contain factory functions for es client options
     └─client
        ├─elasticsearch   # wrap ES search apis
        │  ├─esclient.go  # construct ES search body
        │  ├─interface.go # general interface methods for ES clients
        │  ├─options.go   # ES client options
        │  └─versions     # client code by ES versions
        │     ├─v5
        │     ├─v6
        │     └─v7
        └─fluentbit       # autogenerated client code for fluent-bit CRD
```

## API Design

There are two types of APIs in logging, one for log query, and the other one for interacting with the CustomResourceDefinition used by [Fluent-bit Operator](https://github.com/kubesphere/fluentbit-operator). For information about CRD and Fluent-bit Operator, please go to its own repo.

To support multi-tenant isolation, KubeSphere's logging query APIs have the format like below, though the underlying logic is using Elastic Search APIs:
  
```bash
GET /namespaces/{namespace}/pods/{pod}/containers/{container}
```

KubeSphere API gateway will decode the URL and conduct authorization. A person who doesn't belong to a namespace will be refused to make a request.
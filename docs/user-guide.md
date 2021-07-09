# User Guide

## Consuming the Bindings from Applications

The Application Projection section of the spec describes how the bindings are projected into the application.  The primary mechanism of projection is through files mounted at a specific directory.  The directory path can be discovered through `SERVICE_BINDING_ROOT` environment variable.  The operator must have injected `SERVICE_BINDING_ROOT` environment to all the containers where binding is applied.  Within this service binding root directory, there could be multiple bindings projected from various sources.  Let's take a look at the example given in the spec:

```
$SERVICE_BINDING_ROOT
├── account-database
│   ├── type
│   ├── provider
│   ├── uri
│   ├── username
│   └── password
└── transaction-event-stream
    ├── type
    ├── connection-count
    ├── uri
    ├── certificates
    └── private-key
```

Files within each binding directory has a special file named `type`, and you can rely on the value of that file to identify the type of the binding projected into that directory.  In certain directories, you can also see another file named `provider`.  The provider is an additional identifier to narrow down the type further.

### Purpose of the type and the provider fields in the application projection

Service Binding specification mandates `type` field and recommends `provider` field in the binding Secret resource.  The provider field is recommended to support scenarios where there could be different providers for the same Provisioned Service type.  For example, if the type is `mysql`, the provider value could be `mariadb`, `oracle`, `bitnami`, `aws-rds`, etc.  When the application is reading the binding values, if necessary, the application could consider `type` and `provider` as a composite key to avoid ambiguity.  It will be helpful if an application needs to choose a particular provider based on the deployment environment.

The operator autogenerates the binding directory name, and relying on the binding name is fragile.  So, it is not a good practice to use the binding name to look up the bindings.

### Programming language specific library APIs

The application can retrieve the binding values through existing libraries available for your programming langauge of the choice.  The API should have a pattern described here.  This is not an exhaustive list of APIs.  Consult the library API documentation to confirm the behavior.

The language binding API can create separate functions to retrieve bindings. For example, in Go:

```
AllBindings() []map[string]string
FilterBindings(_type string) []map[string]string
FilterBindingsWithProvider(_type, provider string) []map[string]string
```

Languages with function overloading can use a method with different parameters to retrieve bindings. For example, in Java:

```
public List<Binding> filterBindings(@Nullable String type)
public List<Binding> filterBindings(@Nullable String type, @Nullable String provider)
```

(Example taken from [Spring Cloud Bindings](https://github.com/spring-cloud/spring-cloud-bindings))

Since the APIs are returning more than one value, depending on your application need, you can choose to connect to the first entry or all of them, if required.

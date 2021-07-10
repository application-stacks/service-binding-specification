# User Guide

## Consuming the Bindings from Applications

The Application Projection section of the spec describes how the bindings are projected into the application.  The primary mechanism of projection is through files mounted at a specific directory.  The bindings directory path can be discovered through `SERVICE_BINDING_ROOT` environment variable.  The operator must have injected `SERVICE_BINDING_ROOT` environment to all the containers where bindings are created.

Within this service binding root directory, there could be multiple bindings projected from different Service Bindings.  For example, suppose an application requires to connect to a database and cache server. In that case, one Service Binding can provide the database, and the other Service Binding can offer bindings to the cache server.

Let's take a look at the example given in the spec:

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
In the above example, there are two bindings under the `SERVICE_BINDING_ROOT` directory.  The `account-database` and `transaction-event-system` are the names of the bindings.  Files within each bindings directory has a special file named `type`, and you can rely on the value of that file to identify the type of the binding projected into that directory.  In certain directories, you can also see another file named `provider`.  The provider is an additional identifier to narrow down the type further.  Retrieving the bindings through the bindings directory name is not a practice -- it makes your application less portable.  Always use the `type` field and, if necessary, `provider` to look up the bindings.

Usually, operators use `ServiceBinding` resource name (`.metadata.name`) as the bindings directory name, but the spec also provides a way to override that name through the `.spec.name` field. So, there is a chance for bindings name collision.  However, due to the nature of the volume mount in Kubernetes, the bindings directory will contain values from only one of the Secret resources.  This is a caveat of using the bindings directory name to look up the bindings.

### Purpose of the type and the provider fields in the application projection

Service Binding specification mandates `type` field and recommends `provider` field in the binding Secret resource.  Normally `type` field should be good enough to look up the bindings.  The provider field could be used where there are different providers for the same Provisioned Service type.  For example, if the type is `mysql`, the provider value could be `mariadb`, `oracle`, `bitnami`, `aws-rds`, etc.  When the application is reading the binding values, if necessary, the application could consider `type` and `provider` as a composite key to avoid ambiguity.  It will be helpful if an application needs to choose a particular provider based on the deployment environment.  In the deployment environment (`stage`, `prod`, etc.), at any given time, you need to ensure only one bindings projection exits for a given `type` or `type` & `provider` -- unless your application needs to connect to all the services.

### Programming language specific library APIs

The application can retrieve the bindings through the library available for your programming language of choice.  The language-specific APIs are encouraged to follow the pattern described here.  This is not an exhaustive list of APIs.  Consult your library API documentation to confirm the behavior.

The language binding API can create separate functions to retrieve bindings. For example, in Go:

```
Bindings(_type string) []map[string]string
BindingsWithProvider(_type, provider string) []map[string]string
```

Languages with function overloading can use a method with different parameters to retrieve bindings. For example, in Java:

```
public List<Binding> filterBindings(@Nullable String type)
public List<Binding> filterBindings(@Nullable String type, @Nullable String provider)
```

(Example taken from [Spring Cloud Bindings](https://github.com/spring-cloud/spring-cloud-bindings))

Since the APIs are returning more than one value, depending on your application need, you can choose to connect to the first entry or all of them, if required.

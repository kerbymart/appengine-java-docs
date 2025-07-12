# API Parameter and Return Types

  

1.  [Supported parameter types](#supported_parameter_types)
    1.  [Path parameters](#path_parameters)
    2.  [Query parameters](#query_parameters)
2.  [Injected types](#injected_types)
3.  [Return types and request body types](#return_types_and_request_body_types)
4.  [Effect of API Transformers on Types](#effect_of_api_transformers_on_types)
    1.  [API Transformers and parameter types](#api_transformers_and_parameter_types)
    2.  [API Transformers and entity types](#api_transformers_and_entity_types)
    3.  [API Transformers and injected types](#api_transformers_and_injected_types)

This page lists the data types you can use as API parameter types in the path or query parameters for your a backend API methods, and the types you can use as method return types or request body types.

**Note:** The API parameter types cannot be used as method *return* types or as the request body of an API request.

API parameters must always be annotated with [`@Named`](https://web.archive.org/web/20160424230550/https://cloud.google.com/appengine/docs/java/endpoints/annotations#named). For example:

```
public Resource get(@Named("id") int id) {  }
```

## Supported parameter types

The supported parameter types are the following:

-   java.lang.String
-   java.lang.Boolean and boolean
-   java.lang.Integer and int
-   java.lang.Long and long
-   java.lang.Float and float
-   java.lang.Double and double
-   java.util.Date
-   com.google.api.server.spi.types.DateAndTime
-   com.google.api.server.spi.types.SimpleDate
-   Any enum
-   Any array or java.util.Collection of a parameter type

### Path parameters

Path parameters are the method parameters included in the `path` property of the [`@ApiMethod`](https://web.archive.org/web/20160424230550/https://cloud.google.com/appengine/docs/java/endpoints/annotations#apimethod_method-scoped_annotations) annotation. If `path` is unspecified, any parameters not annotated with [`@Nullable`](https://web.archive.org/web/20160424230550/https://cloud.google.com/appengine/docs/java/endpoints/annotations#nullable) or `@DefaultValue` will be automatically added to the path (they will be path parameters). For example:

```
public Resource get(@Named("id") int id) {  }
```

To manually add a parameter to `path`, include the parameter name in `{}` marks in the path. Parameters manually added to `path` cannot be annotated with `@Nullable` or `@DefaultValue`. For example:

```
@ApiMethod(path = "resources/{id}")
public Resource get(@Named("id") int id) {  }
```

### Query parameters

Query parameters are the method parameters not included in the `path` property of the `@ApiMethod` annotation. Note that optional parameters, namely, those annotated with `@Nullable` or `@DefaultValue`, are never automatically added to `path`, so they are automatically query parameters if no `path` is specified. For example:

```
public Resource get(@Named("id") @Nullable int id) {  }
```

If `path` is specified, parameters can be made query parameters by not including them in the `path`. For example:

```
@ApiMethod(path = "resources")
public Resource get(@Named("id") int id) {  }
```

## Injected types

Injected types are those types that receive special treatment by the Endpoints framework. If such a type is used as a method parameter, it will not be made a part of the API. Instead, the parameter will be filled in by the framework.

The injected types are the following:

-   com.google.appengine.api.users.User
-   javax.servlet.http.HttpServletRequest
-   javax.servlet.ServletContext

## Return types and request body types

A method return type as well as the request body of an API request must be an entity type. Any type except a parameter or injected type is considered an entity type. (Entity types are generally used for resources in a REST API.) Entity types cannot be annotated with `@Named`, and because an API request can only have one body, there can only be a single entity parameter per method. For example:

```
public Resource set(Resource resource) {  }
```
**Note:** Returning a simple type such as String or int is not supported. The return value needs to be a JavaBean (non-parametrized), an array or a Collection. CollectionResponse (`com.google.api.server.spi.response.CollectionResponse`) or its subclasses have special treatment. In general parametrized beans are not currently supported.

## Effect of API Transformers on Types

If you use [API transformers](https://web.archive.org/web/20160424230550/https://cloud.google.com/appengine/docs/java/endpoints/annotations#apitransformer), you'll need to be aware of how the type is impacted by the transformation.

### API Transformers and parameter types

When an API transformer is used, whether or not the resulting method parameter is considered a *parameter type* depends on the result of transformation, *not on the original type*. If a type is transformed to a parameter type, the method parameter will be considered an API parameter and must be annotated with `@Named`.

### API Transformers and entity types

Similarly, whether or not a method parameter or return type is considered an *entity type* depends on the result of transformation, not on the original type. If a type is transformed to an entity type, the type can be used as a return type, and when used as a method parameter, it cannot be annotated with `@Named` as it will be considered an entity type.

### API Transformers and injected types

Injected types are not affected by API transformers. A method parameter is only considered an injected type if its original non-transformed type is an injected type.

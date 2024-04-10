# Endpoint Annotations

  

1.  [@Api: API-Scoped Annotations](#api_api-scoped_annotations)
    1.  [Required Imports](#required_imports)
    2.  [Attributes](#attributes)
    3.  [Sample @Api Annotation](#sample_api_annotation)
    4.  [Client IDs and Audiences](#client_ids_and_audiences)
2.  [@ApiMethod: Method-Scoped Annotations](#apimethod_method-scoped_annotations)
    1.  [Required Imports](#required_imports_1)
    2.  [Attributes](#attributes_1)
    3.  [Sample @ApiMethod Annotation](#sample_apimethod_annotation)
3.  [@Named](#named)
    1.  [Required Imports](#required_imports_2)
4.  [@ApiNamespace](#apinamespace)
5.  [@Nullable](#nullable)
6.  [@ApiClass](#apiclass)
7.  [@ApiReference](#apireference)
8.  [@ApiResourceProperty](#apiresourceproperty)
    1.  [Required Imports](#required_imports_3)
    2.  [Attributes](#attributes_2)
    3.  [Sample Class with @ApiResourceProperty](#sample_class_with_apiresourceproperty)
9.  [@ApiTransformer](#apitransformer)
    1.  [Required imports](#required_imports_4)
    2.  [Sample class with @ApiTransformer](#sample_class_with_apitransformer)
    3.  [Sample Endpoints transformer class](#sample_endpoints_transformer_class)

See the [backend API tutorial](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/) for a recommended way of adding annotations using a Maven project. Maven App Engine Endpoints artifacts are provided to make it easy to create and build a backend API, and generate a client library from it. Alternatively, you could use the [Google Plugin for Eclipse](https://web.archive.org/web/20160424225723/https://cloud.google.com/eclipse/docs/cloud_endpoints) (GPE).

Endpoint annotations describe API configuration, methods, parameters, and other vital details that define the properties and behavior of the Endpoint. The annotation that specifies configuration and behavior across the entire API (affecting all the classes exposed in the API and all their exposed methods) is [@Api](#API-Scoped-Annotations). *All public, non-static, non-bridge methods of a class annotated with `@Api` will be exposed in the public API.*

If you need special API configuration for a particular method, you can optionally use [@ApiMethod](#Method-Scoped-Annotations) to set configuration on a per method basis. You configure these annotations by setting various attributes, as shown in the tables below.

**Note:** The Javadoc lists some annotations that are currently not implemented. These annotations are: [ApiCacheControl](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/ApiCacheControl), and [ApiFrontendLimits](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/ApiFrontendLimits).

## @Api: API-Scoped Annotations

**Note:** In the following table on backend configuration, we refer to the paths `/_ah/spi`, `/_ah/api`. Applications send their requests to `/_ah/api`, and the Endpoints backend handles these requests at the path `/_ah/spi`.

The annotation [@Api](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/Api) is used to configure the whole API, and apply to all public methods of a class unless overridden by `@ApiMethod`.

**Important:** If you implement your API from multiple classes, see [Multiclass APIs](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/multiclass).

(To override a given `@Api` annotation for a specific class within an API, see [@ApiClass](#@ApiClass) and [@ApiReference](#@ApiReference).)

### Required Imports

To use this feature, you need the following import:

```
import com.google.api.server.spi.config.Api;
```

### Attributes

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>@Api Attributes</th>
<th>Description</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>audiences</code></td>
<td>Required if your API requires authentication and if you are supporting Android clients. For more information, see <a href="#client_ids_and_audiences">Client IDs and Audiences</a>.</td>
<td><code>audiences = {"1-web-apps.apps.googleusercontent.com", "2-web-apps.apps.googleusercontent.com"}</code></td>
</tr>
<tr class="even">
<td><code>backendRoot</code></td>
<td>The root URL used for method calls to backend instances. If not supplied, the default <code>https://your_app_id.appspot.com/_ah/spi</code> is used.</td>
<td><code>backendRoot = "https://example.appspot.com/_ah/spi"</code></td>
</tr>
<tr class="odd">
<td><code>canonicalName</code></td>
<td>Used to specify a different or more readable name for the API <em>in the client library</em>. This name is used to generate the names in the client library; the backend API continues to use the value specified in the <code>name</code> property.<br />
<br />
For example, if your API has the <code>name</code> set to <strong>dfaanalytics</strong>, you could use this property to specify a canonical name of <strong>DFA Group Analytics</strong>; the generated client classes would then contain the name <code>DfaGroupAnalytics</code>.<br />
<br />
You should include the relevant spaces between the names as shown above; these will be replaced by the appropriate camel casing or underscores.</td>
<td><code>canonicalName = "DFA Analytics:"</code>n</td>
</tr>
<tr class="even">
<td><code>clientIds</code></td>
<td>Required if your API uses authentication. List of client IDs for clients allowed to request tokens. For more information, see <a href="#client_ids_and_audiences">Client IDs and Audiences</a>.</td>
<td><code>clientIds = {"1-web-apps.apps.googleusercontent.com", "2-android-apps.apps.googleusercontent.com"}</code></td>
</tr>
<tr class="odd">
<td><code>defaultVersion</code></td>
<td>Specifies whether a default version is used if none is supplied in the <code>version</code> attribute.</td>
<td><code>defaultVersion = AnnotationBoolean.TRUE</code></td>
</tr>
<tr class="even">
<td><code>description</code></td>
<td>A short description of the API. This is exposed in the discovery service to describe your API, and may optionally also be used to generate documentation.</td>
<td><code>description = "Sample API for a simple game"</code></td>
</tr>
<tr class="odd">
<td><code>documentationLink</code></td>
<td>The URL where users can find documentation about this version of the API. This will be surfaced in the API Explorer "Learn More" highlight at the top of the API Explorer page and also in the <a href="https://web.archive.org/web/20160424225723/https://cloud.google.com/eclipse/docs/cloud_endpoints">GPE plugin</a> to allow users to learn about your service.</td>
<td><code>documentationLink = "http://link_to/docs"</code></td>
</tr>
<tr class="even">
<td><code>name</code></td>
<td>The name of the API, which is used as the prefix for all of the API's methods and paths. The <code>name</code> value:
<ul>
<li><em>Must</em> begin with lowercase</li>
<li><em>Must</em> match the regular expression <code>[a-z]+[A-Za-z0-9]*</code></li>
</ul>
If you don't specify <code>name</code>, the default <code>myapi</code> is used.</td>
<td><code>name = "foosBall"</code></td>
</tr>
<tr class="odd">
<td><code>namespace</code></td>
<td>Configures namespacing for generated clients. See <a href="#apinamespace">@ApiNamespace</a>.</td>
<td><code>namespace=@ApiNamespace(ownerDomain="your-company.com", ownerName="YourCo", packagePath="cloud/platform"</code>)</td>
</tr>
<tr class="even">
<td><code>root</code></td>
<td>The frontend root URL under which your API methods are exposed for. If not supplied, the default <code>https://your_app_id.appspot.com/_ah/api</code> is used.</td>
<td><code>root = "https://example.appspot.com/_ah/api"</code></td>
</tr>
<tr class="odd">
<td><code>scopes</code></td>
<td>If not supplied, the default is the email scope (<code>https://www.googleapis.com/auth/userinfo.email</code>), which is required for OAuth. You can override this to specify more <a href="https://web.archive.org/web/20160424225723/https://cloud.google.com/accounts/docs/OAuth2">OAuth 2.0 scopes</a> if you wish. However, if you do define more than one scope, note that the scope check will pass if the token is minted for <em>any</em> of the specified scopes. To override the scopes specified here for a particular API method, specify different scopes in the <code>@ApiMethod</code> annotation.</td>
<td><code>scopes = {"ss0", "ss1"}</code></td>
</tr>
<tr class="even">
<td><code>title</code></td>
<td>The text displayed in API Explorer as the title of your API, and exposed in the discovery and the directory services.</td>
<td><code>title = "My Backend API"</code></td>
</tr>
<tr class="odd">
<td><code>transformers</code></td>
<td>Specifies a list of custom <a href="#sample_endpoints_transformer_class">transformers</a>. Note that there is an alternative annotation (<a href="#apitransformer">@ApiTransformer</a>) that is preferable. This attribute is overridden by <code>@ApiTransformer</code>.</td>
<td><code>transformers = {BazTransformer.class}</code></td>
</tr>
<tr class="even">
<td><code>version</code></td>
<td>Specifies your Endpoint’s version. If you don't supply this, the default <code>v1</code> is used.</td>
<td><code>version = "v2"</code></td>
</tr>
</tbody>
</table>

### Sample @Api Annotation

This annotation is placed above the class definition:

```
import com.google.api.server.spi.config.AnnotationBoolean;
import com.google.api.server.spi.config.Api;
import com.google.api.server.spi.config.ApiAuth;

@Api(
    version = "v1",
    description = "Sample API",
    scopes = {"ss0", "ss1"},
    audiences = {"aa0", "aa1"},
    clientIds = {"cc0", "cc1"},
    defaultVersion = AnnotationBoolean.TRUE
)
```

### Client IDs and Audiences

For OAuth2 authentication, an OAuth2 token is issued to a specific client ID, which means that this client ID can be used for restricting access to your APIs. When you register an iOS or Android application in the Google Cloud Platform Console, you create a client ID for it. This client ID is the one requesting an OAuth2 token from Google for authentication purposes. When the backend API is protected by auth, an OAuth2 access token is sent and opened by Google Cloud Endpoints, the client ID is extracted from the token, and then the ID is compared to the backend's declared acceptable Client ID list (the `clientIds` list).

So, if you want your Endpoints API to authenticate callers, you need to supply a list of `clientIds` that are allowed to request tokens that can be used with your application. This list should consist of the all client IDs you have obtained through the [Cloud Platform Console](https://web.archive.org/web/20160424225723/https://console.cloud.google.com/) for your [web](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/consume_js), [Android](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/consume_android), or [iOS](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/consume_ios) clients. (This means that the clients must be known at API build-time.) If you specify an empty list, `{}`, then no clients can access the methods protected by Auth.

If you use the `clientIds` attribute and you want to test authenticated calls to your API using the Google API Explorer, you must supply its client ID in the list of `clientIds`: the value to use is `com.google.api.server.spi.Constant.API_EXPLORER_CLIENT_ID`.

**Note:** If you don't use the `clientIds` attribute, you don't need to supply the API Explorer client ID.

#### About Audiences

**Note:** The `audiences` argument is currently used *only for Android clients*.

The `clientIds` list *protects the backend API* from unauthorized clients. But further protection is needed to *protect the clients*, so that their auth token will work only for the intended backend API. For Android clients, this mechanism is the `audiences` attribute, in which you specify the client ID of the backend API.

Note that when you create a Google Cloud Platform Console Project, a default client ID is automatically created and named for use by the project. When you upload your backend API into App Engine, it uses that client ID. This is the web client ID mentioned in the backend API [auth](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/auth) docs and the [backend tutorial](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/auth) docs.

## @ApiMethod: Method-Scoped Annotations

The annotation [@ApiMethod](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/ApiMethod) is used to supply a different API configuration than the defaults provided by the `@Api` or `@ApiClass` annotations. Note that this is optional: all public, non static, non bridge methods in a class with an `@Api` annotation are exposed in the API, whether they have an `@ApiMethod` annotation or not.

Attributes within this annotation allow you to configure details of a single API method. If the same attribute is specified in `@Api` and `@ApiMethod`, `@ApiMethod` overrides.

**Important:** Certain query parameters are reserved and cannot be used in an API method. For a list of these, see <a href="https://web.archive.org/web/20160424225723/https://developers.google.com/drive/web/query-parameters" target="_blank">Standard Query Paramenters</a>

### Required Imports

To use this feature, you need the following imports:

```
import com.google.api.server.spi.config.AnnotationBoolean;
import com.google.api.server.spi.config.ApiMethod;
import com.google.api.server.spi.config.ApiMethod.HttpMethod;
```

### Attributes

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>@ApiMethod Attributes</th>
<th>Description</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>name</code></td>
<td>The name for this method in the generated client library. This is automatically prefixed with your API name to create a unique name for the method. The <code>name</code> value:
<ul>
<li><em>Must</em> begin with lowercase</li>
<li><em>Must</em> match the regular expression <code>[a-z]+[A-Za-z0-9]*</code></li>
</ul>
If you don't specify <code>name</code>, the default <code>myapi</code> is used.</td>
<td><code>name = "foosBall.list"</code></td>
</tr>
<tr class="even">
<td><code>path</code></td>
<td>The URI path to use to access this method. If you don't set this, a default path is used based on the Java method name.</td>
<td><code>path = "foos"</code></td>
</tr>
<tr class="odd">
<td><code>httpMethod</code></td>
<td>The HTTP method to use. If you don't set this, a default is chosen based on the name of the method.</td>
<td><code>httpMethod = HttpMethod.GET</code></td>
</tr>
<tr class="even">
<td><code>scopes</code></td>
<td>Specify one or more OAuth 2.0 scopes, one of which is required for calling this method. If you set <code>scopes</code> for a method, it overrides the setting in the <code>@Api</code> annotation.</td>
<td><code>scopes = {"ss0", "ss1"}</code></td>
</tr>
<tr class="odd">
<td><code>audiences</code></td>
<td>Supply this if you wish to override the configuration in <code>@API</code>. For more information, see <a href="#client_ids_and_audiences">Client IDs and Audiences</a>.</td>
<td><code>audiences = {"1-web-apps.apps.googleusercontent.com", "2-web-apps.apps.googleusercontent.com"}</code></td>
</tr>
<tr class="even">
<td><code>clientIds</code></td>
<td>List of client IDs for clients allowed to request tokens. Required if your API uses authentication.</td>
<td><code>clientIds = {"1-web-apps.apps.googleusercontent.com", "2-android-apps.apps.googleusercontent.com"}</code></td>
</tr>
</tbody>
</table>

### Sample @ApiMethod Annotation

This annotation is placed above the method definition inside a class:

```
import com.google.api.server.spi.config.AnnotationBoolean;
import com.google.api.server.spi.config.ApiMethod;
import com.google.api.server.spi.config.ApiMethod.HttpMethod;

@ApiMethod(
    name = "foos.list",
    path = "foos",
    httpMethod = HttpMethod.GET,
    scopes = {"s0", "s1"},
    audiences = {"a0", "a1"},
    clientIds = {"c0", "c1"}
)
public List<Foo> listFoos() {
  return null;
}
```

Methods that take an entity as a parameter should use `HttpMethod.POST` (for insert operations) or `HttpMethod.PUT` (for update operations):

```
@ApiMethod(
    name = "foos.insert",
    path = "foos",
    httpMethod = HttpMethod.POST
)
public void insertFoo(Foo foo) {
}
```

## @Named

The `@Named` annotation is required for all non-entity type parameters passed to server-side methods. This annotation indicates the name of the parameter in the request that gets injected here. A parameter that is not annotated with `@Named` is injected with the whole request object.

**Important:** There are several commonly used `@Named` annotations: the ones you can use with Endpoints are `javax.inject.Named` or `com.google.api.server.spi.config.Named`.

### Required Imports

To use this feature, you need the following imports:

```
import javax.inject.Named;
```

This sample shows the use of `@Named`:

```
@ApiMethod(name = "foos.remove", path = "foos/{id}", httpMethod = HttpMethod.DELETE)
public void removeFoo(@Named("id") String id) {
}
```

where `@Named` specifies that only the `id` parameter is injected in the request.

## @ApiNamespace

The [@ApiNamespace](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/ApiNamespace.html) annotation causes the generated client libraries to have the namespace you specify, rather than a default constructed during client library generation.

By default, if you don't use this annotation, the namespace that is used is the reverse of `your-project-id.appspot.com`. That is, the package path will be `com.appspot.your-project-id.yourApi`.

You can change the default namespace by supplying the `@ApiNamespace` annotation within the `@Api` annotation:

```
@Api(
    name = "tictactoe",
    version = "v1",
    namespace=@ApiNamespace(ownerDomain="your-company.com", ownerName="YourCo", packagePath="cloud/platform")
)
```

with the `ownerDomain` attribute set to your own company domain and `ownerName` set to your company name, for example, `your-company.com`. The reverse of the `ownerDomain` will then be used for the package path: `com.your-company.yourApi`.

You can optionally use the `packagePath` attribute to provide further scoping. For example, by setting `packagePath` to `cloud`, the package path used in the client library will be `com.your-company.cloud.yourApi`. You can add more values to the package path by supplying the delimiter `/`: `packagePath="cloud/platform"`.

## @Nullable

This annotation indicates that a parameter of a method is optional (and therefore a query parameter). `@Nullable` may only be used with `@Named` parameters.

## @ApiClass

In a [multiclass API](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/multiclass), used to specify different properties for a given class, overriding equivalent properties in the `@Api` configuration. See [Using @ApiClass for Properties that Can Differ Between Classes](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/multiclass#using_apiclass_for_properties_that_can_differ_between_classes) for a complete description of this annotation.

## @ApiReference

In a [multiclass API](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/multiclass), used to supply an alternate method of annotation inheritance. See [Using @ApiReference Inheritance](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/multiclass#using_apireference_inheritance) for a complete description of this annotation.

## @ApiResourceProperty

[@ApiResourceProperty](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/ApiResourceProperty) provides more control over how resource properties are exposed in the API. You can use it on a property getter or setter to omit the property from an API resource. You can also use it on the field itself, if the field is private, to expose it in the API. You can also use this annotation to change the name of a property in an API resource.

### Required Imports

To use this feature, you need the following imports:

```
import com.google.api.server.spi.config.ApiResourceProperty;
import com.google.api.server.spi.config.AnnotationBoolean;
```

### Attributes

<table>
<thead>
<tr class="header">
<th>@ApiResourceProperty Attributes</th>
<th>Description</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>ignored</code></td>
<td>If set to <code>AnnotationBoolean.TRUE</code>, omits the property. If not specified or set to <code>AnnotationBoolean.FALSE</code>, the property is not omitted.</td>
<td><code>@ApiResourceProperty(ignored = AnnotationBoolean.TRUE)</code></td>
</tr>
<tr class="even">
<td><code>name</code></td>
<td>If supplied, it specifies the property name to be exposed in the API.</td>
<td><code>@ApiResourceProperty(name = "baz")</code></td>
</tr>
</tbody>
</table>

### Sample Class with @ApiResourceProperty

The following snippet shows a class with property getters annotated with `@ApiResourceProperty`:

```
import com.google.api.server.spi.config.Api;
import com.google.api.server.spi.config.ApiResourceProperty;
import com.google.api.server.spi.config.AnnotationBoolean;

@Api(name = "myendpoint")
public class MyEndpoint {
  class Resp {
    private String foobar = "foobar";
    private String bin = "bin";

    @ApiResourceProperty
    private String visible = "nothidden";

    @ApiResourceProperty(ignored = AnnotationBoolean.TRUE)
    public String getBin() {
      return bin;
    }

    public void setBin(String bin) {
      this.bin = bin;
    }

    @ApiResourceProperty(name = "baz")
    public String getFoobar() {
      return foobar;
    }

    public void setFoobar(String foobar) {
      this.foobar = foobar;
    }
  }

  public Resp getResp() {
    return new Resp();
  }
}
```

In the code snippet above, `@ApiResourceProperty` is applied to the `getBin` getter for the `bin` property, with the `ignored` attribute setting telling the Endpoints framework to omit this property in the API resource.

`@ApiResourceProperty` is also applied to the private field `visible`, which has no getter or setter. Using this annotation will expose this field as a property in the API resource.

In the same snippet, `@ApiResourceProperty` is also applied to a different getter, `getFoobar`, which returns a property value for the `foobar` property. The `name` attribute in this annotation tells the Endpoints framework to change the property name in the API resource. The property value itself is unchanged.

In the above example snippet, the JSON representation of a `Resp` object would look something like this:

```
{"baz": "foobar", "visible": "nothidden"}
```

## @ApiTransformer

The [@ApiTransformer](https://web.archive.org/web/20160424225723/https://cloud.google.com/appengine/docs/java/endpoints/javadoc/com/google/api/server/spi/config/ApiTransformer) annotation customizes how a type is exposed in Endpoints through transformation to and from another type. (The transformer specified must be an implementation of `com.google.api.server.spi.config.Transformer`.)

Using the `@ApiTransformer` annotation on a class is the preferred way to specify a transformer. However, you could alternatively specify your custom transformer in the `transformer` attribute of the [@Api](#api_api-scoped_annotations) annotation.

**Note:** Using `@ApiTransformer` is preferred to specifying transformers on an `@Api` annotation. If your app consists of multiple APIs, the transformer specified in `@ApiTransformer` works for all of them, whereas using `@Api` to specify the transformer requires you to repeat this specification in each API.

### Required imports

To use this feature, you need this import:

```
import com.google.api.server.spi.config.ApiTransformer;
```

### Sample class with @ApiTransformer

The following snippet shows a class annotated with `@ApiTransformer`:

```
import com.google.api.server.spi.config.ApiTransformer;

@ApiTransformer(BarTransformer.class)
public class Bar {
  private final int x;
  private final int y;

  public Bar(int x, int y) {
    this.x = x;
    this.y = y;
  }

  public int getX() {
    return x;
  }

  public int getY() {
    return y;
  }
}
```

This class will be transformed by the `BarTransformer` class.

### Sample Endpoints transformer class

The following snippet shows a sample transformer class named `BarTransformer`. This is the transformer referenced by `@ApiTransformer` in the preceding snippet:

```
import com.google.api.server.spi.config.Transformer;

public class BarTransformer implements Transformer<Bar, String> {
  public String transformTo(Bar in) {
    return in.getX() + "," + in.getY();
  }

  public Bar transformFrom(String in) {
    String[] xy = in.split(",");
    return new Bar(Integer.parseInt(xy[0]), Integer.parseInt(xy[1]));
  }
}
```

Assuming we have an object with a property `bar` of type `Bar`, without the above transformer, the object would be represented as:

```
{"bar": {"x": 1, "y": 2}}
```

With the transformer, the object is represented as:

```
{"bar": "1,2"}
```
**Note:** Since `BarTransformer` converts `Bar` objects to `String` objects, the same rules for `String` now apply to `Bar`. That is, `Bar` can no longer be used as the response type of an API method, as primitives and enums are disallowed as API resource types. This also means that `Bar` can only be a request parameter type if used as a named parameter.

**Note:** The source type for a transformer can be a raw generic type. This allows the transformer to handle transformation for all instances of the generic type.

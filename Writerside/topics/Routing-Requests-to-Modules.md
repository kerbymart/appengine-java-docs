# Routing Requests to Modules

  

HTTP requests from users can reach the appropriate module/version/instance in two ways: A request with a URL that ends at the domain level can be routed according to App Engine's default address routing rules. Alternatively, you can include a dispatch file that routes specific URL patterns according to your own rules.

If you test your app using the [development server](https://web.archive.org/web/20160424230701/https://cloud.google.com/appengine/docs/java/tools/devserver) the available routing and dispatch features are slightly different. To programmatically create URLs that work with both production and development servers, use the [`ModulesService.getVersionHostname`](https://web.archive.org/web/20160424230701/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/modules/ModulesService#getVersionHostname(java.lang.String,%20java.lang.String,%20int)) method. See [routing in the development server](#Java_Routing_in_the_development_server) to learn more.

1.  [Routing via URL](#routing_via_url)
    1.  [Default module](#default_module)
    2.  [Soft routing](#soft_routing)
    3.  [Restricting access to a module](#restricting_access_to_a_module)
2.  [Routing with a dispatch file](#routing_with_a_dispatch_file)
    1.  [Uploading the dispatch file](#uploading_the_dispatch_file)
3.  [Routing in the development server](#routing_in_the_development_server)
    1.  [Discovering instance addresses](#discovering_instance_addresses)
    2.  [Dispatch files](#dispatch_files)

## Routing via URL

You can target an HTTP request with varying degrees of specificity. In the following examples appspot.com can be replaced with your app's custom domain if you have one. The URL substrings "instance", "version", "module", and "app-id" represent application and module attributes that you have defined yourself.

**Note:** Google [recommends](https://web.archive.org/web/20160424230701/http://googleonlinesecurity.blogspot.com/2014/08/https-as-ranking-signal_6.html) using the HTTPS protocol to send requests to your app. Google does not issue SSL certificates for double-wildcard domains hosted at `appspot.com`. Therefore with HTTPS you must use the string "-dot-" instead of "." to separate subdomains, as shown in the examples below. You can use a simple "." with your own custom domain or with HTTP addresses.

These address forms are guaranteed to reach their target (if it exists). They will never be intercepted and rerouted by a pattern in the dispatch file:

`https://instance-dot-version-dot-module-dot-app-id.appspot.com`  
  
`http://instance.version.module.app-id.my-custom-domain.com`    
Sends the request to the named module, version, and instance.  
  

`https://version-dot-module-dot-app-id.appspot.com`  
  
`http://version.module.app-id.my-custom-domain.com`    
Sends the request to an available instance of the named module and version.

These address forms have a default routing behavior. Note that the default routing is overridden if there is a matching pattern in the dispatch file:

`https://module-dot-app-id.appspot.com`  
  
`http://module.app-id.my-custom-domain.com`    
Sends the request to an available instance of the default version of the named module.  
  

`https://version-dot-app-id.appspot.com`  
  
`http://version.app-id.my-custom-domain.com`    
Sends the request to an available instance of the given version of the default module.  
  

`https://app-id.appspot.com`  
  
`http://app-id.my-custom-domain.com`    
Sends the request to an available instance of the default version of the default module.  
  

### Default module

The default module is defined by explicitly giving a module the name "default," or by not including the name parameter in the module's config file. Requests that specify no module or an invalid module are routed to the default module. You can designate a default version for a module, when appropriate, in the [Google Cloud Platform Console versions](https://web.archive.org/web/20160424230701/https://console.cloud.google.com//project/_/appengine/versions) tab.

### Soft routing

If a request matches the `app-id.appspot` portion of the hostname, but includes a module, version, or instance name that does not exist, then the request is routed to the default version of the default module. Soft routing does not apply to custom domains; requests to them will return a 404 if the hostname is invalid.

**Note:** Version names should begin with a letter, to distinguish them from numeric instances which are always specified by a number. This avoids the ambiguity with URLs like `123.my-module.appspot.com`, which can be interpreted two ways: If version “123” exists, the target will be version “123” of the given module. If that version does not exist, the target will be instance number 123 of the default version of the module.

### Restricting access to a module

All modules are public by default. If you want to restrict access to a module, add the “login: admin” parameter to its handlers.

## Routing with a dispatch file

For certain URLs (described above), you can create a dispatch file that overrides the routing rules. This lets you send incoming requests to a specific module based on the path or hostname in the URL. For example, say that you want to route mobile requests like `http://simple-sample.appspot.com/mobile/` to a mobile frontend, route worker requests like `http://simple-sample.appspot.com/work/` to a static backend, and serve all static content from the default module.

To do this you can create a custom routing with a `dispatch.xml` file. The file should be placed in the WEB-INF directory of the default module.

```
<?xml version="1.0" encoding="UTF-8"?>
<dispatch-entries>
  <dispatch>
      <!-- Default module serves the typical web resources and all static resources. -->
      <url>*/favicon.ico</url>
      <module>default</module>
  </dispatch>
  <dispatch>
      <!-- Default module serves simple hostname request. -->
      <url>simple-sample.appspot.com/</url>
      <module>default</module>
  </dispatch>
  <dispatch>
      <!-- Send all mobile traffic to the mobile frontend. -->
      <url>*/mobile/*</url>
      <module>mobile-frontend</module>
  </dispatch>
  <dispatch>
      <!-- Send all work to the one static backend. -->
      <url>*/work/*</url>
      <module>static-backend</module>
  </dispatch>
</dispatch-entries>
```

The dispatch file can contain up to 10 routing rules. When specifying the URL string, neither the hostname nor the path can be longer than 100 characters.

As you can see, `dispatch.xml` includes support for glob characters. Glob characters can be used only before the hostname and at the end of the path. If you prefer general routing rules that match many possible requests, you could specify the following:

```
<!-- Send any path that begins with simple-sample.appspot.com/mobile to the mobile-frontend module. -->
<url>simple-sample.appspot.com/mobile*</url>
<module>mobile-frontend</module>

<!-- Send any domain/sub-domain with a path that starts with work to the static backend module. -->
<url>*/work*</url>
<module>static-backend</module>
```

You can also write expressions that are more strict:

```
<!-- Matches the path "/fun", but not "/fun2" or "/fun/other"  -->
<url>*/fun</url>
<module>mobile-frontend</module>

<!-- Matches the hostname 'customer1.myapp.com', but not '1.customer1.myapp.com. -->
<url>customer1.myapp.com/*</url>
<module>static-backend</module>
```
**Note:** Dispatch rules apply to the URLs in a [cron file](https://web.archive.org/web/20160424230701/https://cloud.google.com/appengine/docs/java/config/cron).

### Uploading the dispatch file

To upload the dispatch file, use the [appcfg update\_dispatch](https://web.archive.org/web/20160424230701/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp#Java_Updating_the_dispatch_file) command, and specify the war directory for the default module. Be sure that all the modules mentioned in the file have already been uploaded before using this command.
```
# cd to the war directory containing the default module
appcfg.sh update_dispatch .
```

You can also upload the dispatch file at the same time you upload one or more modules, by adding the optional `auto_update_dispatch` flag, which can be used in two forms:

```
appcfg.sh --auto_update_dispatch update <app-directory>|<files...>
appcfg.sh -D update <app-directory>|<files...>
```

## Routing in the development server

### Discovering instance addresses

The development server creates all instances at startup. Note that at this time basic scaling instances are not supported on the development server. Every instance that is created is assigned its own port. The port assignments appear in the server's log message stream. Web clients can communicate with a particular instance by targeting its port. Only one instance (and port) is created for automatic scaled modules. It looks like this in the server log:

```
INFO: Module instance module2-auto is running at http://localhost:37251/
```

A unique port is assigned to each instance of a manual scaled module:

```
INFO: Module instance manualmodule instance 0 is running at http://localhost:43190/
INFO: Module instance manualmodule instance 1 is running at http://localhost:52642/
```

In addition, each manual scaled module is assigned one extra port so clients can access the module without specifying a specific instance. Requests to this port are automatically routed to one of the configured instances:

```
INFO: Module instance manualmodule is running at http://localhost:12361/
```

The following table shows how these modules can be called in the development server and in the App Engine environment:

<table>
<thead>
<tr class="header">
<th>Module</th>
<th>Instance</th>
<th>Development Server Address</th>
<th>App Engine Address</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>module2-auto</td>
<td>(not used)</td>
<td><code>http://localhost:37251/</code></td>
<td><code>http://v1.module2-auto.appid.appspot.com/</code></td>
</tr>
<tr class="even">
<td>manualmodule</td>
<td>0</td>
<td><code>http://localhost:43190/</code></td>
<td><code>http://0.v1.manualmodule.appid.appspot.com/</code></td>
</tr>
<tr class="odd">
<td>manualmodule</td>
<td>1</td>
<td><code>http://localhost:52642/</code></td>
<td><code>http://1.v1.manualmodule.appid.appspot.com/</code></td>
</tr>
<tr class="even">
<td>manualmodule</td>
<td>(not used)</td>
<td><code>http://localhost:12361/</code></td>
<td><code>http://v1.manualmodule.appid.appspot.com/</code></td>
</tr>
</tbody>
</table>

### Dispatch files

All dispatch files are ignored when running the development server. The only way to target instances is through their ports.

# Generating Libraries and Discovery Docs with the Endpoints Tool

  

1.  [Generating a client library bundle from a backend API](#generating_a_client_library_bundle_from_a_backend_api)
2.  [Generating a discovery doc from a backend API](#generating_a_discovery_doc_from_a_backend_api)
3.  [Command line syntax for the Endpoints tool](#command_line_syntax_for_the_endpoints_tool)
4.  [Options](#options)

You can use the Endpoints command line tool to generate client library bundles and discovery docs. Android clients need a client library bundle to [access the backend API](https://web.archive.org/web/20160424230828/https://cloud.google.com/appengine/docs/java/endpoints/consume_android). iOS clients need the discovery doc.

These instructions assume you have created your backend API. (See the [backend API tutorial](https://web.archive.org/web/20160424230828/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend) for more information on creating backend APIs.)

**Note:** You can alternatively use Maven to generate Maven client library bundles and discovery documents from the backend API, as described in [Generating Client Libraries](https://web.archive.org/web/20160424230828/https://cloud.google.com/appengine/docs/java/endpoints/gen_clients). However, this option won't generate a Gradle client bundle to use with an Android project.

The Endpoints command line tool is provided in the SDK: `appengine-java-sdk-x.x.x/bin/endpoints.sh`.

## Generating a discovery doc from a backend API

**Warning:** The generated discovery document should not be edited.

A [Google discovery document](https://web.archive.org/web/20160424230828/https://cloud.google.com/discovery/v1/reference/apis) describes the surface for a particular version of an API. The information provided by the discovery document includes API-level properties such as an API description, resource schemas, authentication scopes, and methods. The Endpoints command line tool can generate a discovery document in either REST format (the default) or RPC format (required for generating an iOS client).

To generate the discovery document:

**Note:** The instructions below are for a Maven project. If you use Eclipse, the project layout will be slightly different and you will have to invoke the Endpoints commmand line tool from the parent directory of the `war` directory in your target build output.

1.  If you haven't already done so, build the backend API.

2.  Invoke the Endpoints command line tool as follows:

    ```
    appengine-java-sdk-x.x.x/bin/endpoints.sh get-discovery-doc --war=target/helloendpoints-1.0-SNAPSHOT \
    com.google.appengine.samples.helloendpoints.Greetings
    ```

    where `target/helloendpoints-1.0-SNAPSHOT` is the relative or absolute path to the target build directory containing `WEB-INF/appengine-web.xml` and the compiled backend classes:

    ![Where to invoke endpoints.sh](https://web.archive.org/web/20160424230828im_/https://cloud.google.com/appengine/docs/java/endpoints/images/target_layout_war.png)

3.  When successful, the tool displays a message similar to:

    ```
    INFO: Successfully processed /usr/local/directory/helloendpoints/target/helloendpoints-1.0-SNAPSHOT/WEB-INF/appengine-web.xml
    API Discovery Document written to ./helloworld-v1-rest.discovery
    ```

If you are getting the discovery doc for use with an iOS client, see [Using Endpoints in an iOS Client](https://web.archive.org/web/20160424230828/https://cloud.google.com/appengine/docs/java/endpoints/consume_ios#Java_Compiling_the_client_library_generator_and_generating_your_library) for information on using the discovery document.

## Generating a client library bundle from a backend API

The Endpoints command line tool can be used to generate a Maven client bundle, a Gradle client bundle, or a bundle that contains only the dependency libraries and source jar. This bundle is the Java library be used in your client to call the backend API; it provides your client with all of the Google API client capabilities, including OAuth.

If you are using the client library with an Android app, we recommend that you use a Gradle client bundle.

### How to generate the client library bundle

Before starting, note two things about the instructions provided below:

-   The instructions use the `endpoints.sh` command for Linux and Mac OSX. If you use Windows, use `endpoints.cmd` instead. The options and invocations are the same for both versions of this tool.
-   The instructions are for a Maven project. If you use Eclipse, the project layout will be slightly different.

To generate a client library bundle:

1.  If you haven't already done so, build the backend API.

2.  Invoke the Endpoints command line tool similarly to the following:

    ```
    appengine-java-sdk-x.x.x/bin/endpoints.sh get-client-lib --war=target/helloendpoints-1.0-SNAPSHOT \
    -bs gradle com.google.appengine.samples.helloendpoints.Greetings
    ```

    where we are building a Gradle bundle and where `target/helloendpoints-1.0-SNAPSHOT` is the relative or absolute path to the the target build directory containing `WEB-INF/appengine-web.xml` and the compiled backend classes:

    ![Where to invoke endpoints.sh](https://web.archive.org/web/20160424230828im_/https://cloud.google.com/appengine/docs/java/endpoints/images/target_layout_war.png)

3.  When successful, the tool displays a message similar to:

    ```
    INFO: Successfully processed /usr/local/directory/helloendpoints/target/helloendpoints-1.0-SNAPSHOT/WEB-INF/appengine-web.xml
    API client library written to ./helloworld-v1-java.zip
    ```

The client library bundle is written to the current directory unless you specify some other output directory using the [`output` option](#options).

### What is supported in the client library bundle

The following are supported in the Java client bundle:

-   Java 5 and higher: standard (SE) and enterprise (EE).
-   [Android](https://web.archive.org/web/20160424230828/https://code.google.com/p/google-api-java-client/wiki/Android) 1.5 and higher.
-   [Google App Engine](https://web.archive.org/web/20160424230828/https://code.google.com/p/google-api-java-client/wiki/GoogleAppEngine).
-   The bundle also contains a `readme.html` file that provides details on using the client library, which dependent jars are needed for each application type (web, installed, or Android application), and so forth.

## Command line syntax for the Endpoints tool

Before you use the Endpoints command line tool, you need to build your backend project because the tool requires compiled binaries. You can optionally supply the `--war=` option pointing to the build target output directory containing the `WEB-INF/appengine-web.xml` file and compiled java classes if you don't want to use the default (`--war="./war"`).

The basic syntax is as follows:

```
appengine-java-sdk-x.x.x/bin/endpoints.sh <command> <options> [class-name]
```

where:

-   `<command>` is either `get-client-lib` or `get-discovery-doc`.
-   `<options>`, if supplied, is one or more items shown in the [Options table](#options).
-   `[class name]` is the full class name of your API.

**Note:** If your API is implemented from multiple classes, specify each of them separated by a space.

## Options

You can use the following options:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Option Name</th>
<th>Description</th>
<th>Example</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>build-system</code></td>
<td>Lets you specify which type of client bundle should be produced. Specify <code>gradle</code> for a Gradle client bundle for Android, <code>maven</code> for a Maven client bundle, or <code>default</code> (or simply omit this option) for a bundle that contains only the dependency libraries and source jar.</td>
<td><code>--build-system=gradle</code><br />
<code>-bs gradle</code></td>
</tr>
<tr class="even">
<td><code>classpath</code></td>
<td>Lets you specify the service class or classes from a path other than the default <code>&lt;war-directory&gt;/WEB-INF/libs</code> and <code>war-directory&gt;/WEB-INF/classes</code> <em>in your build target directory</em>, where <code>war-directory</code> is the directory specified in the <code>war</code> option, or simply <code>war</code> if that option is not supplied.</td>
<td><code>--classpath=/usr/lib com.google.devrel.samples.ttt.spi.BoardV1</code><br />
<code>-cp /usr/lib com.google.devrel.samples.ttt.spi.BoardV1</code></td>
</tr>
<tr class="odd">
<td><code>format</code></td>
<td>For <code>get-discovery-doc</code> only. Sets the format of the generated discovery document.<br />
Possible values: <code>rest</code> or <code>rpc</code>.<br />
Default is <code>rest</code>.</td>
<td><code>--format=rpc</code><br />
<code>-f rpc</code></td>
</tr>
<tr class="even">
<td><code>war</code></td>
<td>Sets the path to the build target directory <code>WEB-INF</code> that contains <code>appengine-web.xml</code> and other metadata.<br />
Default: <code>./war</code>.</td>
<td><code>--war=target/helloendpoints-1.0-SNAPSHOT</code><br />
<code>-w target/helloendpoints-1.0-SNAPSHOT</code></td>
</tr>
<tr class="odd">
<td><code>output</code></td>
<td>Sets the directory where the output will be written to.<br />
Default: the directory the tool is invoked from.</td>
<td><code>--output=/mydir</code><br />
<code>-o /mydir</code></td>
</tr>
</tbody>
</table>

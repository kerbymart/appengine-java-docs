# Generating Client Libraries

  

1.  [Generating a client library for Android using Maven](#generating_a_client_library_for_android_using_maven)
2.  [Generating a client library for Android using endpoints.sh](#generating_a_client_library_for_android_using_endpointssh)
3.  [Generating a discovery doc for iOS using Maven](#generating_a_discovery_doc_for_ios_using_maven)
4.  [Generating a discovery doc for iOS using endpoints.sh](#generating_a_discovery_doc_for_ios_using_endpointssh)

In order for an Android client to access a backend API, the client must use a Java client library generated from the backend API.

To generate a client library from your backend API, you must have already [annotated](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/annotations) the backend API code. (See the [backend API tutorial](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend) for more information. The following instructions assume that you have already finished annotating your backend API.

## Generating a client library for Android using Maven

If you created your backend API in a Maven project using the App Engine Maven artifacts as described in the [backend API tutorial](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend), you generate the client library as follows:

1.  Invoke the following command in the directory containing your project `pom.xml` file:

    ```
    mvn appengine:endpoints_get_client_lib
    ```

2.  Locate the generated client library source directory and files, which are inside your project root directory at this subdirectory location: `/target/endpoints-client-libs/<yourProjectName>` and change directory to this location.

3.  Invoke Maven to build the client library:

    ```
    mvn install
    ```

4.  This builds the required class files and creates the JAR file you need to add to your client project. The JAR file is located inside your current directory at this subdirectory location:

    ```
    /target/<yourProjectVersion>.<versionString>-rc-SNAPSHOT.jar
    ```

5.  Add the client library JAR to the Android app as described in [Using Endpoints in an Android Client](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/consume_android).

**Important:** if you modify the API backend, you must repeat the above steps in order to update the client.

## Generating a client library for Android using `endpoints.sh`

If don't use Maven, you can generate the client libray using the `endpoints.sh` command line tool:

1.  Generate the library following the instructions provided in [Using the Endpoints Command Line Tool](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/endpoints_tool).

2.  Remember to repeat the client library generation when you modify the backend API.

3.  Add the client library to the Android app as described in [Using Endpoints in an Android Client](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/consume_android).

## Generating a discovery doc for iOS using Maven

To generate the discovery doc for iOS using Maven:

1.  Invoke the following command in the directory containing your project `pom.xml` file:

    ```
    mvn appengine:endpoints_get_discovery_doc
    ```

2.  Note the location of the generated discovery doc. The console will display a message similar to the following:

    ```
    API Discovery Document written to
    /<your-path>/<your-projectdir>/target/generated-sources/appengine-endpoints/WEB-INF/helloworld-v1-rpc.discovery
    ```

Supply the generated discovery doc to the OS X [library generator](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/consume_ios#Java_Compiling_the_client_library_generator_and_generating_your_library).

## Generating a discovery doc for iOS using `endpoints.sh`

For iOS, you need to generate the discovery document for RPC. To generate the discovery doc using the command line tool:

1.  Change directory to the parent directory of your project's `/war` directory. Alternatively, use the [`--war`](#options) option to specify the location of `war`.

2.  Invoke the Endpoints command line tool similar to the following:

    ```
    appengine-sdk/bin/endpoints.sh get-discovery-doc --format=rpc \
        com.google.devrel.samples.ttt.spi.BoardV1
    ```

    where the format is set explicitly to `rpc` (since the commmand line tool default is REST), and where you replace `com.google.devrel.samples.ttt.spi.BoardV1` with your own API class. (If your API is implemented across several classes, list each class separated by a space.)

3.  When successful, the tool displays a message similar to:

    ```
    INFO: Successfully processed ./war/WEB-INF/appengine-web.xml
    API Discovery Document written to ./tictactoe-v1-rest.discovery
    ```

    The discovery document is written to the current directory unless you specify some other output directory using the [`output` option](#options).

Supply the generated discovery doc to the OS X [library generator](https://web.archive.org/web/20160424225931/https://cloud.google.com/appengine/docs/java/endpoints/consume_ios#Java_Compiling_the_client_library_generator_and_generating_your_library).

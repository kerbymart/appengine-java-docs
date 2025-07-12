# Local Unit Testing for Java

  

[Unit testing](https://web.archive.org/web/20160424225837/https://en.wikipedia.org/wiki/Unit_testing) allows you to check the quality of your code after you've written it, but you can also use unit testing to improve your development process as you go along. Instead of writing tests after you finish developing your application, consider writing the tests as you go. This helps you design small, maintainable, reusable units of code. It also makes it easier for you to test your code thoroughly and quickly.

When you do local unit testing, you run tests that stay inside your own development environment without involving remote components. App Engine provides testing utilities that use local implementations of datastore and other [App Engine services](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/apis). This means you can exercise your code's use of these services locally, without deploying your code to App Engine, by using service stubs.

A service stub is a method that simulates the behavior of the service. For example, the datastore service stub shown in [Writing Datastore and Memcache Tests](#Java_Writing_Datastore_and_memcache_tests) allows you to test your datastore code without making any requests to the real datastore. Any entity stored during a datastore unit test is held in memory, not in the datastore, and is deleted after the test run. You can run small, fast tests without any dependency on datastore itself.

This document provides some information about setting up a testing framework, then describes how to write unit tests against several local App Engine services.

1.  [Setting up a testing framework](#Java_Setting_up_a_testing_framework)
2.  [Introducing the Java testing utilities](#Java_Introducing_the_Java_testing_utilities)
3.  [Writing Datastore and memcache tests](#Java_Writing_Datastore_and_memcache_tests)
4.  [Writing High Replication Datastore tests](#Java_Writing_High_Replication_Datastore_tests)
5.  [Writing task queue tests](#Java_Writing_task_queue_tests)
6.  [Writing deferred task tests](#Java_Writing_deferred_task_tests)
7.  [Writing local service capabilities tests](#Java_Writing_local_service_capabilities_tests)
8.  [Writing tests for other services](#Java_Writing_tests_for_other_services)
9.  [Writing tests with authentication expectations](#Java_Writing_tests_with_authentication_expectations)

## Setting up a testing framework

Even though the SDK's testing utilities are not tied to any specific framework, we'll use [JUnit](https://web.archive.org/web/20160424225837/http://www.junit.org/) for our examples so you have something concrete and complete to work from. Before you begin writing tests you'll need to add the appropriate [JUnit 4 JAR](https://web.archive.org/web/20160424225837/https://github.com/junit-team/junit/wiki/Download-and-Install) to your testing classpath. Once that's done you're ready to write a very simple JUnit test.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/MyFirstTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/MyFirstTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/MyFirstTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/MyFirstTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

If you're running Eclipse 3.5, select the source file of the test to run. Select the **Run** menu &gt; **Run As** &gt; **JUnit Test**. The results of the test appear in the Console window.

For information on running your tests with Apache Ant, please read the [Ant JUnit Task](https://web.archive.org/web/20160424225837/http://ant.apache.org/manual/Tasks/junit.html) documentation.

**Note:** It is good practice to store your unit tests in a different location than your application code. Also avoid deploying JUnit and other testing packages with your application.

## Introducing the Java testing utilities

`MyFirstTest` demonstrates the simplest possible test setup, and for tests that have no dependency on App Engine APIs or local service implementations, you may not need anything more. However, if your tests or code under test have these dependencies, add the following JAR files to your testing classpath:

-   `${SDK_ROOT}/lib/impl/appengine-api.jar`
-   `${SDK_ROOT}/lib/impl/appengine-api-labs.jar`
-   `${SDK_ROOT}/lib/impl/appengine-api-stubs.jar`
-   `${SDK_ROOT}/lib/impl/appengine-tools-sdk.jar`

These JARs make the runtime APIs and the local implementations of those APIs available to your tests.

App Engine services expect a number of things from their execution environment, and setting these things up involves a fair amount of boilerplate code. Rather than set it up yourself, you can use the utilities in the `com.google.appengine.tools.development.testing` package. To use this package, add the following JAR file to your testing classpath:

-   `${SDK_ROOT}/lib/testing/appengine-testing.jar`

Take a minute to browse the [javadoc](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/localunittesting/javadoc) for the `com.google.appengine.tools.development.testing` package. The most important class in this package is [LocalServiceTestHelper](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/localunittesting/javadoc/com/google/appengine/tools/development/testing/LocalServiceTestHelper), which handles all of the necessary environment setup and gives you a top-level point of configuration for all the local services you might want to access in your tests.

To write a test that accesses a specific local service:

-   Create an instance of [`LocalServiceTestHelper`](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/localunittesting/javadoc/com/google/appengine/tools/development/testing/LocalServiceTestHelper) with a [`LocalServiceTestConfig`](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/localunittesting/javadoc/com/google/appengine/tools/development/testing/LocalServiceTestConfig) implementation for that specific local service.
-   Call `setUp()` on your `LocalServiceTestHelper` instance before each test and `tearDown()` after each test.

## Writing Datastore and memcache tests

The following example tests the use of the [datastore](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/datastore/index) service.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalDatastoreTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/LocalDatastoreTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalDatastoreTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalDatastoreTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

In this example, `LocalServiceTestHelper` sets up and tears down the parts of the execution environment that are common to all local services, and `LocalDatastoreServiceTestConfig` sets up and tears down the parts of the execution environment that are specific to the local datastore service. If you read the [javadoc](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/localunittesting/javadoc) you'll learn that this involves configuring the local datastore service to keep all data in memory (as opposed to flushing to disk at regular intervals) and wiping out all in-memory data at the end of every test. This is just the default behavior for a datastore test, and if this behavior isn't what you want you can change it.

#### Changing the example to access memcache instead of datastore

To create a test that accesses the local memcache service, you can use the code that is shown above, with a few small changes.

Instead of importing classes related to datastore, import those related to memcache. You still need to import `LocalServiceTestHelper`.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

Change the name of the class that you are creating, and change the instance of `LocalServiceTestHelper`, so that they are specific to memcache.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

And finally, change the way you actually run the test so that it is relevant to memcache.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalMemcacheTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

As in the datastore example, the `LocalServiceTestHelper` and the service-specific `LocalServiceTestConfig` (in this case `LocalMemcacheServiceTestConfig`) manage the execution environment.

## Writing High Replication Datastore tests

If your app uses the High Replication Datastore (HRD), you may want to write tests that verify your application's behavior in the face of eventual consistency. `LocalDatastoreServiceTestConfig` exposes options that make this easy:

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalHighRepDatastoreTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/LocalHighRepDatastoreTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalHighRepDatastoreTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalHighRepDatastoreTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

By setting the unapplied job percentage to 100, we are instructing the local datastore to operate with the maximum amount of eventual consistency. Maximum eventual consistency means writes will commit but always fail to apply, so global (non-ancestor) queries will consistently fail to see changes. This is of course not representative of the amount of eventual consistency your application will see when running in production, but for testing purposes, it's very useful to be able to configure the local datastore to behave this way every time.

If you want more fine-grained control over which transactions fail to apply, you can register your own `HighRepJobPolicy`:

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalCustomPolicyHighRepDatastoreTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/LocalCustomPolicyHighRepDatastoreTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalCustomPolicyHighRepDatastoreTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/LocalCustomPolicyHighRepDatastoreTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

The testing APIs are useful for verifying that your application behaves properly in the face of eventual consistency, but please keep in mind that the local High Replication read consistency model is an approximation of the production High Replication read consistency model, not an exact replica. In the local environment, performing a `get()` of an `Entity` that belongs to an entity group with an unapplied write will always make the results of the unapplied write visible to subsequent global queries. In production this is not the case.

## Writing task queue tests

Tests that use the local task queue are a bit more involved because, unlike datastore and memcache, the task queue API does not expose a facility for examining the state of the service. We need to access the local task queue itself to verify that a task has been scheduled with the expected parameters. To do this, we need `com.google.appengine.api.taskqueue.dev.LocalTaskQueue`.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/TaskQueueTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

Notice how we ask the `LocalTaskqueueTestConfig` for a handle to the local service instance, and then we investigate the local service itself to make sure the task was scheduled as expected. All `LocalServiceTestConfig` implementations expose a similar method. You may not always need it, but sooner or later you'll be glad it's there.

#### Setting the `queue.xml` configuration file

The task queue test libraries allow any number of `queue.xml` configurations to be specified on a per-LocalServiceTestHelper basis via the `LocalTaskQueueTestConfig.setQueueXmlPath` method. Currently any queue's rate limit settings are ignored by the local development server. It is not possible to run simultaneous tasks at a time locally.

For example, a project may need to test against the `queue.xml` file that will be uploaded and used by the App Engine application. Assuming that the `queue.xml` file is in the standard location, the above sample code could be modified as follows to grant the test access to the queues specified in the `src/main/webapp/WEB-INF/queue.xml` file:

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

Modify the path to the `queue.xml` file to fit your project's file structure.

Use the `QueueFactory.getQueue` method to access queues by name:

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/TaskQueueConfigTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

## Writing deferred task tests

If your application code uses [Deferred Tasks](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/taskqueue/DeferredTask), the Java Testing Utilities make it easy to write an integration test that verifies the results of these tasks.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/DeferredTaskTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/DeferredTaskTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/DeferredTaskTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/DeferredTaskTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

As with our first Local Task Queue example, we are using a `LocalTaskqueueTestConfig`, but this time we are initializing it with some additional arguments that give us an easy way to verify not just that the task was scheduled but that the task was executed: We call `setDisableAutoTaskExecution(false)` to tell the Local Task Queue to automatically execute tasks. We call `setCallbackClass(LocalTaskQueueTestConfig.DeferredTaskCallback.class)` to tell the Local Task Queue to use a callback that understands how to execute Deferred tasks. And finally we call `setTaskExecutionLatch(latch)` to tell the Local Task Queue to decrement the latch after each task execution. This configuration allows us to write a test in which we enqueue a Deferred task, wait until that task runs, and then verify that the task behaved as expected when it ran.

## Writing local service capabilities tests

Capabilities testing involves changing the status of some service, such as datastore, blobstore, memcache, and so forth, and running your application against that service to determine whether your application is responding as expected under different conditions. The capability status can be changed using the [LocalCapabilitiesServiceTestConfig](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/localunittesting/javadoc/com/google/appengine/tools/development/testing/LocalCapabilitiesServiceTestConfig) class.

**Note:** The capability status can also be changed in the dev app server console, or via command line options. For complete details, see [Capabilities Service Test Configuration](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/capabilities).

The following code snippet changes the capability status of the datastore service to disabled and then runs a test on the datastore service. (You can substitute other services for datastore as needed.)

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/ShortTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/ShortTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/ShortTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/ShortTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

The sample test first creates a `Capability` object initialized to datastore, then creates a `CapabilityStatus` object set to DISABLED. The `LocalCapabilitiesServiceTestConfig` is created with the capability and status set using the `Capability` and `CapabilityStatus` objects just created.

The `LocalServiceHelper` is then created using the `LocalCapabilitiesServiceTestConfig` object. Now that the test has been set up, the `DatastoreService` is created and a Query is sent to it to determine whether the test generates the expected results, in this case, a `CapabilityDisabledException`.

## Writing tests for other services

Testing utilities are available for blobstore and other App Engine services. For a list of all the services that have local implementations for testing, see the [`LocalServiceTestConfig`](https://web.archive.org/web/20160424225837/https://cloud.google.com/appengine/docs/java/tools/localunittesting/javadoc/com/google/appengine/tools/development/testing/LocalServiceTestConfig) documentation.

## Writing tests with authentication expectations

This example shows how to write tests that verify logic that uses UserService to determine if a user is logged on or has admin privileges.

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/AuthenticationTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/AuthenticationTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/AuthenticationTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/AuthenticationTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

In this example, we're configuring the `LocalServiceTestHelper` with the `LocalUserServiceTestConfig` so we can use the `UserService` in our test, but we're also configuring some authentication-related environment data on the `LocalServiceTestHelper` itself.

This example shows how to write tests that verify logic using OAuthService:

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/OAuthTest.java" target="_blank" style="color: white;">unittests/src/test/java/com/google/appengine/samples/OAuthTest.java</a>

<a href="https://web.archive.org/web/20160424225837/https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/OAuthTest.java" class="button" target="_blank" data-track-type="github" data-track-name="gitHubViewButton" data-track-metadata-link-destination="https://github.com/GoogleCloudPlatform/java-docs-samples/blob/master/unittests/src/test/java/com/google/appengine/samples/OAuthTest.java">View on GitHub</a>

\[This section requires a browser that supports JavaScript and iframes.\]

In this example, we're configuring the `LocalServiceTestHelper` with the `LocalUserServiceTestConfig` so we can use the `OAuthService`.

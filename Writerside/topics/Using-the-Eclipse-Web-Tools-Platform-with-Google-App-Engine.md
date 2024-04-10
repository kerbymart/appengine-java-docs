# Using the Eclipse Web Tools Platform with Google App Engine

  

1.  [Configuration](#configuration)
    1.  [Checking the environment](#checking_the_environment)
    2.  [Creating a new server](#creating_a_new_server)
2.  [Creating a project](#creating_a_project)
    1.  [Dynamic web project](#dynamic_web_project)
    2.  [Enterprise application](#enterprise_application)
    3.  [Importing an existing Maven project](#importing_an_existing_maven_project)
    4.  [Adding web project classes to another project's classpath](#adding_web_project_classes_to_another_projects_classpath)
3.  [Running the project on the server](#running_the_project_on_the_server)
4.  [Debugging](#debugging)
5.  [Monitoring web application network data](#monitoring_web_application_network_data)
6.  [Deploying a project to Google App Engine](#deploying_a_project_to_google_app_engine)
7.  [Using the JPA facet](#using_the_jpa_facet)
8.  [Working with Cloud Endpoints](#working_with_cloud_endpoints)

## Configuration

### Checking the environment

Be sure that the Web Tools Platform (WTP) is installed on your system. If it isn't, follow the [instructions](https://web.archive.org/web/20160424225938/http://wiki.eclipse.org/WTP_FAQ#How_do_I_install_WTP.3F) at the Eclipse website.

During startup, the WTP plugin looks for the expected Web Tools Platform (WTP) runtime instance.

-   If the WTP runtime instance is not found, a new one is created. The WTP instance is automatically configured to use the default Java Runtime Environment and default Google App Engine SDK.
-   If the Google App Engine SDK is not found, the WTP runtime is created, but you'll get an error message. In this case, install the Google App Engine SDK and configure the WTP runtime manually, either by editing the invalid runtime or creating a new one in Eclipse.

To create a new runtime in Eclipse:

1.  Open **Eclipse** &gt; **Preferences** &gt; **Server** &gt; **Runtime Environments**.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_0.png)

2.  Click **Edit...** and select or configure the Google App Engine SDK.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_1.png)

    **Note:** If you don't have the Google App Engine SDK version installed, click **Configure SDKs...** and add a Google App Engine SDK.

3.  Click **Finish** and then **OK**.

### Creating a new server

To create a new server:

1.  Switch to the Java EE perspective, if necessary (**Window** &gt; **Open perspective** &gt; **Other...** &gt; **Java EE**), and open the **Servers** view.

2.  RIght-click and select **New** &gt; **Server** (or click the **new server** link).

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_2.png)

    The **New Server** wizard looks like this:

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_3.png)

3.  Navigate to the **Google** category and select **Google App Engine**.

4.  Choose a name for the server as desired, and make sure the **Server runtime environment** is set to Google App Engine.

5.  Click **Next** to display the Server instance configuration page, and review the settings.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_4.png)

    The **Server Port** is the port number on which the Development Server will listen for connections.

    The **Auto scan** setting determines whether the SDK automatically reloads the application when resources change. When this feature is enabled, the Development Server scans the published project at a regular interval, and if it detects changes, it reloads the application without the need to manually restart. The default scan interval is 5 seconds. You can enter zero or a negative value to disable this feature.

    The **HRD** option configures the local datastore to simulate the consistency model of the [High Replication Datastore](https://web.archive.org/web/20160424225938/https://developers.google.com/appengine/docs/java/tools/devserver#Using_the_Datastore). Setting this value to zero disables the HRD.

6.  Click **Finish** to create the server.

Once the server is created, you won't be able to start it until you first associate it with a project. See [Running the project on the server](#running_the_project_on_the_server) for more information.

**Note:** In Eclipse Web Tools, a project is sometimes called a "module" or "Web module", which is not the same as an [App Engine module](https://web.archive.org/web/20160424225938/https://cloud.google.com/appengine/docs/java/modules/).

If you double-click on the server node in the **Servers** view, the server editor window comes up. You can fine-tune some parameters, such as **Publishing**. By default, **Publishing** is set to automatic, with a zero-second delay interval. This means that the plugin performs an automatic redeployment of the web application project to the local Development Server as soon as you save a change to a resource in the project. It can be a change to a servlet, a JSP, or any class or web artifact in the project.

![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_5.png)

If automatic scanning and reloading is enabled for the Development Server, the changes are applied after publishing is done. In other words, Eclipse detects the changes in your project, then builds the project, and then publishes the project to the Development Server (assuming the publish settings are set to automatic). After that, changes are reloaded by the Development Server at the defined scan cycle time, which is set to 5 seconds by default.

If you need to pass additional VM arguments to the server launch configuration, you can add them here. Don't edit the launch configuration directly; your input will be ignored!

## Creating a project

This section provides three examples of creating an App Engine project:

-   creating a dynamic web project
-   creating an enterprise application project
-   importing an existing Maven project

### Dynamic web project

To create a dynamic web project:

1.  Open the **New** wizard using the menu or by typing Ctrl-N (Cmd-N on a Mac) and selecting **Dynamic web project**.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_6.png)

    If you don't have any other runtimes defined, the Google App Engine runtime is pre-selected. The runtime sets up the project classpath to include the required libraries of the Google App Engine SDK.

2.  Click **Finish** to create the project with default settings, or click **Next** to tune additional setting for the Google App Engine facet or other facets. For this example, we'll ignore the other facets and skip to the **Google App Engine Wizard** page.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_7.png)

    **Package** is the default package name that is used to generate project contents. Change it to any desired name.

    The **Deployment** section allows to configure your Application ID and the version to deploy to. You can omit these while you're working with the local Development server, but you have to provide a valid Application ID before deploying to App Engine. Click **My applications** and **Existing versions** to visit the Google App Engine site, where you can configure your applications.

    The **Deployment Options** section contains options which affect the deployment procedure. See [Uploading and Managing a Java App](https://web.archive.org/web/20160424225938/https://developers.google.com/appengine/docs/java/tools/uploadinganapp) for details.

    By default, the **Sample Code** feature is set to generate sample code. Uncheck this option to generate an empty project with the required project structure, including `appengine-web.xml` and `web.xml`. The generated sample is a servlet which is registered in the web.xml file.

    If you want to add any Google API libraries (such as the Google Calendar API) to your new project, select the checkbox labeled **Open Google API Import Wizard upon project creation**. You can add Google API libraries to your project later, using the Google context menu.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_8.png)

The new project is now created, and its classpath is set to develop using the Google App Engine SDK. It is fully capable of incorporating any servlet or JSP, or any other class or web artifact.

### Enterprise application

Google App Engine supports project development for enterprise applications that use the Enterprise ARchive (EAR) format.

To create a new enterprise application project:

1.  Open the **New** wizard using the menu or by typing Ctrl-N (Cmd-N on a Mac) and selecting **Enterprise Application Project**.

2.  In the wizard, select the Google App Engine SDK as the **Target runtime**.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_9.png)

3.  The next page of the wizard allows you to add Web modules into the EAR project.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_10.png)

    The page lists all available projects in the workspace that are suitable to add to the EAR. Since we created a web project named MyFirstProject, we select it to be added to the EAR. If you have to create a new project to be added to the EAR, click **New Module...**.

    **Note:** Google App Engine requires the `application.xml` deployment descriptor, and this file is created regardless of how you set the option **Generate application.xml deployment descriptor**.

    **Note:** The App Engine does not support Enterprise Java Bean (EJB) Module projects.

    The next wizard page is the Google App Engine page.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_11.png)

    Notice that the EAR project has Application ID and deployment options, just as in a web project. During deployment, the Application ID and deployment options previously set in the child web project are ignored, and only the settings in the EAR project are used.

4.  Click **Finish**. You now have your new enterprise application project for Google App Engine.

You can add as many dynamic web projects as needed into your EAR project. To distinguish different Web modules within the same EAR project, use a module deployment descriptor tag. You can set this tag by opening the project properties and selecting the **Google App Engine** &gt; **Deployment** property page.

![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_12.png)

If you have only one Web module in your EAR application, then you don't need to provide a module ID; a value of `default` is used.

<span id="maven"></span>

### Importing an existing Maven project

An alternative to creating a new dynamic web project is to import an existing [Maven](https://web.archive.org/web/20160424225938/http://maven.apache.org/) project. When you do this, Eclipse uses Maven to manage dependencies and builds. (See [Using Apache Maven](https://web.archive.org/web/20160424225938/https://cloud.google.com/appengine/docs/java/tools/maven) for more information.) The directory at the root of the Maven project must have a [`pom.xml`](https://web.archive.org/web/20160424225938/http://maven.apache.org/guides/introduction/introduction-to-the-pom.html) file, and the `<plugins>` element in the `pom.xml` file must include a [`<plugin>` element for `appengine-maven-plugin`](https://web.archive.org/web/20160424225938/https://cloud.google.com/appengine/docs/java/tools/maven#adding_the_app_engine_maven_plugin_to_an_existing_maven_project).

**Note:** The directory at the root of the Maven project must *not* contain any of the following:

-   A subdirectory named `target`
-   A subdirectory named `.settings`
-   A file named `.classpath`
-   A file named `.project`

If any of these subdirectories or files exists, remove it manually before you import the project. The subdirectory or file should be created by the import process; if it is already present, it may interfere with the import process.

To import the existing App Engine project:

1.  Select the **File** menu &gt; **Import** &gt; **Maven** &gt; **Existing Maven Projects**.
2.  Click **Next**.
3.  Specify the root directory of the Maven project (the directory containing the `pom.xml` file). You can either type the path into the **Root Directory** text field or click the **Browse** button and use a file-selection dialog.
4.  Click **Finish**.

**Note:** Currently, only Maven WAR projects are supported, not Maven EAR projects.

### Adding web project classes to another project's classpath

For information on this use case, refer to the [Eclipse WTP FAQ](https://web.archive.org/web/20160424225938/http://wiki.eclipse.org/M2E-WTP_FAQ#How_do_I_add_my_web_project_classes_to_another_project.27s_classpath.3F).

<span id="runningonserver"></span>

## Running the project on the server

To run the project using the local Google App Engine Development Server:

1.  Select the **Run As** menu and then **Run on Server**. If you select a web project that is a child of the EAR project, the parent EAR project is selected to run on the Server.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_13.png)

2.  When prompted for a server to run this project, select the Google App Engine server that was created earlier. If you skipped the server creation, you can create the server now by selecting **Manually define a new server**.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_14.png)

3.  Click **Finish** to publish your project to the local Google App Engine Development Server. The server starts and an internal browser is opened and pointed to the local Development Server.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_15.png)

    At this moment the server is started and synchronized. While the application is running, applying changes to resources in the project causes the project to be re-published and picked up by the Development Server without manually restarting.

4.  Click the link **MyFirstProject** in the browser to call MyFirstProjectServlet on the Development Server, which outputs "Hello, World".

If you've created a server before, you can also start the server using the toolbar in the **Servers** view. **Run on Server** and the **Servers** view icon both create launch configurations, so both methods of launching are equivalent. The launch configuration is auto-generated and not intended to be edited by the user. If you need to pass additional VM arguments to the Development Server, use the Server Editor. (Access the editor by double-clicking on your server in the **Servers** view.)

In the case of a Maven project, you can also go to the **Run** menu and select **Run As** &gt; **1 Run on Server**. This takes you to a page where you can select a server. The first time you run the application, you should use the **Manually define a new server** option.

![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/run_on_server.png "=306x")

Feel free to change the server name if you like, but accept the remaining defaults and click **Finish.** On subsequent runs of the application, you should use the **Choose an existing server** option, make sure the server you defined for this application is highlighted, and click **Finish**.

![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/run_again_on_server.png "=306x")

## Debugging

You can install breakpoints on any Java class (including servlets, JPA, Web Services, etc.) of a web application, or on JSP pages which have Java code fragments.

To debug your application:

1.  Click the **Restart the server in debug mode** tool button in the **Servers** view (or select **Debug As** and **Debug on Server** from the menu, if you've never run the server before).

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_16.png)

2.  Once you get the server running in debug mode, connect your browser (internal or external) to the Google App Engine Development Server and click the **MyFirstProject** link to invoke the code being debugged.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_17.png)

When your breakpoint is reached you can take advantage of all the features of the Eclipse Java debugger.

## Monitoring web application network data

To enable monitoring:

1.  Run the server (in debug mode or run mode)

2.  Right-click on the server in the **Servers** view and select **Monitoring** &gt; **Properties**.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_18.png)

    The **Monitoring properties** dialog appears.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_19.png)

3.  Click **Add** and add the Development Server port to do monitoring:

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_20.png)

    Note that the actual port to be monitored is not the port on which the Development Server is running. By default, the port number equals the server port plus 1.

4.  After the port is added, click **Start** to begin monitoring the port.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_21.png)

    The status changes to **Started**, signifying that now it's possible to see the data flow between the Development Server and your browser.

5.  Open the TCP/IP Monitor view by selecting **Window** &gt; **Show View** and choosing **TCP/IP Monitor**.

6.  Access your application in a browser, using the port you specified when adding the monitor (not the actual Development Server port). You can then see the data in the TCP/IP Monitor view.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_22.png)

## Deploying a project to Google App Engine

After the project is associated with the server, you can deploy it to Google App Engine.

To deploy the project:

1.  Select the desired server in the **Servers** view and click the **Google App Engine** tool button (or select it using the context menu). Select the **Deploy to Remote Server** menu item.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_23.png)

2.  When prompted, log in to your Google account. If you didn't do it earlier, enter the Application ID in the project properties.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_24.png)

3.  Once you've entered a valid Application ID, click **OK** to deploy your project to Google App Engine, where it is available at `<your_app_id>.appspot.com`.

    Alternatively, for a Maven project, go to the **Run** menu and select **Run As** &gt; **5 Maven build...**. On the Edit Configuration panel that pops up, enter `appengine:update` in the **Goals** field and click **Run**.

## Using the JPA facet

During project creation, or later by editing project facets in project properties, it's possible to activate JPA functionality for your project.

To activate JPA functionality:

1.  Go to **Project Facets** in the project properties.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_25.png)

2.  Click the **Further configuration available...** link to display the JPA facet configuration page.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_26.png)

3.  Select **Google App Engine (Datanucleus)** as the platform and **Google App Engine SDK** as the JPA implementation. This sets up a classpath container in your project with the libs needed for using JPA. You'll also need to select the connection from the combo box in the Connection section, or click the **Add connection** link to create a new connection profile.

4.  Click **OK** to add the JPA facet to the project, add the classpath container, and enable the class enhancer for classes in the project.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_27.png)

Once the JPA facet is installed, you can configure classes which require enhancing in the project properties. The Google App Engine ORM property page appears under the JPA category after you've installed the JPA facet.

![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/image_28.png)

## Working with Cloud Endpoints

[Cloud Endpoints](https://web.archive.org/web/20160424225938/https://cloud.google.com/appengine/docs/java/endpoints/) work with WTP-based projects. They work similarly to how they're described in the [Google Plugin for Eclipse Endpoints](https://web.archive.org/web/20160424225938/https://cloud.google.com/eclipse/docs/cloud_endpoints) pages, except that you need to activate the J2EE perspective to see the Cloud Endpoints options for WTP-based projects.

To use Cloud Endpoints with your project:

1.  Activate the J2EE perspective, which enables you to see the **Google App Engine WTP** context menu. Refer to the [Eclipse documentation](https://web.archive.org/web/20160424225938/http://help.eclipse.org/kepler/index.jsp?topic=%2Forg.eclipse.platform.doc.user%2Ftasks%2Ftasks-28.htm) if you need more information on switching perspectives.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/google_app_engine_wtp_context_menu.png)

2.  From there, you can run Cloud Endpoint-related operations to generate an endpoint class and client libraries for your endpoints. (See [GPE documentation](https://web.archive.org/web/20160424225938/https://cloud.google.com/eclipse/docs/cloud_endpoints) for more information on endpoints.)

    If you'd like to generate a WTP-based App Engine backend for an existing Android Application, select **Generate App Engine Backend** from the **Google App Engine WTP** context menu (not the **Google** context menu). This takes you to the **Generate Backend Wizard**, customized for WTP.

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/gen_backend_wizard_1.png)

    ![image](https://web.archive.org/web/20160424225938im_/https://cloud.google.com/appengine/docs/images/webtoolsplatform/gen_backend_wizard_2.png)

**Note:** The **App Engine Connected Android Project** wizard does not generate a WTP-based backend. You need to generate the Android project first, and then generate the App Engine backend using the **Google App Engine WTP** context menu.

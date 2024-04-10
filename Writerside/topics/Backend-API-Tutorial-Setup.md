# Setup

To set up your environment:

1.  You must use Java 7. If you don't have Java 7, [download](https://web.archive.org/web/20160424225834/http://www.java.com/en/download/manual.jsp) and install it.

2.  Set your `JAVA_HOME` environment variable to the path of your JDK installation. If you are a `bash` user, the following considerations apply:

    *   For a typical Linux installation, add a line similar to the following to your `.bashrc` file:

    ```
    export JAVA_HOME=/usr/local/tools/java/jdk1.7.0_45.jdk
    ```
 
    *   If you use Mac OS X and the default Terminal app, your shell session doesn't load `.bashrc` by default. So you may need to add a line similar to the following to your `.bash_profile`:
        
    ```
    [ -r ~/.bashrc ] && source ~/.bashrc
    ```

    *   If you use Mac OS X but don't use the default terminal app, for example, you use a terminal management app such as tmux, you may need to add a line similar to the following line to your `.bashrc` file:
        
    ```
    export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home
    ```

        
3.  If you don't have Maven installed, [download and install](https://web.archive.org/web/20160424225834/http://maven.apache.org/) Maven.
    

Creating an App Engine backend for deployment
---------------------------------------------

You'll need to create a new Google Cloud Platform Console project for your backend, and you'll need to create client IDs for client.

### Creating a new project

1.  In the Cloud Platform Console, go to the [Projects page](https://web.archive.org/web/20160424225834/https://console.cloud.google.com/project).
2.  Select a project, or click **Create Project** to create a new Cloud Platform Console project.
3.  In the dialog, name your project. Make a note of your generated project ID.
4.  Click **Create** to create a new project.

### Creating OAuth 2.0 client IDs for the backend

1.  Open the [Credentials page](https://web.archive.org/web/20160424225834/https://console.cloud.google.com/project/_/apiui/credential/oauthclient) for your project, and select **Web application** as the application type.
2.  Fill out the form that is displayed:
    
    1.  Specify a name for the web client.
    2.  Specify `http://localhost:8080` in the textbox labeled **Authorized JavaScript origins**, because you'll be running this locally.
3.  Click **Create**.
    

Note the client ID that is generated. This is the client ID you need to use in your backend and in your client application. You can always return to the Credentials page to view the client ID.

[Creating a Simple Hello World Backend API >>](https://web.archive.org/web/20160424225834/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/backend/hello_world)

Except as otherwise noted, the content of this page is licensed under the [Creative Commons Attribution 3.0 License](https://web.archive.org/web/20160424225834/http://creativecommons.org/licenses/by/3.0/), and code samples are licensed under the [Apache 2.0 License](https://web.archive.org/web/20160424225834/http://www.apache.org/licenses/LICENSE-2.0). For details, see our [Site Policies](https://web.archive.org/web/20160424225834/https://cloud.google.com/site-policies).

Java is a registered trademark of Oracle and/or its affiliates.

Last updated January 14, 2016.
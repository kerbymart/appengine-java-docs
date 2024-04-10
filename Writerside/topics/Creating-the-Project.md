# Creating the Project

  

**Important:** The instructions in this tutorial assume that you are using a terminal on your local machine and running a browser on that same machine to test against the local development server.

To create a new project, you can use an App Engine-provided Maven App Engine artifact called `appengine-skeleton-archetype`, described below. The App Engine Maven artifact is what creates the project layout and files required to deploy and run on App Engine.

For more on using Maven with App Engine, see the [Using Apache Maven Guide](https://web.archive.org/web/20160424225841/https://cloud.google.com/appengine/docs/java/tools/maven). If you want to instead use Eclipse, see the [Google Plugin For Eclipse](https://web.archive.org/web/20160424225841/https://developers.google.com/eclipse/docs/getting_started), or [Apache Ant](https://web.archive.org/web/20160424225841/https://cloud.google.com/appengine/docs/java/tools/ant).

## Getting a project ID

To deploy your app to App Engine, you'll need a Google Cloud Platform project. To create a new project:

1.  Visit <a href="https://web.archive.org/web/20160424225841/https://console.cloud.google.com/" data-track-name="consoleLink" data-track-type="tutorial" data-track-metadata-position="body">Google Cloud Platform Console</a> in your web browser.
2.  If this is your first project, you'll see the *Getting Started* page: click **Create an empty project**. (After you create your first project, you might not see the *Getting Started* page, but may instead see a landing page listing your projects along with a **Create Project** button, which you can click to create new projects.)
3.  Supply the project name *Guestbook* and accept the project ID that is auto-generated for you.
4.  Click **Create**.

Make a note of the project ID. You will use the ID when you generate the starter files in the next step.

**Note:** Alternatively, you can use the ID from an existing project in the <a href="https://web.archive.org/web/20160424225841/https://console.cloud.google.com/" data-track-name="consoleLink" data-track-type="tutorial" data-track-metadata-position="note">Google Cloud Platform Console</a> if you want to use it for this project. <span class="expandable inline" title="Expand to show additional information"></span>

**Important:** The name you use must be between 4 and 30 characters. When you type the name, the form will suggest a project ID, which you can edit. The project ID you use must be between 6 and 30 characters, with a lowercase letter as the first character. You can use a dash, lowercase letter, or digit for the remaining characters, but the last character cannot be a dash. Also, you should be aware that some resource identifiers (such as project IDs) might be retained beyond the life of your project. For this reason, avoid storing sensitive information in resource identifiers.

<span class="expand-control once">...see naming guidelines</span>

## Generating the starter files

Change to a directory where you want to build your project, then invoke Maven as follows, replacing `your-app-id` with your project ID:

```
mvn archetype:generate -Dappengine-version=1.9.34 -Dapplication-id=your-app-id -Dfilter=com.google.appengine.archetypes:
```

The Maven `archetype:generate` command asks you a series of questions, then generates a set of starter files based on your answers. To match this tutorial, answer the questions as follows:

1.  From the artifact list, choose `1` to select the archetype `com.google.appengine.archetypes:appengine-skeleton-archetype`.

2.  For the archetype version, accept the default answer (press enter).

3.  When prompted to `Define value for property 'groupId'`, enter the desired namespace for your app. For this tutorial, specify: `com.example.guestbook`

4.  When prompted to `Define value for property 'artifactId'`, enter a name for the project. For this tutorial, specify: `guestbook`

5.  When prompted to `Define value for property 'version'`, accept the default value.

6.  When prompted to `Define value for property 'package'`, supply your preferred package name (or accept the default). The generated Java files will have the package name you specify here. For this tutorial, accept the default answer (`com.example.guestbook`).

7.  Confirm your choices by pressing enter (`Y`).

Maven generates your starter files in a directory named after the value you entered for the `artifactId`, in this case `guestbook`. The directory structure looks like this:

![Maven Project Layout](https://web.archive.org/web/20160424225841im_/https://cloud.google.com/appengine/docs/java/gettingstarted/images/maven_layout.png)

-   You'll add your own application Java classes to `src/main/java/...`
-   You'll configure your application using the file `src/main/webapp/WEB-INF/appengine-web.xml`
-   You'll configure your application deployment using the file `src/main/webapp/WEB-INF/web.xml`

We'll describe what to do inside these subdirectories later.

## Install dependencies

Change directory to the new `guestbook` that you just created:

```
cd guestbook
```

Run this command to install dependencies and do a clean build:

```
mvn clean install
```

Wait for the project to build. When the project successfully finishes you will see a message similar to this one:

```
[INFO] --------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] --------------------------------------------------
[INFO] Total time: 1:16.656s
[INFO] Finished at: Mon Apr 29 11:42:22 PDT 2015
[INFO] Final Memory: 16M/228M
[INFO] --------------------------------------------------
```

------------------------------------------------------------------------

You are now ready to add application code and UI.

<a href="https://web.archive.org/web/20160424225841/https://cloud.google.com/appengine/docs/java/gettingstarted/ui_and_code" class="button">Adding Application Code and UI &gt;&gt;</a>

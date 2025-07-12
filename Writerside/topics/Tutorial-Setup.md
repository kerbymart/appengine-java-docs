# Tutorial Setup

  

**Note:** This tutorial uses Maven. As an alternative to Maven, you could instead use Eclipse and the [Google Plugin For Eclipse](https://web.archive.org/web/20160424230312/https://developers.google.com/eclipse/docs/getting_started), or you could use [Apache Ant](https://web.archive.org/web/20160424230312/https://cloud.google.com/appengine/docs/java/tools/ant).

To complete this tutorial, you'll need the proper versions of Java and [Maven](https://web.archive.org/web/20160424230312/http://maven.apache.org/). Maven is a useful build automation/packaging tool for Java projects that manages build execution and also dependencies.

## Required Java version

We recommend using Java 7, preferably the Enterprise Edition.

If you have Java 8, you must specify Java 7 as the source and the target. In Maven, you do this in the `pom.xml` file for your project, within the `maven-compiler-plugin` configuration settings:

```
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <version>3.2</version>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>7</source>
                <target>7</target>
            </configuration>
        </plugin>
```

This will result in an application you can deploy and run on production App Engine, since the production instances are Java 7. However, if you have Java 8 on your development machine, and you try to run this app locally using the development server, you'll be running on Java 8 and so you may experience some compatibility issues even with the application compiled for Java 7 as described. For local testing, we recommend the use of the Java 7 SDK.

### Downloading Java

If you don't have Java, follow these instructions to download the Java Development Kit (JDK) for Java version 7:

1.  [Download](https://web.archive.org/web/20160424230312/http://www.oracle.com/technetwork/java/javase/downloads/index.html) and install the JDK.

2.  Set your `JAVA_HOME` environment variable. If you use the `bash` shell:

    1.  For a typical Linux installation, add a line similar to the following to your `.bashrc` file:

        ```
        export JAVA_HOME=/usr/local/tools/java/<jdk_version>
        ```

        where `<jdk_version>` is your JDK version, for example, `jdk1.7.0_45.jdk`.

    2.  If you use Mac OSX and the default Terminal app, your shell session doesn't load `.bashrc` by default. So you may need to add a line similar to the following to your `.bash_profile`:

        ```
        [ -r ~/.bashrc ] && source ~/.bashrc
        ```

    3.  If you use Mac OSX but you use a different terminal application such as tmux, you may need to add a line similar to the following line to your `.bashrc` file:

        ```
        export JAVA_HOME=/Library/Java/JavaVirtualMachines/<jdk_version>/Contents/Home
        ```

        where `<jdk_version>` is your JDK version, for example, `jdk1.7.0_45.jdk`.

        Alternatively, if your OS supports [/usr/libexec/java\_home](https://web.archive.org/web/20160424230312/http://stackoverflow.com/questions/1348842/what-should-i-set-java-home-to-on-osx), you can request the current JDK version in your export command line, as follows:

        ```
        export JAVA_HOME=$(/usr/libexec/java_home -v 1.7)
        ```

## Required Maven version

This tutorial requires Apache Maven 3.1 or greater. To determine whether Maven is installed and which version you have, invoke the following command:

```
 mvn -v
```

This command should display a long string of information beginning with something like `Apache Maven 3.1.0`.

If you don't have a proper version of Maven installed:

1.  [Download](https://web.archive.org/web/20160424230312/http://maven.apache.org/download.cgi) Maven 3.1 or greater from the Maven website.
2.  [Install](https://web.archive.org/web/20160424230312/http://maven.apache.org/install.html) Maven 3.1 or greater on your local machine.

**Note:** You cannot use `apt-get install` to install Maven 3.1 or greater.

## The Google App Engine SDK and Maven

When you use Maven, you don't need to download the Java libraries from the Google App Engine SDK. Maven does that for you. You'll also use Maven to test your app locally and upload (deploy) it to production App Engine.

**Note:** After you complete the tutorial, you can learn more about the App Engine Maven plugin capabilities by visiting [App Engine Maven plugin goals](https://web.archive.org/web/20160424230312/https://cloud.google.com/appengine/docs/java/tools/maven#app_engine_maven_plugin_goals).

------------------------------------------------------------------------

<a href="https://web.archive.org/web/20160424230312/https://cloud.google.com/appengine/docs/java/gettingstarted/creating" class="button">Creating the Project &gt;&gt;</a>

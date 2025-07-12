# Using Apache Ant

  

If you are not using [Eclipse](https://web.archive.org/web/20160424225634/http://www.eclipse.org/) and [the Google Plugin for Eclipse](https://web.archive.org/web/20160424225634/https://developers.google.com/eclipse/docs/appengine), you'll probably want some other way to manage the process of building and testing your App Engine application. [Apache Ant](https://web.archive.org/web/20160424225634/http://ant.apache.org/) makes it easy to manage your project from the command line, or from other IDEs that work with Ant. The Java SDK includes a set of Ant macros to perform common App Engine development tasks, including starting the development server and uploading the app to App Engine.

This article describes an Ant build file useful for most projects. To skip the description and go directly to the complete file for copying and pasting, see [The Complete Build](#The_Complete_Build_File).

**Note:** If you are using Apache Ant with [JDO](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/datastore/jdo/)/[JPA](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/datastore/jpa/), please note that this article applies to JDO version 2.3 and JPA version 1.0. Upgrading to the latest (experimental) versions requires changes to the `build.xml` file that are not documented on this page. For more information, see [JDO upgrade instructions](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/datastore/jdo/overview-dn2#Migrating_to_Version_2.0_of_the_DataNucleus_Plugin) and [JPA upgrade instructions](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/datastore/jpa/overview-dn2#Migrating_to_Version_2.0_of_the_DataNucleus_Plugin).

## Installing Ant

If you do not already have Ant installed on your system, visit [the Apache Ant web site](https://web.archive.org/web/20160424225634/http://ant.apache.org/) to download and install it.

When Ant is installed and the `ant` command is on your command path, you can run the following command to verify that it works, and see which version is installed:

```
ant -version
```

## The Project Directory

As described in the [Getting Started Guide](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/gettingstarted), your App Engine project must produce a directory structure using the [WAR](https://web.archive.org/web/20160424225634/http://java.sun.com/j2ee/tutorial/1_3-fcs/doc/WCC3.html) standard layout for Java web applications. (WAR archive files are not yet supported by the SDK.)

For these instructions, we will put all project-related files in a single directory named `Guestbook/` (for the guest book project described in the Getting Started Guide, with a subdirectory named `src/` for Java source code and a subdirectory named `war/` for the finished application files. The build process will compile the source code, and put the compiled Java bytecode in the appropriate location under `war/`. You will create other files for the WAR directly in the `war/` directory.

The complete project directory should look like this:

```
Guestbook/
  src/
    ...Java source code...
    META-INF/
      ...other configuration...
  war/
    ...JSPs, images, data files...
    WEB-INF/
      ...app configuration...
      classes/
        ...compiled classes...
      lib/
        ...JARs for libraries...
```

To create this directory structure from the command prompt, use the following commands. If you are using Windows, change the forward slashes `/` in these paths to backslashes `\`. (Within the Ant file itself, paths using forward slashes `/` work for all operating systems.)

```
mkdir Guestbook
cd Guestbook
mkdir -p src/META-INF
mkdir -p war/WEB-INF/classes
mkdir -p war/WEB-INF/lib
```

## Creating the Build File

In the `Guestbook/` directory, create a file named `build.xml` with the following contents:

```
<project>
  <property name="sdk.dir" location="../appengine-java-sdk" />

  <import file="${sdk.dir}/config/user/ant-macros.xml" />

</project>
```

This build file doesn't do anything yet: it doesn't contain a "target" with instructions to perform tasks. The file defines two Ant properties, which we will use when we define targets and paths later.

The `sdk.dir` property is the path to the App Engine Java SDK, the `appengine-java-sdk` directory created when you unpacked the SDK Zip file. `"../appengine-java-sdk"` assumes this directory is in the parent directory above the project directory. Adjust this value, if necessary. (Paths are relative to the location of the `build.xml` file.)

The `<import>` element loads a set of Ant macros for App Engine development, included in the App Engine Java SDK. We will use several of these macros later.

## Defining the Class Path

When you compile your classes, the Java class path must contain the JARs for the App Engine API, and any other JARs you may have added to your project's `war/WEB-INF/lib/` directory (such as the DataNucleus JDO/JPA JARs). The application can access classes from JARs added to this directory. Additionally, the JARs in the SDK's `lib/shared/` directory will be on the class path.

Edit `build.xml` and add the following lines above the closing `</project>` tag to define a class path containing these JARs:

```
  <path id="project.classpath">
    <pathelement path="war/WEB-INF/classes" />
    <fileset dir="war/WEB-INF/lib">
      <include name="**/*.jar" />
    </fileset>
    <fileset dir="${sdk.dir}/lib">
      <include name="shared/**/*.jar" />
    </fileset>
  </path>
```

## Copying JARs

Your app must include `appengine-api-*.jar` (where `<i>*</i>` represents a version number of the API and SDK), a JAR included with the SDK, in the directory `war/WEB-INF/lib/`. App Engine checks for classes from this JAR to determine which version of the API your app is using. If your application uses the JDO or JPA interfaces to the App Engine datastore, the app must include the JDO/JPA implementation in `war/WEB-INF/lib/`. All of these JARs are in the SDK's `lib/user/` directory.

You could copy these JARs into the WAR manually. However, it's useful to make copying these JARs part of the build process. This ensures that your app is using the same versions of these interfaces as the ones included in the SDK, even if you upgrade the SDK at a later date.

All of the JARs must be in the `war/WEB-INF/lib/` directory, they cannot be in subdirectories. This Ant target will copy JARs recursively from `${sdk.dir}/lib/user/` using the `copy` task's `flatten="true"` attribute so all JARs end up in the same directory.

Edit `build.xml` and add the following lines above the closing `</project>` tag:

```
  <target name="copyjars"
      description="Copies the App Engine JARs to the WAR.">
    <copy
        todir="war/WEB-INF/lib"
        flatten="true">
      <fileset dir="${sdk.dir}/lib/user">
        <include name="**/*.jar" />
      </fileset>
    </copy>
  </target>
```

This creates a build target named `"copyjars"` that copies the JARs to the appropriate location. To test this target, make sure the current working directory is the project directory (`Guestbook/`), then run the following command:

```
ant copyjars
```

Check the contents of `war/WEB-INF/lib/` to make sure the JARs are there.

## Compiling Source Files

Edit `build.xml` and add the following lines above the closing `&lt/project>` tag:

```
  <target name="compile" depends="copyjars"
      description="Compiles Java source and copies other source files to the WAR.">
    <mkdir dir="war/WEB-INF/classes" />
    <copy todir="war/WEB-INF/classes">
      <fileset dir="src">
        <exclude name="**/*.java" />
      </fileset>
    </copy>
    <javac
        srcdir="src"
        destdir="war/WEB-INF/classes"
        classpathref="project.classpath"
        debug="on" />
  </target>
```

This defines a build target named `"compile"` that compiles all Java source files found in the `src/` directory and stores the compiled classes in `war/WEB-INF/classes/`. All other files found in `src/`, such as the `META-INF/` directory, are copied verbatim to `war/WEB-INF/classes/`. The target depends on `"copyjars"`, so both targets are built, if necessary.

To compile your project's source files, run the following command:

```
ant compile
```

## Turning all \*.class files into a single jar file

If you want to jar all the classes of your Ant project in a single jar file, you can follow [these instructions](https://web.archive.org/web/20160424225634/https://ant.apache.org/manual/Tasks/jar.html).

## Enhancing JDO Classes

This section describes how to use Ant to build your project with support for the Java Data Objects (JDO) interface to the datastore. If your project does not use the JDO interface, you can [proceed to the next section](#Running_the_Development_Server).

As described in [Getting Started Guide](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/gettingstarted) and the [datastore documentation](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/datastore), the JDO interface allows you to store instances of your classes in the datastore, and retrieve them later as objects. You describe how instances are to be stored and retrieved using Java annotations in the class definition.

In order for JDO to see the annotations, you must process the bytecode for your data classes after they have been compiled using a tool included with DataNucleus Access Platform, the JDO implementation distributed with App Engine. DataNucleus describes this process as "enhancing" the classes.

The App Engine SDK includes Ant macros for enhancing JDO data classes. The `<enhance_war>` macro enhances every JDO data class in the project, using the correct class path for the project. For more precise control, you can use the `<enhance>` macro.

Edit `build.xml` and add the following lines above the closing `</project>;` tag:

```
  <target name="datanucleusenhance" depends="compile"
      description="Performs JDO enhancement on compiled data classes.">
    <enhance_war war="war" />
  </target>
```

This target depends on `"compile"`, so building this target ensures that the classes have been compiled and are up to date before performing the enhancement task.

## Running the Development Server

You can run the development web server using an Ant macro. By making this target depend on the `"datanucleusenhance"` target (or the `"compile"` target if not using DataNucleus), you can build your project and start the server all with one easy command.

Edit `build.xml` and add the following lines above the closing `</project>` tag:

```
  <target name="runserver" depends="datanucleusenhance"
      description="Starts the development server.">
    <dev_appserver war="war" />
  </target>
```

You can give the development server arguments using attributes and an `<options>` element. For example, the following target starts the server using the port 8888, and enables remote Java debugging on port 9999:

```
  <target name="runserver" depends="datanucleusenhance"
      description="Starts the development server.">
    <dev_appserver war="war" port="8888" >
      <options>
        <arg value="--jvm_flag=-Xdebug"/>
        <arg value="--jvm_flag=-Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=9999"/>
        <arg value="--jvm_flag=-Ddatastore.default_high_rep_job_policy_unapplied_job_pct=20"/>
      </options>
    </dev_appserver>
  </target>
```

To build the project and run the server, use the following command:

```
ant runserver
```

To stop the server, hit Control-C.

## Uploading and Other AppCfg Tasks

You can define Ant tasks to upload the application to App Engine and perform other actions provided by [AppCfg](https://web.archive.org/web/20160424225634/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp).

The `<appcfg>` macro takes the name of the action as the `action` attribute, and the path to the project's WAR as the `war` attribute. The element can have optional `<options>` and `<args>` elements.

Edit `build.xml` and add the following lines:

```
  <target name="update" depends="datanucleusenhance"
      description="Uploads the application to App Engine.">
    <appcfg action="update" war="war" />
  </target>

  <target name="update_indexes" depends="datanucleusenhance"
      description="Uploads just the datastore index configuration to App Engine.">
    <appcfg action="update_indexes" war="war" />
  </target>

  <target name="rollback" depends="datanucleusenhance"
      description="Rolls back an interrupted application update.">
    <appcfg action="rollback" war="war" />
  </target>

  <target name="request_logs"
      description="Downloads log data from App Engine for the application.">
    <appcfg action="request_logs" war="war">
      <options>
        <arg value="--num_days=5"/>
      </options>
      <args>
        <arg value="logs.txt"/>
      </args>
    </appcfg>
  </target>
```

## The Complete Build File

Here is the complete `build.xml` file described by these instructions:

```
<project>
  <property name="sdk.dir" location="../appengine-java-sdk" />

  <import file="${sdk.dir}/config/user/ant-macros.xml" />

  <path id="project.classpath">
    <pathelement path="war/WEB-INF/classes" />
    <fileset dir="war/WEB-INF/lib">
      <include name="**/*.jar" />
    </fileset>
    <fileset dir="${sdk.dir}/lib">
      <include name="shared/**/*.jar" />
    </fileset>
  </path>

  <target name="copyjars"
      description="Copies the App Engine JARs to the WAR.">
    <copy
        todir="war/WEB-INF/lib"
        flatten="true">
      <fileset dir="${sdk.dir}/lib/user">
        <include name="**/*.jar" />
      </fileset>
    </copy>
  </target>

  <target name="compile" depends="copyjars"
      description="Compiles Java source and copies other source files to the WAR.">
    <mkdir dir="war/WEB-INF/classes" />
    <copy todir="war/WEB-INF/classes">
      <fileset dir="src">
        <exclude name="**/*.java" />
      </fileset>
    </copy>
    <javac
        srcdir="src"
        destdir="war/WEB-INF/classes"
        classpathref="project.classpath"
        debug="on" />
  </target>

  <target name="datanucleusenhance" depends="compile"
      description="Performs JDO enhancement on compiled data classes.">
    <enhance_war war="war" />
  </target>

  <target name="runserver" depends="datanucleusenhance"
      description="Starts the development server.">
    <dev_appserver war="war" />
  </target>

  <target name="update" depends="datanucleusenhance"
      description="Uploads the application to App Engine.">
    <appcfg action="update" war="war" />
  </target>

  <target name="update_indexes" depends="datanucleusenhance"
      description="Uploads just the datastore index configuration to App Engine.">
    <appcfg action="update_indexes" war="war" />
  </target>

  <target name="rollback" depends="datanucleusenhance"
      description="Rolls back an interrupted application update.">
    <appcfg action="rollback" war="war" />
  </target>

  <target name="request_logs"
      description="Downloads log data from App Engine for the application.">
    <appcfg action="request_logs" war="war">
      <options>
        <arg value="--num_days=5"/>
      </options>
      <args>
        <arg value="logs.txt"/>
      </args>
    </appcfg>
  </target>

</project>
```

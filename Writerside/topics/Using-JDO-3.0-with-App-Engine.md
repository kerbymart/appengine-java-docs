# Using JDO 3.0 with App Engine

  

Java Data Objects (JDO) is a standard interface for accessing databases in Java, providing a mapping between Java classes and database tables. There is an open-source plugin available for using JDO with Datastore, and this page provides information on how to get started with it.

**Warning:** We think most developers will have a better experience using the low-level Datastore API, or one of the open-source APIs developed specifically for Datastore, such as [Objectify](https://web.archive.org/web/20160424230337/https://github.com/objectify/objectify). JDO was designed for use with traditional relational databases, and so has no way to explicitly represent some of the aspects of Datastore that make it different from relational databases, such as entity groups and ancestor queries. This can lead to subtle issues that are difficult to understand and fix.

The App Engine Java SDK includes version 2.x of the DataNucleus plugin for App Engine. This plugin corresponds to version 3.0 of the DataNucleus Access Platform, which enables you to use the App Engine Datastore via JDO 3.0.

See [the Access Platform 3.0 documentation](https://web.archive.org/web/20160424230337/http://www.datanucleus.org/products/accessplatform_3_0/index.html) for more information about JDO. In particular, see [JDO Mapping](https://web.archive.org/web/20160424230337/http://www.datanucleus.org/products/accessplatform_3_0/jdo/index.html) and [JDO API](https://web.archive.org/web/20160424230337/http://www.datanucleus.org/products/accessplatform_3_0/jdo/api.html).

**Warning:** The DataNucleus plugin 2.x for App Engine uses DataNucleus v3.x. This new plugin is not fully backwards-compatible with the previous (1.x) plugin. If you upgrade to the new version, be sure to update and test your application.

## Contents

1.  [Build tools and IDEs supporting JDO 2.x and 3.0](#build_tools)
2.  [Migrating to Version 2.x of the DataNucleus Plugin](#Migrating_to_Version_2.0_of_the_DataNucleus_Plugin)
    1.  [New Default Behaviors](#New_Default_Behaviors)
    2.  [Changes to Configuration Files](#Changes_to_Configuration_Files)
3.  [Setting Up JDO 3.0](#Setting_Up_JDO_3_0)
    1.  [Copying the JARs](#Copying_the_JARs)
    2.  [Creating the jdoconfig.xml File](#Creating_the_jdoconfig_xml_File)
    3.  [Setting the Datastore Read Policy and Call Deadline](#Setting_the_Datastore_Read_Policy_and_Call_Deadline)
4.  [Enhancing Data Classes](#Enhancing_Data_Classes)
5.  [Getting a PersistenceManager Instance](#Getting_a_PersistenceManager_Instance)
6.  [Unsupported Features of JDO 3.0](#Unsupported_Features_of_JDO_3_0)
7.  [Disabling Transactions and Porting Existing JDO Apps](#Disabling_Transactions_and_Porting_Existing_JDO_Apps)

## Build tools and IDEs supporting JDO 2.x and 3.0

You can use Apache Ant, the App Engine Eclipse plugin, or Maven to use version 2.x or 3.0 of the DataNucleus plugin for App Engine:

-   **For Ant users:** The SDK includes an Ant task that performs the enhancement step. You must copy the JARs and create the configuration file when you set up your project. See [Using Apache Ant](https://web.archive.org/web/20160424230337/https://cloud.google.com/appengine/docs/java/tools/ant) for more information about the Ant task.

-   **For Eclipse users:** The App Engine Eclipse plugin supports DataNucleus plugin 2.0: you can select which version to use in the GUI configuration window.

-   **For Maven users:** You can enhance the classes with the following configurations in your `pom.xml` file:

    ```
                <plugin>
                    <groupId>org.datanucleus</groupId>
                    <artifactId>maven-datanucleus-plugin</artifactId>
                    <version>3.2.0-m1</version>
                    <configuration>
                        <api>JDO</api>
                        <props>${basedir}/datanucleus.properties</props>
                        <verbose>true</verbose>
                        <enhancerName>ASM</enhancerName>
                    </configuration>
                    <executions>
                        <execution>
                            <phase>process-classes</phase>
                            <goals>
                                <goal>enhance</goal>
                            </goals>
                        </execution>
                    </executions>
                    <dependencies>
                        <dependency>
                            <groupId>org.datanucleus</groupId>
                            <artifactId>datanucleus-core</artifactId>
                            <version>3.1.3</version>
                        </dependency>
                    </dependencies>
                </plugin>
    ```

## Migrating to Version 2.x of the DataNucleus Plugin

This section provides instructions for upgrading your app to use version 2.x of the DataNucleus plugin for App Engine, which corresponds to DataNucleus Access Platform 3.0 and JDO 3.0. This plugin is not fully backwards-compatible with the 1.x plugin and may change. If you upgrade, be sure to update and test your application code.

### New Default Behaviors

Version 2.x of the App Engine DataNucleus plugin has some different defaults than the previous version:

-   Non-transactional calls to `PersistenceManager.makePersistent()`, and `PersistenceManager.deletePersistent()` are now executed atomically. (These functions were previously executed in the next transaction or upon `PersistenceManager.close()`.)
-   There is now no longer an exception on duplicate PersistenceManagerFactory (PMF) allocation. Instead, if you have the persistence property `datanucleus.singletonPMFForName` set to *true* then it will return the currently allocated singleton PMF for that name.
-   Unowned relationships are now supported. See [Unowned Relationships](https://web.archive.org/web/20160424230337/https://cloud.google.com/appengine/docs/java/datastore/jdo/relationships.html#Unowned_Relationships).

For a full list of new features, see the [release notes](https://web.archive.org/web/20160424230337/http://code.google.com/p/datanucleus-appengine/source/browse/branches/2_1_2/dist/RELEASE_NOTES.ORM).

### Changes to Configuration Files

To upgrade your application to version 2.x of the App Engine DataNucleus plugin, you need to change the configuration settings in `build.xml` and `jdoconfig.xml`.

**Warning!** After updating your configuration, you need to test your application code to ensure backwards-compatibility. If you are setting up a new application and wish to use the latest version of the plugin, proceed to [Setting up JDO 3.0](#Setting_Up_JDO).

1.  `PersistenceManagerFactoryClass` property has changed. Change this line in `jdoconfig.xml`:  
      

    ```
            <property name="javax.jdo.PersistenceManagerFactoryClass"
                      value="org.datanucleus.store.appengine.jdo.DatastoreJDOPersistenceManagerFactory"/>
    ```

      
    to:  

    ```
            <property name="javax.jdo.PersistenceManagerFactoryClass"
                      value="org.datanucleus.api.jdo.JDOPersistenceManagerFactory"/>
    ```

#### In build.xml

The `copyjars` target needs to change in order to accommodate DataNucleus 2.x:

1.  The `copyjars` target has changed. Update this section:  

    ```
      <target name="copyjars"
          description="Copies the App Engine JARs to the WAR.">
        <mkdir dir="war/WEB-INF/lib" />
        <copy
            todir="war/WEB-INF/lib"
            flatten="true">
          <fileset dir="${sdk.dir}/lib/user">
            <include name="**/*.jar" />
          </fileset>
        </copy>
      </target>
    ```

      
    to:  

    ```
      <target name="copyjars"
          description="Copies the App Engine JARs to the WAR.">
        <mkdir dir="war/WEB-INF/lib" />
        <copy
            todir="war/WEB-INF/lib"
            flatten="true">
          <fileset dir="${sdk.dir}/lib/user">
            <include name="**/appengine-api-1.0-sdk*.jar" />
          </fileset>
          <fileset dir="${sdk.dir}/lib/opt/user">
            <include name="appengine-api-labs/v1/*.jar" />
            <include name="jsr107/v1/*.jar" />
            <include name="datanucleus/v2/*.jar" />
          </fileset>
        </copy>
      </target>
    ```

2.  The `datanucleusenhance` target has changed. Update this section:  

    ```
      <target name="datanucleusenhance" depends="compile"
          description="Performs enhancement on compiled data classes.">
        <enhance_war war="war" />
      </target>
    ```

      
    to:  

    ```
      <target name="datanucleusenhance" depends="compile"
          description="Performs enhancement on compiled data classes.">
          <enhance_war war="war">
                  <args>
                  <arg value="-enhancerVersion"/>
                  <arg value="v2"/>
              </args>
          </enhance_war>
      </target>
    ```

## Setting Up JDO 3.0

To use JDO to access the datastore, an App Engine app needs the following:

-   The JARs for JDO and the DataNucleus plugin must be in the app's `war/WEB-INF/lib/` directory.
-   A configuration file named `jdoconfig.xml` must be in the app's `war/WEB-INF/classes/META-INF/` directory, with configuration that tells JDO to use the App Engine datastore.
-   The project's build process must perform a post-compilation "enhancement" step on the compiled data classes to associate them with the JDO implementation.

### Copying the JARs

The JDO and datastore JARs are included with the App Engine Java SDK. You can find them in the `appengine-java-sdk/lib/opt/user/datanucleus/v2/` directory.

Copy the JARs to your application's `war/WEB-INF/lib/` directory.

Make sure the `appengine-api.jar` is also in the `war/WEB-INF/lib/` directory. (You may have already copied this when creating your project.) The App Engine DataNucleus plugin uses this JAR to access the datastore.

### Creating the jdoconfig.xml File

The JDO interface needs a configuration file named `jdoconfig.xml` in the application's `war/WEB-INF/classes/META-INF/` directory. You can create this file in this location directly, or have your build process copy this file from a source directory.

Create the file with the following contents:

```
<?xml version="1.0" encoding="utf-8"?>
<jdoconfig xmlns="http://java.sun.com/xml/ns/jdo/jdoconfig"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://java.sun.com/xml/ns/jdo/jdoconfig">

    <persistence-manager-factory name="transactions-optional">
        <property name="javax.jdo.PersistenceManagerFactoryClass"
            value="org.datanucleus.api.jdo.JDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="true"/>
        <property name="datanucleus.appengine.singletonPMFForName" value="true"/>
    </persistence-manager-factory>
</jdoconfig>
```

### Setting the Datastore Read Policy and Call Deadline

As described on the [Datastore Queries](https://web.archive.org/web/20160424230337/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Data_consistency) page, you can customize the behavior of the Datastore by setting the read policy (strong versus eventual consistency) and call deadline. In JDO, you do this by specifying the desired values in the `<persistence-manager-factory>` element of the `jdoconfig.xml` file. All calls made with a given `PersistenceManager` instance will use the configuration values in effect when the manager was created by the `PersistenceManagerFactory`. You can also override these settings for a single `Query` object.

To set the read policy for a `PersistenceManagerFactory`, include a property named `datanucleus.appengine.datastoreReadConsistency`. Possible values are `EVENTUAL` and `STRONG`: if not specified, the default is `STRONG`. (Note, however, that these settings apply only to ancestor queries within a given entity group. Non-ancestor queries are always eventually consistent regardless of the prevailing read policy.)

```
        <property name="datanucleus.appengine.datastoreReadConsistency" value="EVENTUAL" />
```

You can set separate datastore call deadlines for reads and for writes. For reads, use the JDO standard property `javax.jdo.option.DatastoreReadTimeoutMillis`. For writes, use `javax.jdo.option.DatastoreWriteTimeoutMillis`. The value is an amount of time, in milliseconds.

```
        <property name="javax.jdo.option.DatastoreReadTimeoutMillis" value="5000" />
        <property name="javax.jdo.option.DatastoreWriteTimeoutMillis" value="10000" />
```

If you want to use cross-group (XG) transactions, add the following property:

```
        <property name="datanucleus.appengine.datastoreEnableXGTransactions" value="true" />
```

You can have multiple `<persistence-manager-factory>` elements in the same `jdoconfig.xml` file, using different `name` attributes, to use `PersistenceManager` instances with different configurations in the same app. For example, the following `jdoconfig.xml` file establishes two sets of configuration, one named `"transactions-optional"` and another named `"eventual-reads-short-deadlines"`:

```
<?xml version="1.0" encoding="utf-8"?>
<jdoconfig xmlns="http://java.sun.com/xml/ns/jdo/jdoconfig"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://java.sun.com/xml/ns/jdo/jdoconfig">

    <persistence-manager-factory name="transactions-optional">
        <property name="javax.jdo.PersistenceManagerFactoryClass"
            value="org.datanucleus.api.jdo.JDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="true"/>
    </persistence-manager-factory>

    <persistence-manager-factory name="eventual-reads-short-deadlines">
        <property name="javax.jdo.PersistenceManagerFactoryClass"
            value="org.datanucleus.api.jdo.JDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="true"/>

        <property name="datanucleus.appengine.datastoreReadConsistency" value="EVENTUAL" />
        <property name="javax.jdo.option.DatastoreReadTimeoutMillis" value="5000" />
        <property name="javax.jdo.option.DatastoreWriteTimeoutMillis" value="10000" />
        <property name="datanucleus.singletonPMFForName" value="true" />
    </persistence-manager-factory>
</jdoconfig>
```

See [Getting a PersistenceManager Instance](#Getting_a_PersistenceManager_Instance) below for information on creating a `PersistenceManager` with a named configuration set.

## Enhancing Data Classes

JDO uses a post-compilation "enhancement" step in the build process to associate data classes with the JDO implementation.

You can perform the enhancement step on compiled classes from the command line with the following command:

```
java -cp classpath com.google.appengine.tools.enhancer.Enhance
class-files
```

The *classpath* must contain the JAR `appengine-tools-api.jar` from the `appengine-java-sdk/lib/` directory, as well as all of your data classes.

For more information on the DataNucleus bytecode enhancer, see [the DataNucleus documentation](https://web.archive.org/web/20160424230337/http://www.datanucleus.org/products/datanucleus/jdo/enhancer.html).

## Getting a PersistenceManager Instance

An app interacts with JDO using an instance of the PersistenceManager class. You get this instance by instantiating and calling a method on an instance of the PersistenceManagerFactory class. The factory uses the JDO configuration to create PersistenceManager instances.

Because a PersistenceManagerFactory instance takes time to initialize, an app should reuse a single instance. An easy way to manage the PersistenceManagerFactory instance is to create a singleton wrapper class with a static instance, as follows:

**`PMF.java`**

```
import javax.jdo.JDOHelper;
import javax.jdo.PersistenceManagerFactory;

public final class PMF {
    private static final PersistenceManagerFactory pmfInstance =
        JDOHelper.getPersistenceManagerFactory("transactions-optional");

    private PMF() {}

    public static PersistenceManagerFactory get() {
        return pmfInstance;
    }
}
```

**Tip:** `"transactions-optional"` refers to the name of the configuration set in the `jdoconfig.xml` file. If your app uses multiple configuration sets, you'll have to extend this code to call `JDOHelper.getPersistenceManagerFactory()` as desired. Your code should cache a singleton instance of each `PersistenceManagerFactory`.

The app uses the factory instance to create one PersistenceManager instance for each request that accesses the datastore.

```
import javax.jdo.JDOHelper;
import javax.jdo.PersistenceManager;
import javax.jdo.PersistenceManagerFactory;

import PMF;

// ...
    PersistenceManager pm = PMF.get().getPersistenceManager();
```

You use the PersistenceManager to store, update, and delete data objects, and to perform datastore queries.

When you are done with the PersistenceManager instance, you must call its `close()` method. It is an error to use the PersistenceManager instance after calling its `close()` method.

```
    try {
        // ... do stuff with pm ...
    } finally {
        pm.close();
    }
```

## Unsupported Features of JDO 3.0

The following features of the JDO interface are not supported by the App Engine implementation:

-   Owned many-to-many relationships.
-   "Join" queries. You cannot use a field of a child entity in a filter when performing a query on the parent kind. Note that you can test the parent's relationship field directly in query using a key.
-   JDOQL grouping and other aggregate queries.
-   Polymorphic queries. You cannot perform a query of a class to get instances of a subclass. Each class is represented by a separate entity kind in the datastore.

## Disabling Transactions and Porting Existing JDO Apps

The JDO configuration we recommend using sets a property named `datanucleus.appengine.autoCreateDatastoreTxns` to `true`. This is an App Engine-specific property that tells the JDO implementation to associate datastore transactions with the JDO transactions that are managed in application code. If you are building a new app from scratch, this is probably what you want. However, if you have an existing, JDO-based application that you want to get running on App Engine, you may want to use an alternate persistence configuration which sets the value of this property to `false`:

```
<?xml version="1.0" encoding="utf-8"?>
<jdoconfig xmlns="http://java.sun.com/xml/ns/jdo/jdoconfig"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://java.sun.com/xml/ns/jdo/jdoconfig">

    <persistence-manager-factory name="transactions-optional">
        <property name="javax.jdo.PersistenceManagerFactoryClass"
            value="org.datanucleus.api.jdo.JDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="false"/>
    </persistence-manager-factory>
</jdoconfig>
```

In order to understand why this may be useful, remember that you can only operate on objects that belong to the same entity group within a transaction. Applications built using traditional databases typically assume the availability of global transactions, which allow you to update any set of records inside a transaction. Since the App Engine datastore does not support global transactions, App Engine throws exceptions if your code assumes the availability of global transactions. Instead of going through your (potentially large) codebase and removing all your transaction management code, you can simply disable datastore transactions. This does nothing to address assumptions your code makes about atomicity of multi-record modifications, but it allows you to get your app working so that you can then focus on refactoring your transactional code incrementally and as needed, rather than all at once.

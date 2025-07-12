# Using JDO 2.3 with App Engine

  

Java Data Objects (JDO) is a standard interface for accessing databases in Java, providing a mapping between Java classes and database tables. There is an open-source plugin available for using JDO with Datastore, and this page provides information on how to get started with it.

**Warning:** We think most developers will have a better experience using the low-level Datastore API, or one of the open-source APIs developed specifically for Datastore, such as [Objectify](https://web.archive.org/web/20160424230348/https://github.com/objectify/objectify). JDO was designed for use with traditional relational databases, and so has no way to explicitly represent some of the aspects of Datastore that make it different from relational databases, such as entity groups and ancestor queries. This can lead to subtle issues that are difficult to understand and fix.

The App Engine Java SDK includes an implementation of JDO 2.3 for the App Engine Datastore. The implementation is based on version 1.0 of the DataNucleus Access Platform, the open-source reference implementation for JDO 2.3.

See [the Access Platform 1.1 documentation](https://web.archive.org/web/20160424230348/http://www.datanucleus.org/products/accessplatform_1_1.html) for more information about JDO.

**Note:** The instructions on this page apply to JDO version 2.3, which uses version 1.0 of the DataNucleus plugin for App Engine. App Engine now offers the DataNucleus 2.x plugin that allows you to run JDO 3.0. The new plugin supports unowned relationships and provides a number of new APIs and features. This upgrade is not fully backwards-compatible with the previous version. If you rebuild an application using JDO 3.0, you need to update and retest your code. For more information on the new version, see [JDO 3.0](https://web.archive.org/web/20160424230348/http://www.datanucleus.org/products/accessplatform_2_2/guides/jdo/jdo_3.html). For more information about upgrading, see [Using JDO 3.0 with App Engine](https://web.archive.org/web/20160424230348/https://cloud.google.com/appengine/docs/java/datastore/jdo/overview-dn2.html).

## Contents

1.  [Setting Up JDO 2.3](#Setting_Up_JDO_2_3)
    1.  [Copying the JARs](#Copying_the_JARs)
    2.  [Creating the jdoconfig.xml File](#Creating_the_jdoconfig_xml_File)
    3.  [Setting the Datastore Read Policy and Call Deadline](#Setting_the_Datastore_Read_Policy_and_Call_Deadline)
2.  [Enhancing Data Classes](#Enhancing_Data_Classes)
3.  [Getting a PersistenceManager Instance](#Getting_a_PersistenceManager_Instance)
4.  [Unsupported Features of JDO 2.3](#Unsupported_Features_of_JDO_2_3)
5.  [Disabling Transactions and Porting Existing JDO Apps](#Disabling_Transactions_and_Porting_Existing_JDO_Apps)

## Setting Up JDO 2.3

To use JDO to access the datastore, an App Engine app needs the following:

-   The JDO and DataNucleus App Engine plugin JARs must be in the app's `war/WEB-INF/lib/` directory.
-   A configuration file named `jdoconfig.xml` must be in the app's `war/WEB-INF/classes/META-INF/` directory, with configuration that tells JDO to use the App Engine datastore.
-   The project's build process must perform a post-compilation "enhancement" step on the compiled data classes to associate them with the JDO implementation.

If you are using [the Google Plugin for Eclipse](https://web.archive.org/web/20160424230348/https://developers.google.com/eclipse/docs/appengine), these three things are taken care of for you. The new project wizard puts the JDO and DataNucleus App Engine plugin JARs in the correct location, and creates the `jdoconfig.xml` file. The build process performs the "enhancement" step automatically.

If you are using Apache Ant to build your project, you can use an Ant task included with the SDK to perform the enhancement step. You must copy the JARs and create the configuration file when you set up your project. See [Using Apache Ant](https://web.archive.org/web/20160424230348/https://cloud.google.com/appengine/docs/java/tools/ant) for more information about the Ant task.

### Copying the JARs

The JDO and datastore JARs are included with the App Engine Java SDK. You can find them in the `appengine-java-sdk/lib/user/orm/` directory.

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
            value="org.datanucleus.store.appengine.jdo.DatastoreJDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="true"/>
    </persistence-manager-factory>
</jdoconfig>
```

### Setting the Datastore Read Policy and Call Deadline

As described on the [Datastore Queries](https://web.archive.org/web/20160424230348/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Data_consistency) page, you can customize the behavior of the Datastore by setting the read policy (strong versus eventual consistency) and call deadline. In JDO, you do this by specifying the desired values in the `<persistence-manager-factory>` element of the `jdoconfig.xml` file. All calls made with a given `PersistenceManager` instance will use the configuration values in effect when the manager was created by the `PersistenceManagerFactory`. You can also override these settings for a single `Query` object.

To set the read policy for a `PersistenceManagerFactory`, include a property named `datanucleus.appengine.datastoreReadConsistency`. Possible values are `EVENTUAL` and `STRONG`: if not specified, the default is `STRONG`. Note that these settings apply only to ancestor queries within a given entity group. Non-ancestor and cross-group queries are always eventually consistent regardless of the prevailing read policy.

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
            value="org.datanucleus.store.appengine.jdo.DatastoreJDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="true"/>
    </persistence-manager-factory>

    <persistence-manager-factory name="eventual-reads-short-deadlines">
        <property name="javax.jdo.PersistenceManagerFactoryClass"
            value="org.datanucleus.store.appengine.jdo.DatastoreJDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="true"/>

        <property name="datanucleus.appengine.datastoreReadConsistency" value="EVENTUAL" />
        <property name="javax.jdo.option.DatastoreReadTimeoutMillis" value="5000" />
        <property name="javax.jdo.option.DatastoreWriteTimeoutMillis" value="10000" />
    </persistence-manager-factory>
</jdoconfig>
```

See [Getting a PersistenceManager Instance](#Getting_a_PersistenceManager_Instance) below for information on creating a `PersistenceManager` with a named configuration set.

## Enhancing Data Classes

JDO uses a post-compilation "enhancement" step in the build process to associate data classes with the JDO implementation.

If you are using Eclipse, [the Google Plugin for Eclipse](https://web.archive.org/web/20160424230348/https://cloud.google.com/appengine/docs/java/tools/eclipse) does this step automatically when building.

If you are using Apache Ant, the SDK includes an Ant task to perform this step. See [Using Apache Ant](https://web.archive.org/web/20160424230348/https://cloud.google.com/appengine/docs/java/tools/ant) for more information on using the Ant task.

You can perform the enhancement step on compiled classes from the command line with the following command:

```
java -cp classpath com.google.appengine.tools.enhancer.Enhance
class-files
```

The *classpath* must contain the JAR `appengine-tools-api.jar` from the `appengine-java-sdk/lib/` directory, as well as all of your data classes.

For more information on the DataNucleus bytecode enhancer, see [the DataNucleus documentation](https://web.archive.org/web/20160424230348/http://www.datanucleus.org/products/datanucleus/jdo/enhancer.html).

## Getting a PersistenceManager Instance

An app interacts with JDO using an instance of the PersistenceManager class. You get this instance by instantiating and calling a method on an instance of the PersistenceManagerFactory class. The factory uses the JDO configuration to create PersistenceManager instances.

Because a PersistenceManagerFactory instance takes time to initialize, an app should reuse a single instance. To enforce this, an exception is thrown if the app instantiates more than one PersistenceManagerFactory (with the same configuration name). An easy way to manage the PersistenceManagerFactory instance is to create a singleton wrapper class with a static instance, as follows:

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

## Unsupported Features of JDO 2.3

The following features of the JDO interface are not supported by the App Engine implementation:

-   Unowned relationships. You can implement unowned relationships using explicit Key values. JDO's syntax for unowned relationships may be supported in a future release.
-   Owned many-to-many relationships.
-   "Join" queries. You cannot use a field of a child entity in a filter when performing a query on the parent kind. Note that you can test the parent's relationship field directly in query using a key.
-   JDOQL grouping and other aggregate queries.
-   Polymorphic queries. You cannot perform a query of a class to get instances of a subclass. Each class is represented by a separate entity kind in the datastore.
-   `IdentityType.DATASTORE` for the `@PersistenceCapable` annotation. Only `IdentityType.APPLICATION` is supported.
-   There is currently a bug preventing owned one-to-many relationships where the parent and the child are the same class, making it difficult to model tree structures. This will be fixed in a future release. You can work around this by storing explicit Key values for either the parent or children.

## Disabling Transactions and Porting Existing JDO Apps

The JDO configuration we recommend using sets a property named `datanucleus.appengine.autoCreateDatastoreTxns` to `true`. This is an App Engine-specific property that tells the JDO implementation to associate datastore transactions with the JDO transactions that are managed in application code. If you are building a new app from scratch, this is probably what you want. However, if you have an existing, JDO-based application that you want to get running on App Engine, you may want to use an alternate persistence configuration which sets the value of this property to `false`:

```
<?xml version="1.0" encoding="utf-8"?>
<jdoconfig xmlns="http://java.sun.com/xml/ns/jdo/jdoconfig"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://java.sun.com/xml/ns/jdo/jdoconfig">

    <persistence-manager-factory name="transactions-optional">
        <property name="javax.jdo.PersistenceManagerFactoryClass"
            value="org.datanucleus.store.appengine.jdo.DatastoreJDOPersistenceManagerFactory"/>
        <property name="javax.jdo.option.ConnectionURL" value="appengine"/>
        <property name="javax.jdo.option.NontransactionalRead" value="true"/>
        <property name="javax.jdo.option.NontransactionalWrite" value="true"/>
        <property name="javax.jdo.option.RetainValues" value="true"/>
        <property name="datanucleus.appengine.autoCreateDatastoreTxns" value="false"/>
    </persistence-manager-factory>
</jdoconfig>
```

In order to understand why this may be useful, remember that you can only operate on objects that belong to the same entity group within a transaction. Applications built using traditional databases typically assume the availability of global transactions, which allow you to update any set of records inside a transaction. Since the App Engine datastore does not support global transactions, App Engine throws exceptions if your code assumes the availability of global transactions. Instead of going through your (potentially large) codebase and removing all your transaction management code, you can simply disable datastore transactions. This does nothing to address assumptions your code makes about atomicity of multi-record modifications, but it allows you to get your app working so that you can then focus on refactoring your transactional code incrementally and as needed, rather than all at once.

# Using JPA with App Engine (2.0)

  

Java Persistence API (JPA) is a standard interface for accessing databases in Java, providing an automatic mapping between Java classes and database tables. There is an open-source plugin available for using JPA with Datastore, and this page provides information on how to get started with it.

**Warning:** We think most developers will have a better experience using the low-level Datastore API, or one of the open-source APIs developed specifically for Datastore, such as [Objectify](https://web.archive.org/web/20160424230323/https://github.com/objectify/objectify). JPA was designed for use with traditional relational databases, and so has no way to explicitly represent some of the aspects of Datastore that make it different from relational databases, such as entity groups and ancestor queries. This can lead to subtle issues that are difficult to understand and fix.

The App Engine Java SDK includes version 2.x of the DataNucleus plugin for Datastore. This plugin corresponds to version 3.0 of the DataNucleus Access Platform, which enables you to use the App Engine Datastore via JPA 2.0.

See [the Access Platform 3.0 documentation](https://web.archive.org/web/20160424230323/http://www.datanucleus.org/products/accessplatform_3_0/index.html) for more information about JPA. In particular, see [JPA Documentation](https://web.archive.org/web/20160424230323/http://www.datanucleus.org/products/accessplatform_3_0/jpa/index.html) and [JPA API](https://web.archive.org/web/20160424230323/http://www.datanucleus.org/products/accessplatform_3_0/jpa/api.html).

**Warning:** Version 2.x of the DataNucleus plugin for App Engine uses DataNucleus v3.x. The 2.x plugin is not fully backwards-compatible with the previous 1.x plugin. If you upgrade to the new version, be sure to update and test your application.

1.  [Build tools and IDEs supporting JPA 2.x and 3.0](#build_tools)
2.  [Migrating to Version 2.x of the DataNucleus Plugin](#Migrating_to_Version_2.0_of_the_DataNucleus_Plugin)
3.  [Setting Up JPA 2.0](#Setting_Up_JPA_2_0)
4.  [Enhancing Data Classes](#Enhancing_Data_Classes)
5.  [Getting an EntityManager Instance](#Getting_an_EntityManager_Instance)
6.  [Class and Field Annotations](#Class_and_Field_Annotations)
7.  [Inheritance](#Inheritance)
8.  [Unsupported Features of JPA 2.0](#Unsupported_Features_of_JPA)

## Build tools and IDEs supporting JPA 2.x and 3.0

You can use Apache Ant, the App Engine Eclipse plugin, or Maven to use version 2.x or 3.0 of the DataNucleus plugin for App Engine:

-   **For Ant users:** The SDK includes an Ant task that performs the enhancement step. You must copy the JARs and create the configuration file when you set up your project. See [Using Apache Ant](https://web.archive.org/web/20160424230323/https://cloud.google.com/appengine/docs/java/tools/ant) for more information about the Ant task.

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
                            <artifactId>datanucleus-api-jdo</artifactId>
                            <version>3.1.3</version>
                        </dependency>
                    </dependencies>
                </plugin>
    ```

## Migrating to Version 2.x of the DataNucleus Plugin

This section provides instructions for upgrading your app to use version 2.x of the DataNucleus plugin for App Engine, which corresponds to DataNucleus Access Platform 3.0 and JPA 2.0. The 2.x plugin version is not fully backwards-compatible with version 1.x and may change without warning. If you upgrade, be sure to update and test your application code.

### New Default Behaviors

Version 2.x of the App Engine DataNucleus plugin has some different defaults than the previous 1.x version:

-   JPA "persistence provider" is now `org.datanucleus.api.jpa.PersistenceProviderImpl`.
-   Level2 Caching is enabled by default. To get the previous default behavior, set the persistence property `datanucleus.cache.level2.type` to *none*. (Alternatively include the datanucleus-cache plugin in the classpath, and set the persistence property `datanucleus.cache.level2.type` to *javax.cache* to use Memcache for L2 caching.
-   Datastore `IdentifierFactory` now defaults to *datanucleus2*. To get the previous behavior, set the persistence property `datanucleus.identifierFactory` to *datanucleus1*.
-   Non-transactional calls to `EntityManager.persist()`, `EntityManager.merge()`, and `EntityManager.remove()` are now executed atomically. (Previously, execution occurred at the next transaction or upon `EntityManager.close()`.
-   JPA has `retainValues` enabled, which means the values of loaded fields are retained in objects after a commit.
-   `javax.persistence.query.chunkSize` is no longer used. Use `datanucleus.query.fetchSize` instead.
-   There is now no longer an exception on duplicate EMF allocation. If you have the persistence property `datanucleus.singletonEMFForName` set to *true*, then it will return the currently allocated singleton EMF for that name.
-   Unowned relationships are now supported.
-   Datastore Identity is now supported.

For a full list of new features, see the [release notes](https://web.archive.org/web/20160424230323/http://code.google.com/p/datanucleus-appengine/source/browse/branches/2_1_2/dist/RELEASE_NOTES.ORM).

### Changes to Configuration Files

To upgrade your app to use version 2.0 of the DataNucleus plugin for App Engine, you need to change some configuration settings in `build.xml` and `persistence.xml`. If you are setting up a new application and wish to use the latest version of the DataNucleus plugin, proceed to [Setting up JPA 2.0](#Setting_Up_JPA_2_0).

**Warning!** After updating your configuration, you need to test your application code to ensure backwards-compatibility.

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

#### In persistence.xml

The `<provider>` target has changed. Update this section:

```
        <provider>org.datanucleus.store.appengine.jpa.DatastorePersistenceProvider</provider>
```

to:

```
        <provider>org.datanucleus.api.jpa.PersistenceProviderImpl</provider>
```

## Setting Up JPA 2.0

To use JPA to access the datastore, an App Engine app needs the following:

-   The JPA and datastore JARs must be in the app's `war/WEB-INF/lib/` directory.
-   A configuration file named `persistence.xml` must be in the app's `war/WEB-INF/classes/META-INF/` directory, with configuration that tells JPA to use the App Engine datastore.
-   The project's build process must perform a post-compilation "enhancement" step on the compiled data classes to associate them with the JPA implementation.

### Copying the JARs

The JPA and datastore JARs are included with the App Engine Java SDK. You can find them in the `appengine-java-sdk/lib/opt/user/datanucleus/v2/` directory.

Copy the JARs to your application's `war/WEB-INF/lib/` directory.

Make sure the `appengine-api.jar` is also in the `war/WEB-INF/lib/` directory. (You may have already copied this when creating your project.) The App Engine DataNucleus plugin uses this JAR to access the datastore.

### Creating the persistence.xml File

The JPA interface needs a configuration file named `persistence.xml` in the application's `war/WEB-INF/classes/META-INF/` directory. You can create this file in this location directly, or have your build process copy this file from a source directory.

Create the file with the following contents:

```
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
        http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd" version="1.0">

    <persistence-unit name="transactions-optional">
        <provider>org.datanucleus.api.jpa.PersistenceProviderImpl</provider>
        <properties>
            <property name="datanucleus.NontransactionalRead" value="true"/>
            <property name="datanucleus.NontransactionalWrite" value="true"/>
            <property name="datanucleus.ConnectionURL" value="appengine"/>
            <property name="datanucleus.singletonEMFForName" value="true"/>
        </properties>

    </persistence-unit>

</persistence>
```

### Datastore Read Policy and Call Deadline

As described on the [Datastore Queries](https://web.archive.org/web/20160424230323/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Data_consistency) page, you can set the read policy (strong consistency vs. eventual consistency) and the datastore call deadline for a `EntityManagerFactory` in the `persistence.xml` file. These settings go in the `<persistence-unit>` element. All calls made with a given `EntityManager` instance use the configuration selected when the manager was created by the `EntityManagerFactory`. You can also override these options for an individual `Query` (described below).

To set the read policy, include a property named `datanucleus.appengine.datastoreReadConsistency`. Its possible values are `EVENTUAL` (for reads with eventual consistency) and `STRONG` (for reads with strong consistency). If not specified, the default is `STRONG`.

```
            <property name="datanucleus.appengine.datastoreReadConsistency" value="EVENTUAL" />
```

You can set separate datastore call deadlines for reads and for writes. For reads, use the JPA standard property `javax.persistence.query.timeout`. For writes, use `datanucleus.datastoreWriteTimeout`. The value is an amount of time, in milliseconds.

```
            <property name="javax.persistence.query.timeout" value="5000" />
            <property name="datanucleus.datastoreWriteTimeout" value="10000" />
```

If you want to use cross-group (XG) transactions, add the following property:

```
            <property name="datanucleus.appengine.datastoreEnableXGTransactions" value="true" />
```

You can have multiple `<persistence-unit>` elements in the same `persistence.xml` file, using different `name` attributes, to use `EntityManager` instances with different configurations in the same app. For example, the following `persistence.xml` file establishes two sets of configuration, one named `"transactions-optional"` and another named `"eventual-reads-short-deadlines"`:

```
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
        http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd" version="1.0">

    <persistence-unit name="transactions-optional">
        <provider>org.datanucleus.api.jpa.PersistenceProviderImpl</provider>
        <properties>
            <property name="datanucleus.NontransactionalRead" value="true"/>
            <property name="datanucleus.NontransactionalWrite" value="true"/>
            <property name="datanucleus.ConnectionURL" value="appengine"/>
        </properties>
    </persistence-unit>

    <persistence-unit name="eventual-reads-short-deadlines">
        <provider>org.datanucleus.api.jpa.PersistenceProviderImpl</provider>
        <properties>
            <property name="datanucleus.NontransactionalRead" value="true"/>
            <property name="datanucleus.NontransactionalWrite" value="true"/>
            <property name="datanucleus.ConnectionURL" value="appengine"/>

            <property name="datanucleus.appengine.datastoreReadConsistency" value="EVENTUAL" />
            <property name="javax.persistence.query.timeout" value="5000" />
            <property name="datanucleus.datastoreWriteTimeout" value="10000" />
            <property name="datanucleus.singletonEMFForName" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

See [Getting an EntityManager Instance](#Getting_an_EntityManager_Instance) below for information on creating an `EntityManager` with a named configuration set.

You can override the read policy and call deadline for an individual `Query` object. To override the read policy for a `Query`, call its `setHint()` method as follows:

```
        Query q = em.createQuery("select from " + Book.class.getName());
        q.setHint("datanucleus.appengine.datastoreReadConsistency", "EVENTUAL");
```

As above, the possible values are `"EVENTUAL"` and `"STRONG"`.

To override the read timeout, call `setHint()` as follows:

```
        q.setHint("javax.persistence.query.timeout", 3000);
```

There is no way to override the configuration for these options when you fetch entities by key.

## Enhancing Data Classes

The DataNucleus implementation of JPA uses a post-compilation "enhancement" step in the build process to associate data classes with the JPA implementation.

You can perform the enhancement step on compiled classes from the command line with the following command:

```
java -cp classpath org.datanucleus.enhancer.DataNucleusEnhancer
class-files
```

The *classpath* must contain the JARs `datanucleus-core-*.jar`, `datanucleus-jpa-*`, `datanucleus-enhancer-*.jar`, `asm-*.jar`, and `geronimo-jpa-*.jar` (where `*` is the appropriate version number of each JAR) from the `appengine-java-sdk/lib/tools/` directory, as well as all of your data classes.

For more information on the DataNucleus bytecode enhancer, see [the DataNucleus documentation](https://web.archive.org/web/20160424230323/http://www.datanucleus.org/products/datanucleus/jpa/enhancer.html).

## Getting an EntityManager Instance

An app interacts with JPA using an instance of the `EntityManager` class. You get this instance by instantiating and calling a method on an instance of the `EntityManagerFactory` class. The factory uses the JPA configuration (identified by the name `"transactions-optional"`) to create `EntityManager` instances.

Because an `EntityManagerFactory` instance takes time to initialize, it's a good idea to reuse a single instance as much as possible. An easy way to do this is to create a singleton wrapper class with a static instance, as follows:

**`EMF.java`**

```
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public final class EMF {
    private static final EntityManagerFactory emfInstance =
        Persistence.createEntityManagerFactory("transactions-optional");

    private EMF() {}

    public static EntityManagerFactory get() {
        return emfInstance;
    }
}
```

**Tip:** `"transactions-optional"` refers to the name of the configuration set in the `persistence.xml` file. If your app uses multiple configuration sets, you'll have to extend this code to call `Persistence.createEntityManagerFactory()` as desired. Your code should cache a singleton instance of each `EntityManagerFactory`.

The app uses the factory instance to create one `EntityManager` instance for each request that accesses the datastore.

```
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;

import EMF;

// ...
    EntityManager em = EMF.get().createEntityManager();
```

You use the `EntityManager` to store, update, and delete data objects, and to perform datastore queries.

When you are done with the `EntityManager` instance, you must call its `close()` method. It is an error to use the `EntityManager` instance after calling its `close()` method.

```
    try {
        // ... do stuff with em ...
    } finally {
        em.close();
    }
```

## Class and Field Annotations

Each object saved by JPA becomes an entity in the App Engine datastore. The entity's kind is derived from the simple name of the class (without the package name). Each persistent field of the class represents a property of the entity, with the name of the property equal to the name of the field (with case preserved).

To declare a Java class as capable of being stored and retrieved from the datastore with JPA, give the class a `@Entity` annotation. For example:

```
import javax.persistence.Entity;

@Entity
public class Employee {
    // ...
}
```

Fields of the data class that are to be stored in the datastore must either be of a type that is persisted by default or expliclty declared as persistent. You can find a chart detailing JPA default persistence behavior on [the DataNucleus website](https://web.archive.org/web/20160424230323/http://www.datanucleus.org/products/accessplatform/jpa/types.html). To explicitly declare a field as persistent, you give it an `@Basic` annotation:

```
import java.util.Date;
import javax.persistence.Enumerated;

import com.google.appengine.api.datastore.ShortBlob;

// ...
    @Basic
    private ShortBlob data;
```

The type of a field can be any of the following:

-   one of the core types supported by the datastore
-   a Collection (such as a `java.util.List<...>`) of values of a core datastore type
-   an instance or Collection of instances of a `@Entity` class
-   an embedded class, stored as properties on the entity

A data class must have a public or protected default constructor and one field dedicated to storing the primary key of the corresponding datastore entity. You can choose between four different kinds of key fields, each using a different value type and annotations. (See [Creating Data: Keys](https://web.archive.org/web/20160424230323/https://cloud.google.com/appengine/docs/java/datastore/jdo/creatinggettinganddeletingdata#Keys) for more information.) The simplest key field is a long integer value that is automatically populated by JPA with a value unique across all other instances of the class when the object is saved to the datastore for the first time. Long integer keys use a `@Id` annotation, and a `@GeneratedValue(strategy = GenerationType.IDENTITY)` annotation:

```
import com.google.appengine.api.datastore.Key;

import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

// ...
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Key key;
```

Here is an example data class:

```
import com.google.appengine.api.datastore.Key;

import java.util.Date;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Key key;

    private String firstName;

    private String lastName;

    private Date hireDate;

    // Accessors for the fields. JPA doesn't use these, but your application
    does.

    public Key getKey() {
        return key;
    }

    public String getFirstName() {
        return firstName;
    }
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public Date getHireDate() {
        return hireDate;
    }
    public void setHireDate(Date hireDate) {
        this.hireDate = hireDate;
    }
}
```

## Inheritance

JPA supports creating data classes that use inheritance. Before we talk about how JPA inheritance works on App Engine, we recommend you read [the DataNucleus documentation](https://web.archive.org/web/20160424230323/http://www.datanucleus.org/products/accessplatform/jpa/orm/inheritance.html) on this subject and then come back. Done? Ok. JPA inheritance on App Engine works as described in the DataNucleus documentation with some additional restrictions. We'll discuss these restrictions and then give some concrete examples.

The "JOINED" inheritance strategy allows you to split the data for a single data object across multiple "tables," but since the App Engine datastore does not support joins, operating on a data object with this inheritance strategy requires a remote procedure call for each level of inheritance. This is potentially very inefficient, so the "JOINED" inheritance strategy is not supported on data classes.

Second, the "SINGLE\_TABLE" inheritance strategy allows you to store the data for a data object in a single "table" associated with the persistent class at the root of your inheritance hierarchy. Although there are no inherent inefficiencies in this strategy, it is not currently supported. We may revisit this in future releases.

Now the good news: The "TABLE\_PER\_CLASS" and "MAPPED\_SUPERCLASS" strategies work as described in the DataNucleus documentation. Let's look at an example:

**`Worker.java`**

```
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.MappedSuperclass;

@Entity
@MappedSuperclass
public abstract class Worker {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Key key;

    private String department;
}
```

**`Employee.java`**

```
// ... imports ...

@Entity
public class Employee extends Worker {
    private int salary;
}
```

**`Intern.java`**

```
import java.util.Date;
// ... imports ...

@Entity
public class Intern extends Worker {
    private Date internshipEndDate;
}
```

In this example we've added an `@MappedSuperclass` annotation to the `Worker` class declaration. This tells JPA to store all persistent fields of the `Worker` in the datastore entities of its subclasses. The datastore entity created as the result of calling `persist()` with an `Employee` instance will have two properties named "department" and "salary". The datastore entity created as the result of calling `persist()` with an `Intern` instance will have two properties named "department" and "inernshipEndDate". There will not be any entities of kind "Worker" in the datastore.

Now let's make things a little more interesting. Suppose, in addition to having `Employee` and `Intern`, we also want a specialization of `Employee` that describes employees who have left the company:

**`FormerEmployee.java`**

```
import java.util.Date;
import javax.persistence.Inheritance;
import javax.persistence.InheritanceType;
// ... imports ...

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class FormerEmployee extends Employee {
    private Date lastDay;
}
```

In this example we've added an `@Inheritance` annotation to the `FormerEmployee` class declaration with its `strategy` attribute set to `InheritanceType.TABLE_PER_CLASS`. This tells JPA to store all persistent fields of the `FormerEmployee` and its superclasses in datastore entities corresponding to `FormerEmployee` instances. The datastore entity created as the result of calling `persist()` with an `FormerEmployee` instance will have three properties named "department", "salary", and "lastDay". There will never be an entity of kind "Employee" that corresponds to a `FormerEmployee`, but if you call `persist()` with a object whose runtime type is `Employee` you will create an entity of kind "Employee.

Mixing relationships with inheritance works so long as the declared types of your relationship fields match the runtime types of the objects you are assigning to those fields. Refer to the section on [Polymorphic Relationships](https://web.archive.org/web/20160424230323/https://cloud.google.com/appengine/docs/java/datastore/jdo/relationships#Polymorphic_Relationships) for more information. This section contains JDO examples, but the concepts and the restrictions are the same for JPA.

## Unsupported Features of JPA 2.0

The following features of the JPA interface are not supported by the App Engine implementation:

-   Owned many-to-many relationships.
-   "Join" queries. You cannot use a field of a child entity in a filter when performing a query on the parent kind. Note that you can test the parent's relationship field directly in a query using a key.
-   Aggregation queries (group by, having, sum, avg, max, min)
-   Polymorphic queries. You cannot perform a query of a class to get instances of a subclass. Each class is represented by a separate entity kind in the datastore.

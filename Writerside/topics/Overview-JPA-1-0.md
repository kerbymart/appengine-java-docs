# Using JPA with App Engine (1.0)
Java Persistence API (JPA) is a standard interface for accessing databases in Java, providing an automatic mapping between Java classes and database tables. There is an open-source plugin available for using JPA with Datastore, and this page provides information on how to get started with it.

**Warning:** We think most developers will have a better experience using the low-level Datastore API, or one of the open-source APIs developed specifically for Datastore, such as [Objectify](https://web.archive.org/web/20160424225528/https://github.com/objectify/objectify). JPA was designed for use with traditional relational databases, and so has no way to explicitly represent some of the aspects of Datastore that make it different from relational databases, such as entity groups and ancestor queries. This can lead to subtle issues that are difficult to understand and fix.

Version 1.x of the plugin is included in the App Engine Java SDK, which implements JPA version 1.0. The implementation is based on DataNucleus Access Platform version 1.1. See [the Access Platform 1.1 documentation](https://web.archive.org/web/20160424225528/http://www.datanucleus.org/products/accessplatform_1_1.html) for more information about the platform.

**Note:** The instructions on this page apply to JPA version 1, which uses version 1.x of the DataNucleus plugin for App Engine. Version 2.x of the DataNucleus plugin is also available, which allows you to use JPA 2.0. The 2.x plugin provides a number of new APIs and features; however, the upgrade is not fully backwards-compatible with the 1.x version. If you rebuild an application using JPA 2.0, you need to update and retest your code. For more information on the new version, see [Using JPA 2.0 with App Engine](https://web.archive.org/web/20160424225528/https://cloud.google.com/appengine/docs/java/datastore/jpa/overview-dn2.html).

1.  [Setting Up JPA 1.0](#Setting_Up_JPA)
2.  [Enhancing Data Classes](#Enhancing_Data_Classes)
3.  [Getting an EntityManager Instance](#Getting_an_EntityManager_Instance)
4.  [Class and Field Annotations](#Class_and_Field_Annotations)
5.  [Inheritance](#Inheritance)
6.  [Unsupported Features of JPA 1.0](#Unsupported_Features_of_JPA)

Setting Up JPA
--------------

To use JPA to access the datastore, an App Engine app needs the following:

*   The JPA and datastore JARs must be in the app's `war/WEB-INF/lib/` directory.
*   A configuration file named `persistence.xml` must be in the app's `war/WEB-INF/classes/META-INF/` directory, with configuration that tells JPA to use the App Engine datastore.
*   The project's build process must perform a post-compilation "enhancement" step on the compiled data classes to associate them with the JPA implementation.

If you are using [the Google Plugin for Eclipse](https://web.archive.org/web/20160424225528/https://developers.google.com/eclipse/docs/appengine), the first and third items are taken care of for you. The new project wizard puts the JPA and datastore JARs in the correct location and the build process performs the "enhancement" step automatically. You must still manually create `persistence.xml` and put it in `war/WEB-INF/classes/META-INF/`. The plugin will be updated soon to do this automatically as well.

If you are using Apache Ant to build your project, you can use an Ant task included with the SDK to perform the enhancement step. You must copy the JARs and create the configuration file when you set up your project. See [Using Apache Ant](https://web.archive.org/web/20160424225528/https://cloud.google.com/appengine/docs/java/tools/ant) for more information about the Ant task.

### Copying the JARs

The JPA and datastore JARs are included with the App Engine Java SDK. You can find them in the `appengine-java-sdk/lib/user/orm/` directory.

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
        <provider>org.datanucleus.store.appengine.jpa.DatastorePersistenceProvider</provider>
        <properties>
            <property name="datanucleus.NontransactionalRead" value="true"/>
            <property name="datanucleus.NontransactionalWrite" value="true"/>
            <property name="datanucleus.ConnectionURL" value="appengine"/>
        </properties>
    </persistence-unit>

</persistence>
```


### Datastore Read Policy and Call Deadline

As described on the [Datastore Queries](https://web.archive.org/web/20160424225528/https://cloud.google.com/appengine/docs/java/datastore/queries#Java_Data_consistency) page, you can set the read policy (strong consistency vs. eventual consistency) and the datastore call deadline for a `EntityManagerFactory` in the `persistence.xml` file. These settings go in the `<persistence-unit>` element. All calls made with a given `EntityManager` instance use the configuration selected when the manager was created by the `EntityManagerFactory`. You can also override these options for an individual `Query` (described below).

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
        <provider>org.datanucleus.store.appengine.jpa.DatastorePersistenceProvider</provider>
        <properties>
            <property name="datanucleus.NontransactionalRead" value="true"/>
            <property name="datanucleus.NontransactionalWrite" value="true"/>
            <property name="datanucleus.ConnectionURL" value="appengine"/>
        </properties>
    </persistence-unit>

    <persistence-unit name="eventual-reads-short-deadlines">
        <provider>org.datanucleus.store.appengine.jpa.DatastorePersistenceProvider</provider>
        <properties>
            <property name="datanucleus.NontransactionalRead" value="true"/>
            <property name="datanucleus.NontransactionalWrite" value="true"/>
            <property name="datanucleus.ConnectionURL" value="appengine"/>

            <property name="datanucleus.appengine.datastoreReadConsistency" value="EVENTUAL" />
            <property name="javax.persistence.query.timeout" value="5000" />
            <property name="datanucleus.datastoreWriteTimeout" value="10000" />
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

Enhancing Data Classes
----------------------

The DataNucleus implementation of JPA uses a post-compilation "enhancement" step in the build process to associate data classes with the JPA implementation.

If you are using Apache Ant, the SDK includes an Ant task to perform this step. See [Using Apache Ant](https://web.archive.org/web/20160424225528/https://cloud.google.com/appengine/docs/java/tools/ant) for more information on using the Ant task.

You can perform the enhancement step on compiled classes from the command line with the following command:

```
java -cp classpath org.datanucleus.enhancer.DataNucleusEnhancer
class-files
```


The _classpath_ must contain the JARs `datanucleus-core-*.jar`, `datanucleus-jpa-*`, `datanucleus-enhancer-*.jar`, `asm-*.jar`, and `geronimo-jpa-*.jar` (where `*` is the appropriate version number of each JAR) from the `appengine-java-sdk/lib/tools/` directory, as well as all of your data classes.

For more information on the DataNucleus bytecode enhancer, see [the DataNucleus documentation](https://web.archive.org/web/20160424225528/http://www.datanucleus.org/products/datanucleus/jpa/enhancer.html).

Getting an EntityManager Instance
---------------------------------

An app interacts with JPA using an instance of the `EntityManager` class. You get this instance by instantiating and calling a method on an instance of the `EntityManagerFactory` class. The factory uses the JPA configuration (identified by the name `"transactions-optional"`) to create `EntityManager` instances.

Because an `EntityManagerFactory` instance takes time to initialize, it's a good idea to reuse a single instance as much as possible. An easy way to do this is to create a singleton wrapper class with a static instance, as follows:

`**EMF.java**`

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


Class and Field Annotations
---------------------------

Each object saved by JPA becomes an entity in the App Engine datastore. The entity's kind is derived from the simple name of the class (without the package name). Each persistent field of the class represents a property of the entity, with the name of the property equal to the name of the field (with case preserved).

To declare a Java class as capable of being stored and retrieved from the datastore with JPA, give the class a `@Entity` annotation. For example:

```
import javax.persistence.Entity;

@Entity
public class Employee {
    // ...
}
```


Fields of the data class that are to be stored in the datastore must either be of a type that is persisted by default or expliclty declared as persistent. You can find a chart detailing JPA default persistence behavior on [the DataNucleus website](https://web.archive.org/web/20160424225528/http://www.datanucleus.org/products/accessplatform/jpa/types.html). To explicitly declare a field as persistent, you give it an `@Basic` annotation:

```
import java.util.Date;
import javax.persistence.Enumerated;

import com.google.appengine.api.datastore.ShortBlob;

// ...
    @Basic
    private ShortBlob data;
```


The type of a field can be any of the following:

*   one of the core types supported by the datastore
*   a Collection (such as a `java.util.List<...>`) of values of a core datastore type
*   an instance or Collection of instances of a `@Entity` class
*   an embedded class, stored as properties on the entity

A data class must have a public or protected default constructor and one field dedicated to storing the primary key of the corresponding datastore entity. You can choose between four different kinds of key fields, each using a different value type and annotations. (See [Creating Data: Keys](https://web.archive.org/web/20160424225528/https://cloud.google.com/appengine/docs/java/datastore/jdo/creatinggettinganddeletingdata#Keys) for more information.) The simplest key field is a long integer value that is automatically populated by JPA with a value unique across all other instances of the class when the object is saved to the datastore for the first time. Long integer keys use a `@Id` annotation, and a `@GeneratedValue(strategy = GenerationType.IDENTITY)` annotation:

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


Inheritance
-----------

JPA supports creating data classes that use inheritance. Before we talk about how JPA inheritance works on App Engine, we recommend you read [the DataNucleus documentation](https://web.archive.org/web/20160424225528/http://www.datanucleus.org/products/accessplatform/jpa/orm/inheritance.html) on this subject and then come back. Done? OK. JPA inheritance on App Engine works as described in the DataNucleus documentation with some additional restrictions. We'll discuss these restrictions and then give some concrete examples.

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

Mixing relationships with inheritance works so long as the declared types of your relationship fields match the runtime types of the objects you are assigning to those fields. Refer to the section on [Polymorphic Relationships](https://web.archive.org/web/20160424225528/https://cloud.google.com/appengine/docs/java/datastore/jdo/relationships#Polymorphic_Relationships) for more information. This section contains JDO examples, but the concepts and the restrictions are the same for JPA.

Unsupported Features of JPA 1.0
-------------------------------

The following features of the JPA interface are not supported by the App Engine implementation:

*   Owned many-to-many relationships, and unowned relationships. You can implement unowned relationships using explicit Key values, though type checking is not enforced in the API.
*   "Join" queries. You cannot use a field of a child entity in a filter when performing a query on the parent kind. Note that you can test the parent's relationship field directly in a query using a key.
*   Aggregation queries (group by, having, sum, avg, max, min)
*   Polymorphic queries. You cannot perform a query of a class to get instances of a subclass. Each class is represented by a separate entity kind in the datastore.
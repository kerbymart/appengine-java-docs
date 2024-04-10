# Defining Data Classes with JDO

  

You can use JDO to store plain Java data objects (sometimes referred to as "Plain Old Java Objects" or "POJOs") in the datastore. Each object that is made persistent with the PersistenceManager becomes an entity in the datastore. You use annotations to tell JDO how to store and recreate instances of your data classes.

**Note:** Earlier versions of JDO use `.jdo` XML files instead of Java annotations. These still work with JDO 2.3. This documentation only covers using Java annotations with data classes.

1.  [Class and Field Annotations](#Class_and_Field_Annotations)
2.  [Core Value Types](#Core_Value_Types)
3.  [Serializable Objects](#Serializable_Objects)
4.  [Child Objects and Relationships](#Child_Objects_and_Relationships)
5.  [Embedded Classes](#Embedded_Classes)
6.  [Collections](#Collections)
7.  [Object Fields and Entity Properties](#Object_Fields_and_Entity_Properties)
8.  [Inheritance](#Inheritance)

## Class and Field Annotations

Each object saved by JDO becomes an entity in the App Engine datastore. The entity's kind is derived from the simple name of the class (inner classes use the `$` path without the package name). Each persistent field of the class represents a property of the entity, with the name of the property equal to the name of the field (with case preserved).

To declare a Java class as capable of being stored and retrieved from the datastore with JDO, give the class a `@PersistenceCapable` annotation. For example:

```
import javax.jdo.annotations.PersistenceCapable;

@PersistenceCapable
public class Employee {
    // ...
}
```

Fields of the data class that are to be stored in the datastore must be declared as persistent fields. To declare a field as persistent, give it a `@Persistent` annotation:

```
import java.util.Date;
import javax.jdo.annotations.Persistent;

// ...
    @Persistent
    private Date hireDate;
```

To declare a field as not persistent (it does not get stored in the datastore, and is not restored when the object is retrieved), give it a `@NotPersistent` annotation.

**Tip:** JDO specifies that fields of certain types are persistent by default if neither the `@Persistent` nor `@NotPersistent` annotations are specified, and fields of all other types are not persistent by default. See [the DataNucleus documentation](https://web.archive.org/web/20160424225513/http://www.datanucleus.org/products/accessplatform_4_0/jdo/types.html) for a complete description of this behavior. Because not all of the App Engine datastore core [value types](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Properties_and_value_types) are persistent by default according to the JDO specification, we recommend explicitly annotating fields as `@Persistent` or `@NotPersistent` to make it clear.

The type of a field can be any of the following. These are described in detail below.

-   one of the core types supported by the datastore
-   a Collection (such as a `java.util.List<...>`) or an array of values of a core datastore type
-   an instance or Collection of instances of a `@PersistenceCapable` class
-   an instance or Collection of instances of a Serializable class
-   an embedded class, stored as properties on the entity

A data class must have one and only one field dedicated to storing the primary key of the corresponding datastore entity. You can choose between four different kinds of key fields, each using a different value type and annotations. (See [Creating Data: Keys](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/jdo/creatinggettinganddeletingdata#Keys) for more information.) The most flexible type of key field is a `Key` object which is automatically populated by JDO with a value unique across all other instances of the class when the object is saved to the datastore for the first time. Primary keys of type `Key` require a `@PrimaryKey` annotation and a `@Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)` annotation:

**Tip:** Make all of your persistent fields `private` or `protected` (or package protected), and only provide public access through accessor methods. Direct access to a persistent field from another class may bypass the JDO class enhancement. Alternatively, you can make other classes `@PersistenceAware`. See [the DataNucleus documentation](https://web.archive.org/web/20160424225513/http://www.datanucleus.org/products/accessplatform_3_3/datanucleus-accessplatform-docs.pdf) for more information.

```
import com.google.appengine.api.datastore.Key;

import javax.jdo.annotations.IdGeneratorStrategy;
import javax.jdo.annotations.PrimaryKey;

// ...
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;
```

Here is an example data class:

```
import com.google.appengine.api.datastore.Key;

import java.util.Date;
import javax.jdo.annotations.IdGeneratorStrategy;
import javax.jdo.annotations.PersistenceCapable;
import javax.jdo.annotations.Persistent;
import javax.jdo.annotations.PrimaryKey;

@PersistenceCapable
public class Employee {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private String firstName;

    @Persistent
    private String lastName;

    @Persistent
    private Date hireDate;

    public Employee(String firstName, String lastName, Date hireDate) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.hireDate = hireDate;
    }

    // Accessors for the fields. JDO doesn't use these, but your application does.

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

## Core Value Types

To represent a property containing a single value of a core type, declare a field of the Java type, and use the `@Persistent` annotation:

```
import java.util.Date;
import javax.jdo.annotations.Persistent;

// ...
    @Persistent
    private Date hireDate;
```

## Serializable Objects

A field value can contain an instance of a Serializable class, storing the serialized value of the instance in a single property value of the type Blob. To tell JDO to serialize the value, the field uses the annotation `@Persistent(serialized=true)`. Blob values are not indexed and cannot be used in query filters or sort orders.

Here is an example of a simple Serializable class that represents a file, including the file contents, a filename and a MIME type. This is not a JDO data class, so there are no persistence annotations.

```
import java.io.Serializable;

public class DownloadableFile implements Serializable {
    private byte[] content;
    private String filename;
    private String mimeType;

    // ... accessors ...
}
```

To store an instance of a Serializable class as a Blob value in a property, declare a field whose type is the class, and use the `@Persistent(serialized = "true")` annotation:

```
import javax.jdo.annotations.Persistent;
import DownloadableFile;

// ...
    @Persistent(serialized = "true")
    private DownloadableFile file;
```

## Child Objects and Relationships

A field value that is an instance of a `@PersistenceCapable` class creates an owned one-to-one relationship between two objects. A field that is a collection of such references creates an owned one-to-many relationship.

**Important:** Owned relationships have implications for transactions, entity groups, and cascading deletes. See [Transactions](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/transactions) and [Relationships](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/relationships) for more information.

Here is a simple example of an owned one-to-one relationship between an Employee object and a ContactInfo object:

**`ContactInfo.java`**

```
import com.google.appengine.api.datastore.Key;
// ... imports ...

@PersistenceCapable
public class ContactInfo {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private String streetAddress;

    @Persistent
    private String city;

    @Persistent
    private String stateOrProvince;

    @Persistent
    private String zipCode;

    // ... accessors ...
}
```

**`Employee.java`**

```
import ContactInfo;
// ... imports ...

@PersistenceCapable
public class Employee {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private ContactInfo myContactInfo;

    // ... accessors ...
}
```

In this example, if the app creates an Employee instance, populates its `myContactInfo` field with a new ContactInfo instance, then saves the Employee instance with `pm.makePersistent(...)`, the datastore creates two entities. One is of the kind `"ContactInfo"`, representing the ContactInfo instance. The other is of the kind `"Employee"`. The key of the `ContactInfo` entity has the key of the `Employee` entity as its entity group parent.

## Embedded Classes

Embedded classes allow you to model a field value using a class without creating a new datastore entity and forming a relationship. The fields of the object value are stored directly in the datastore entity for the containing object.

Any `@PersistenceCapable` data class can be used as an embedded object in another data class. The class's `@Persistent` fields are embedded in the object. If you give the class to embed the `@EmbeddedOnly` annotation, the class can only be used as an embedded class. The embedded class does not need a primary key field because it is not stored as a separate entity.

Here is an example of an embedded class. This example makes the embedded class an inner class of the data class that uses it; this is useful, but not required to make a class embeddable.

```
import javax.jdo.annotations.Embedded;
import javax.jdo.annotations.EmbeddedOnly;
// ... imports ...

@PersistenceCapable
public class EmployeeContacts {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    Key key;
    @PersistenceCapable
    @EmbeddedOnly
    public static class ContactInfo {
        @Persistent
        private String streetAddress;

        @Persistent
        private String city;

        @Persistent
        private String stateOrProvince;

        @Persistent
        private String zipCode;

        // ... accessors ...
    }

    @Persistent
    @Embedded
    private ContactInfo homeContactInfo;
}
```

The fields of an embedded class are stored as properties on the entity, using the name of each field and the name of the corresponding property. If you have more than one field on the object whose type is an embedded class, you must rename the fields of one so they do not conflict with another. You specify new field names using arguments to the `@Embedded` annotation. For example:

```
    @Persistent
    @Embedded
    private ContactInfo homeContactInfo;

    @Persistent
    @Embedded(members = {
        @Persistent(name="streetAddress", columns=@Column(name="workStreetAddress")),
        @Persistent(name="city", columns=@Column(name="workCity")),
        @Persistent(name="stateOrProvince", columns=@Column(name="workStateOrProvince")),
        @Persistent(name="zipCode", columns=@Column(name="workZipCode")),
    })
    private ContactInfo workContactInfo;
```

Similarly, fields on the object must not use names that collide with fields of embedded classes, unless the embedded fields are renamed.

Because the embedded class's persistent properties are stored on the same entity as the other fields, you can use persistent fields of the embedded class in JDOQL query filters and sort orders. You can refer to the embedded field using the name of the outer field, a dot (`.`), and the name of the embedded field. This works whether or not the property names for the embedded fields have been changed using `@Column` annotations.

```
    select from EmployeeContacts where workContactInfo.zipCode == "98105"
```

## Collections

A datastore property can have more than one value. In JDO, this is represented by a single field with a Collection type, where the collection is of one of the core [value types](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/entities#Java_Properties_and_value_types) or a Serializable class. The following Collection types are supported:

-   `java.util.ArrayList<...>`
-   `java.util.HashSet<...>`
-   `java.util.LinkedHashSet<...>`
-   `java.util.LinkedList<...>`
-   `java.util.List<...>`
-   `java.util.Map<...>`
-   `java.util.Set<...>`
-   `java.util.SortedSet<...>`
-   `java.util.Stack<...>`
-   `java.util.TreeSet<...>`
-   `java.util.Vector<...>`

If a field is declared as a List, objects returned by the datastore have an ArrayList value. If a field is declared as a Set, the datastore returns a HashSet. If a field is declared as a SortedSet, the datastore returns a TreeSet.

For example, a field whose type is `List<String>` is stored as zero or more string values for the property, one for each value in the `List`.

```
import java.util.List;
// ... imports ...

// ...
    @Persistent
    List<String> favoriteFoods;
```

A Collection of child objects (of `@PersistenceCapable` classes) creates multiple entities with a one-to-many relationship. See [Relationships](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/relationships).

Datastore properties with more than one value have special behavior for query filters and sort orders. See the [Datastore Queries](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/queries#Surprising_behavior) page for more information.

## Object Fields and Entity Properties

The App Engine datastore distinguishes between an entity without a given property and an entity with a `null` value for a property. JDO does not support this distinction: every field of an object has a value, possibly `null`. If a field with a nullable value type (something other than a built-in type like `int` or `boolean`) is set to `null`, when the object is saved, the resulting entity will have the property set with a null value.

If a datastore entity is loaded into an object and doesn't have a property for one of the object's fields and the field's type is a nullable single-value type, the field is set to `null`. When the object is saved back to the datastore, the `null` property becomes set in the datastore to the null value. If the field is not of a nullable value type, loading an entity without the corresponding property throws an exception. This won't happen if the entity was created from the same JDO class used to recreate the instance, but can happen if the JDO class changes, or if the entity was created using the low-level API instead of JDO.

If a field's type is a Collection of a core data type or a Serializable class and there are no values for the property on the entity, the empty collection is represented in the datastore by setting the property to a single null value. If the field's type is an array type, it is assigned an array of 0 elements. If the object is loaded and there is no value for the property, the field is assigned an empty collection of the appropriate type. Internally, the datastore knows the difference between an empty collection and a collection containing one null value.

If the entity has a property without a corresponding field in the object, that property is inaccessible from the object. If the object is saved back to the datastore, the extra property is deleted.

If an entity has a property whose value is of a different type than the corresponding field in the object, JDO attempts to cast the value to the field type. If the value cannot be cast to the field type, JDO throws a ClassCastException. In the case of numbers (long integers and double-width floats), the value is converted, not cast. If the numeric property value is larger than the field type, the conversion overflows without throwing an exception.

You can declare a property unindexed by adding the line

```
    @Extension(vendorName="datanucleus", key="gae.unindexed", value="true")
```

above the property in the class definition. See the [Unindexed Properties](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/indexes#Java_Unindexed_properties) section of the main docs for additional information about what it means for a property to be unindexed.

## Inheritance

Creating data classes that make use of inheritance is a natural thing to do, and JDO supports this. Before we talk about how JDO inheritance works on App Engine we recommend you read [the DataNucleus documentation on this subject](https://web.archive.org/web/20160424225513/http://www.datanucleus.org/products/accessplatform/jdo/orm/inheritance.html) and then come back. Done? Ok. JDO inheritance on App Engine works as described in the DataNucleus documentation with some additional restrictions. We'll discuss these restrictions and then give some concrete examples.

The "new-table" inheritance strategy allows you to split the data for a single data object across multiple "tables," but since the App Engine datastore does not support joins, operating on a data object with this inheritance strategy requires a remote procedure call for each level of inheritance. This is potentially very inefficient, so the "new-table" inheritance strategy is not supported on data classes that are not at the root of their inheritance hierarchies.

Second, the "superclass-table" inheritance strategy allows you to store the data for a data object in the "table" of its superclass. Although there are no inherent inefficiencies in this strategy, it is not currently supported. We may revisit this in future releases.

Now the good news: The "subclass-table" and "complete-table" strategies work as described in the DataNucleus documentation, and you can also use "new-table" for any data object that is at the root of its inheritance hierarchy. Let's look at an example:

**`Worker.java`**

```
import javax.jdo.annotations.IdGeneratorStrategy;
import javax.jdo.annotations.Inheritance;
import javax.jdo.annotations.InheritanceStrategy;
import javax.jdo.annotations.PersistenceCapable;
import javax.jdo.annotations.Persistent;
import javax.jdo.annotations.PrimaryKey;

@PersistenceCapable
@Inheritance(strategy = InheritanceStrategy.SUBCLASS_TABLE)
public abstract class Worker {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private String department;
}
```

**`Employee.java`**

```
// ... imports ...

@PersistenceCapable
public class Employee extends Worker {
    @Persistent
    private int salary;
}
```

**`Intern.java`**

```
import java.util.Date;
// ... imports ...

@PersistenceCapable
public class Intern extends Worker {
    @Persistent
    private Date internshipEndDate;
}
```

In this example we've added an `@Inheritance` annotation to the `Worker`class declaration with its `strategy>` attribute set to `InheritanceStrategy.SUBCLASS_TABLE`. This tells JDO to store all persistent fields of the `Worker` in the datastore entities of its subclasses. The datastore entity created as the result of calling `makePersistent()` with an `Employee` instance has two properties named "department" and "salary". The datastore entity created as the result of calling `makePersistent()` with an `Intern` instance will have two properties named "department" and "internshipEndDate". The datastore does not contain any entities of kind "Worker".

Now let's make things a little more interesting. Suppose, in addition to having `Employee` and `Intern`, we also want a specialization of `Employee` that describes employees who have left the company:

**`FormerEmployee.java`**

```
import java.util.Date;
// ... imports ...

@PersistenceCapable
@Inheritance(customStrategy = "complete-table")
public class FormerEmployee extends Employee {
    @Persistent
    private Date lastDay;
}
```

In this example, we've added an `@Inheritance` annotation to the `FormerEmployee` class declaration with its `custom-strategy>` attribute set to "complete-table". This tells JDO to store all persistent fields of the `FormerEmployee` and its superclasses in datastore entities corresponding to `FormerEmployee` instances. The datastore entity created as the result of calling `makePersistent()` with an `FormerEmployee` instance will have three properties named "department", "salary", and "lastDay". No entity of kind "Employee"" corresponds to a `FormerEmployee`. However, if you call `makePersistent()` with a object whose runtime type is `Employee`, you create an entity of kind "Employee".

Mixing relationships with inheritance works so long as the declared types of your relationship fields match the runtime types of the objects you are assigning to those fields. Please refer to the section on [Polymorphic Relationships](https://web.archive.org/web/20160424225513/https://cloud.google.com/appengine/docs/java/datastore/jdo/relationships#Polymorphic_Relationships) for more information.

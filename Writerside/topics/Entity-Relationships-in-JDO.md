# Entity Relationships in JDO

  

You can model relationships between persistent objects using fields of the object types. A relationship between persistent objects can be described as *owned*, where one of the objects cannot exist without the other, or *unowned*, where both objects can exist independently of their relationship with one another. The App Engine implementation of the JDO interface can model both owned and unowned one-to-one relationships and one-to-many relationships, both unidirectional and bidirectional.

Unowned relationships are not supported in version 1.0 of the DataNucleus plugin for App Engine, but you can manage these relationships yourself by storing datastore keys in fields directly. App Engine creates related entities in entity groups automatically to support updating related objects together— but it is the app's responsibility to know when to use datastore transactions.

Version 2.x of the DataNucleus plugin for App Engine supports unowned relationships with a natural syntax. The [Unowned Relationships](#Unowned_Relationships) section calls out how to create unowned relationships in each plugin version. To upgrade to version 2.x of the DataNucleus plugin for App Engine, see [Migrating to Version 2.x of the DataNucleus Plugin for App Engine](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/jdo/overview-dn2#Migrating_to_Version_2.0_of_the_DataNucleus_Plugin).

1.  [Owned One-to-One Relationships](#Owned_One_to_One_Relationships)
2.  [Owned One-to-Many Relationships](#Owned_One_to_Many_Relationships)
3.  [Unowned Relationships](#Unowned_Relationships)
4.  [Relationships, Entity Groups, and Transactions](#Relationships_Entity_Groups_and_Transactions)
5.  [Dependent Children and Cascading Deletes](#Dependent_Children_and_Cascading_Deletes)
6.  [Polymorphic Relationships](#Polymorphic_Relationships)

## Owned One-to-One Relationships

You create a unidirectional one-to-one owned relationship between two persistent objects by using a field whose type is the class of the related class.

The following example defines a ContactInfo data class and an Employee data class, with a one-to-one relationship from Employee to ContactInfo.

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

    // ...
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
    private ContactInfo contactInfo;

    ContactInfo getContactInfo() {
        return contactInfo;
    }
    void setContactInfo(ContactInfo contactInfo) {
        this.contactInfo = contactInfo;
    }

    // ...
}
```

The persistent objects are represented as two distinct entities in the datastore, with two different kinds. The relationship is represented using an entity group relationship: the child's key uses the parent's key as its entity group parent. When the app accesses the child object using the parent object's field, the JDO implementation performs an entity group parent query to get the child.

The child class must have a key field whose type can contain the parent key information: either a Key, or a Key value encoded as a string. See [Creating Data: Keys](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/jdo/creatinggettinganddeletingdata#Keys) for information on key field types.

You create a bidirectional one-to-one relationship using fields on both classes, with an annotation on the child class's field to declare that the fields represent a bidirectional relationship. The field of the child class must have a `@Persistent` annotation with the argument `mappedBy = "..."`, where the value is the name of the field on the parent class. If the field on one object is populated, then the corresponding reference field on the other object is populated automatically.

**`ContactInfo.java`**

```
import Employee;

// ...
    @Persistent(mappedBy = "contactInfo")
    private Employee employee;
```

Child objects are loaded from the datastore when they are accessed for the first time. If you do not access the child object on a parent object, the entity for the child object is never loaded. If you want to load the child, you can either "touch" it *before* closing the PersistenceManager (e.g. by calling `getContactInfo()` in the above example) or explicitly add the child field to the default fetch group so it's retrieved and loaded with the parent:

**`Employee.java`**

```
import ContactInfo;

// ...
    @Persistent(defaultFetchGroup = "true")
    private ContactInfo contactInfo;
```

## Owned One-to-Many Relationships

To create a one-to-many relationship between objects of one class and multiple objects of another, use a Collection of the related class:

**`Employee.java`**

```
import java.util.List;

// ...
    @Persistent
    private List<ContactInfo> contactInfoSets;
```

A one-to-many bidirectional relationship is similar to a one-to-one, with a field on the parent class using the annotation `@Persistent(mappedBy = "...")`, where the value is the name of the field on the child class:

**`Employee.java`**

```
import java.util.List;

// ...
    @Persistent(mappedBy = "employee")
    private List<ContactInfo> contactInfoSets;
```

**`ContactInfo.java`**

```
import Employee;

// ...
    @Persistent
    private Employee employee;
```

The collection types listed in [Defining Data Classes: Collections](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/jdo/dataclasses#Collections) are supported for one-to-many relationships. However, arrays are *not* supported for one-to-many relationships.

App Engine does not support join queries: you cannot query a parent entity using an attribute of a child entity. (You can query a property of an embedded class, because embedded classes store properties on the parent entity. See [Defining Data Classes: Embedded Classes](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/jdo/dataclasses#Embedded_Classes).)

### How Ordered Collections Maintain Their Order

Ordered collections, such as `List<...>`, preserve the order of objects when the parent object is saved. JDO requires that databases preserve this order by storing the position of each object as a property of the object. App Engine stores this as a property of the corresponding entity, using a property name equal to the name of the parent's field followed by `_INTEGER_IDX`. **Position properties are inefficient.** If an element is added, removed or moved in the collection, all entities subsequent to the modified place in the collection must be updated. This can be slow, and error prone if not performed in a transaction.

If you do not need to preserve an arbitrary order in a collection, but need to use an ordered collection type, you can specify an ordering based on properties of the elements using an annotation, an extension to JDO provided by DataNucleus:

```
import java.util.List;
import javax.jdo.annotations.Extension;
import javax.jdo.annotations.Order;
import javax.jdo.annotations.Persistent;

// ...
    @Persistent
    @Order(extensions = @Extension(vendorName="datanucleus",key="list-ordering", value="state asc, city asc"))
    private List<ContactInfo> contactInfoSets = new ArrayList<ContactInfo>();
```

The `@Order` annotation (using the `list-ordering` extension) specifies the desired order of the elements of the collection as a JDOQL ordering clause. The ordering uses values of properties of the elements. As with queries, all elements of a collection must have values for the properties used in the ordering clause.

Accessing a collection performs a query. If the ordering clause of a field uses more than one sort order, the query requires a Datastore index; see the [Datastore Indexes](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/indexes) page for more information.

For efficiency, always use an explicit ordering clause for one-to-many relationships of ordered collection types, if possible.

## Unowned Relationships

In addition to owned relationships, the JDO API also provides a facility for managing unowned relationships. This facility works differently depending on which version of the DataNucleus plugin for App Engine you are using:

-   [Version 1](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/jdo/overview) of the DataNucleus plugin does not implement unowned relationships using a natural syntax, but you can still manage these relationships using `Key` values in place of instances (or Collections of instances) of your model objects. You can think of storing Key objects as modeling an arbitrary "foreign key" between two objects. The datastore does not guarantee referential integrity with these Key references, but the use of Key makes it very easy to model (and then fetch) any relationship between two objects.  
      
    However, if you go this route, you must ensure that the keys are of the appropriate kind. JDO and the compiler do not check `Key` types for you.
-   [Version 2.x](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/jdo/overview-dn2) of the DataNucleus plugin implements unowned relationships using a natural syntax.

**Tip:** In some cases, you may find it necessary to model an owned relationship as if it is unowned. This is because all objects involved in an owned relationship are automatically placed in the same entity group, and an entity group can only support one to ten writes per second. So, for example, if a parent object is receiving .75 writes per second and a child object is receiving .75 writes per second, it may make sense to model this relationship as unowned so that both parent and child reside in their own, independent entity groups.

### Unowned One-to-One Relationships

Suppose you want to model person and food, where a person can only have one favorite food but a favorite food does not belong to the person because it can be the favorite food of any number of people. This section shows how to do that.

#### In JDO 2.3

In this example, we give `Person` a member of type `Key`, where the `Key` is the unique identifier of a `Food` object. If an instance of `Person` and the instance of `Food` referred to by `Person.favoriteFood` are *not* in the same entity group, you cannot update the person and that person's favorite food in a single transaction unless your JDO configuration is set to [enable cross-group (XG) transactions](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/transactions#Cross_Group_Transactions).

**`Person.java`**

```
// ... imports ...

@PersistenceCapable
public class Person {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private Key favoriteFood;

    // ...
}
```

**`Food.java`**

```
import Person;
// ... imports ...

@PersistenceCapable
public class Food {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    // ...
}
```

#### In JDO 3.0

In this example, rather than giving `Person` a key representing their favorite food, we create a private member of type `Food`:

**`Person.java`**

```
// ... imports ...

@PersistenceCapable
public class Person {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    @Unowned
    private Food favoriteFood;

    // ...
}
```

**`Food.java`**

```
import Person;
// ... imports ...

@PersistenceCapable
public class Food {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    // ...
}
```

### Unowned One-to-Many Relationships

Now suppose we want to let a person have multiple favorite foods. Again, a favorite food does not belong to the person because it can be the favorite food of any number of people.

#### In JDO 2.3

In this example, rather than giving Person a member of type `Set<Food>` to represent the person's favorite foods, we instead give Person a member of type `Set<Key>`, where the set contains the unique identifiers of `Food` objects. Note that, if an instance of `Person` and an instance of `Food` contained in `Person.favoriteFoods` are not in the same entity group, you must set your JDO configuration to [enable cross-group (XG) transactions](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/transactions#Cross_Group_Transactions) if you want to update them in the same transaction.

**`Person.java`**

```
// ... imports ...

@PersistenceCapable
public class Person {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private Set<Key> favoriteFoods;

    // ...
}
```

#### In JDO 3.0

In this example, we give Person a member of type `Set<Food>` where the set represents the Person's favorite foods.

**`Person.java`**

```
// ... imports ...

@PersistenceCapable
public class Person {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private Set<Food> favoriteFoods;

    // ...
}
```

### Many-to-Many Relationships

We can model a many-to-many relationship by maintaining collections of keys on both sides of the relationship. Let's adjust our example to let `Food` keep track of the people that consider it a favorite:

**`Person.java`**

```
import java.util.Set;
import com.google.appengine.api.datastore.Key;

// ...
    @Persistent
    private Set<Key> favoriteFoods;
```

**`Food.java`**

```
import java.util.Set;
import com.google.appengine.api.datastore.Key;

// ...
    @Persistent
    private Set<Key> foodFans;
```

In this example, the `Person` maintains a Set of `Key` values that uniquely identify the `Food` objects that are favorites, and the `Food` maintains a Set of `Key` values that uniquely identify the `Person` objects that consider it a favorite.

When modeling a many-to-many using `Key` values, be aware that it is the app's responsibility to maintain both sides of the relationship:

**`Album.java`**

```

// ...
public void addFavoriteFood(Food food) {
    favoriteFoods.add(food.getKey());
    food.getFoodFans().add(getKey());
}

public void removeFavoriteFood(Food food) {
    favoriteFoods.remove(food.getKey());
    food.getFoodFans().remove(getKey());
}
```

If an instance of `Person` and an instance of `Food` contained in `Person.favoriteFoods` are not in the same entity group, and you want to update them in a single transaction, you must set your JDO configuration to [enable cross-group (XG) transactions](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/transactions#Cross_Group_Transactions).

## Relationships, Entity Groups, and Transactions

When your application saves an object with owned relationships to the datastore, all other objects that can be reached via relationships and need to be saved (they are new or have been modified since they were last loaded) are saved automatically. This has important implications for transactions and entity groups.

Consider the following example using a unidirectional relationship between the `Employee` and `ContactInfo` classes above:

```
    Employee e = new Employee();
    ContactInfo ci = new ContactInfo();
    e.setContactInfo(ci);

    pm.makePersistent(e);
```

When the new `Employee` object is saved using the `pm.makePersistent()` method, the new related `ContactInfo` object is saved automatically. Since both objects are new, App Engine creates two new entities in the same entity group, using the `Employee` entity as the parent of the `ContactInfo` entity. Similarly, if the `Employee` object has already been saved and the related `ContactInfo` object is new, App Engine creates the `ContactInfo` entity using the existing `Employee` entity as the parent.

Notice, however, that the call to `pm.makePersistent()` in this example does not use a transaction. Without an explicit transaction, both entities are created using separate atomic actions. In this case, it is possible for the creation of the Employee entity to succeed, but the creation of the ContactInfo entity to fail. To ensure that either both entities are created successfully or neither entity is created, you must use a transaction:

```
    Employee e = new Employee();
    ContactInfo ci = new ContactInfo();
    e.setContactInfo(ci);

    try {
        Transaction tx = pm.currentTransaction();
        tx.begin();
        pm.makePersistent(e);
        tx.commit();
    } finally {
        if (tx.isActive()) {
            tx.rollback();
        }
    }
```

If both objects were saved *before* the relationship was established, App Engine cannot "move" the existing `ContactInfo` entity into the `Employee` entity's entity group, because entity groups can only be assigned when the entities are created. App Engine can establish the relationship with a reference, but the related entities will not be in the same group. In this case, the two entities can be updated or deleted in the same transaction if you set your JDO configuration to [enable cross-group (XG) transactions](https://web.archive.org/web/20160424225913/https://cloud.google.com/appengine/docs/java/datastore/transactions#Cross_Group_Transactions). If you don't use XG transactions, the attempt to update or delete entities of different groups in the same transaction will throw a [JDOFatalUserException](https://web.archive.org/web/20160424225913/http://db.apache.org/jdo/api20/apidocs/javax/jdo/JDOFatalUserException.html).

Saving a parent object whose child objects have been modified will save the changes to the child objects. It's a good idea to allow parent objects to maintain persistence for all related child objects in this way, and to use transactions when saving changes.

## Dependent Children and Cascading Deletes

An owned relationship can be "dependent," meaning that the child cannot exist without its parent. If a relationship is dependent and a parent object is deleted, all child objects are also deleted. Breaking an owned, dependent relationship by assigning a new value to the dependent field on the parent also deletes the old child. You can declare an owned one-to-one relationship to be dependent by adding `dependent="true"` to the `Persistent` annotation of the field on the parent object that refers to the child:

```
// ...
    @Persistent(dependent = "true")
    private ContactInfo contactInfo;
```

You can declare an owned one-to-many relationship to be dependent by adding an `@Element(dependent = "true")` annotation to the field on the parent object that refers to the child collection:

```
import javax.jdo.annotations.Element;
// ...
    @Persistent
    @Element(dependent = "true")
    private List contactInfos;
```

As with creating and updating objects, if you need every delete in a cascading delete to occur in a single atomic action, you must perform the delete in a transaction.

**Note:** The JDO implementation does the work to delete dependent child objects, *not* the datastore. If you delete a parent entity using the low-level API or the Google Cloud Platform Console, the related child objects will not be deleted.

## Polymorphic Relationships

Even though the JDO specification includes support for polymorphic relationships, polymorhpic relationships are not yet supported in the App Engine DO implementation. This is a limitation we hope to remove in future releases of the product. If you need to refer to multiple types of objects via a common base class, we recommend the same strategy used for implementing unowned relationships: store a Key reference. For example, if you have a `Recipe` base class with `Appetizer`, `Entree`, nd `Dessert` specializations, and you want to model the favorite `Recipe` of a `Chef`, you can model it as follows:

**`Recipe.java`**

```
import javax.jdo.annotations.IdGeneratorStrategy;
import javax.jdo.annotations.Inheritance;
import javax.jdo.annotations.InheritanceStrategy;
import javax.jdo.annotations.PersistenceCapable;
import javax.jdo.annotations.Persistent;
import javax.jdo.annotations.PrimaryKey;

@PersistenceCapable
@Inheritance(strategy = InheritanceStrategy.SUBCLASS_TABLE)
public abstract class Recipe {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private int prepTime;
}
```

**`Appetizer.java`**

```
// ... imports ...

@PersistenceCapable
public class Appetizer extends Recipe {
// ... appetizer-specific fields
}
```

**`Entree.java`**

```
// ... imports ...

@PersistenceCapable
public class Entree extends Recipe {
// ... entree-specific fields
}
```

**`Dessert.java`**

```
// ... imports ...

@PersistenceCapable
public class Dessert extends Recipe {
// ... dessert-specific fields
}
```

**`Chef.java`**

```
// ... imports ...

@PersistenceCapable
public class Chef {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent(dependent = "true")
    private Recipe favoriteRecipe;
}
```

Unfortunately, if you instantiate an `Entree` and assign it to `Chef.favoriteRecipe` you will get an `UnsupportedOperationException` when you try to persist the `Chef` object. This is because the runtime type of the object, `Entree`, does not match the declared type of relationship field, `Recipe`. The workaround is to change the type of `Chef.favoriteRecipe` from a `Recipe` to a `Key`:

**`Chef.java`**

```
// ... imports ...

@PersistenceCapable
public class Chef {
    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.IDENTITY)
    private Key key;

    @Persistent
    private Key favoriteRecipe;
}
```

Since `Chef.favoriteRecipe` is no longer a relationship field, it can refer to an object of any type. The downside is that, as with an unowned relationship, you need to manage this relationship manually.

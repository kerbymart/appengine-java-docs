# Multiclass APIs

  

1.  [Using @ApiClass for Properties that Can Differ Between Classes](#using_apiclass_for_properties_that_can_differ_between_classes)
2.  [Annotation Inheritance](#annotation_inheritance)
    1.  [Using Java Inheritance](#using_java_inheritance)
    2.  [Using @ApiReference Inheritance](#using_apireference_inheritance)
    3.  [Overriding Inherited Configuration](#overriding_inherited_configuration)
    4.  [Inheriting @ApiMethod Annotations](#inheriting_apimethod_annotations)
3.  [Inheritance and Precedence Rules](#inheritance_and_precedence_rules)
4.  [Common Use Cases for Annotation Inheritance](#common_use_cases_for_annotation_inheritance)

If a single API is particularly complex, you may wish to implement it from multiple Java classes. To make different classes part of the same API, simply give each class the same `name` and `version` strings in their [`@Api` annotation](https://web.archive.org/web/20160424225413/https://cloud.google.com/appengine/docs/java/endpoints/annotations#api_api-scoped_annotations). For example, the following two classes will both be part of the `tictactoe` API:

```
@Api(name = "tictactoe", version = "v1")
class TicTacToeA {  }

@Api(name = "tictactoe", version = "v1")
class TicTacToeB {  }
```

The API configuration is specified through the `@Api` annotation properties. However, for multiple classes in the same API, the `@Api` requirements extend beyond simply having the same `name` and `version` strings in the `@Api` annotation for each class. In fact, your backend API will not work if there are *any* differences in the API configurations specified in the classes' `@Api` properties. Any difference in the `@Api` properties for classes in a multiclass API result in an "ambiguous" API configuration, which will not work in Endpoints.

**Note:** for a way to specify different properties for classes without using `@Api` properties, see [Using `@ApiClass` for Properties that Can Differ Between Classes](#using_apiclass_for_properties_that_can_differ_between_classes).

There are several ways to create a unambiguous multiclass API:

-   Manually ensure that all classes in a single API have the exact same `@Api` annotation properties. We won't describe this further, since it is self-evident.
-   Use annotation inheritance through [Java inheritance](#using_java_inheritance). In this inheritance, all classes in a single API inherit the same API configuration from a common `@Api`-annotated base class.
-   Use annotation inheritance through the [`@ApiReference`](#using_apireference_inheritance) annotation on all classes in a single API to have them reference the same API configuration from a common @Api-annotated class.

## Using `@ApiClass` for Properties that Can Differ Between Classes

While all properties in the `@Api` annotation must match for all classes in an API, you can additionally use the `@ApiClass` annotation to provides properties that do not need to be exactly the same between classes. For example:

```
// API methods implemented in this class allow only "clientIdA".
@Api(name = "tictactoe", version = "v1")
@ApiClass(clientIds = { "clientIdA" })
class TicTacToeA {  }

// API methods implemented in this class provide unauthenticated access.
@Api(name = "tictactoe", version = "v1")
class TicTacToeB {  }
```

where `TicTacToeA` limits access using a whitelist of client IDs containing the allowed client ID, and `TicTacToeB` does not limit acccess.

All properties provided by the `@ApiClass` annotation have an equivalent property in the `@Api` annotation. Note that the `@Api` equivalent property acts as the API-wide default. If there is an API-wide default for that same property, specified in `@Api`, the class-specific `@ApiClass` property will override the API-wide default.

The following examples illustrate the overriding of `@Api` properties by the class specific `@ApiClass` equivalents:

```
// For this class "boards" overrides "games".
@Api(name = "tictactoe", version = "v1", resource = "games")
@ApiClass(resource = "boards")
class TicTacToeBoards {  }

// For this class "scores" overrides "games".
@Api(name = "tictactoe", version = "v1", resource = "games")
@ApiClass(resource = "scores")
class TicTacToeScores {  }

// For this class, the API-wide default "games" is used as the resource.
@Api(name = "tictactoe", version = "v1", resource = "games")
class TicTacToeGames {  }
```

## Annotation Inheritance

The `@Api` and `@ApiClass` annotation properties can be inherited from from other classes, and individual properties can be overridden either through [Java Inheritance](#using_java_inheritance) or [`@ApiReference` inheritance](#using_apireference_inheritance)

### Using Java Inheritance

A class that extends another class with `@Api` or `@ApiClass` annotations will behave as if annotated with the same properties. For example:

```
@Api(name = "tictactoe", version = "v1")
class TicTacToeBase {  }

// TicTacToeA and TicTacToeB both behave as if they have the same @Api annotation as
// TicTacToeBase
class TicTacToeA extends TicTacToeBase {  }
class TicTacToeB extends TicTacToeBase {  }
```

Annotations are only inherited through Java subclassing, not through interface implementation. For example:

```
@Api(name = "tictactoe", version = "v1")
interface TicTacToeBase {  }
// Does *not* behave as if annotated.
class TicTacToeA implements TicTacToeBase {  }
```

As a result, *there is no support for any sort of multiple inheritance* of Endpoints annotations.

Inheritance works for `@ApiClass` too:

```
@ApiClass(resource = "boards")
class BoardsBase {  }

// TicTacToeBoards behaves as if annotated with the @ApiClass from BoardsBase.
// Thus, the "resource" property will be "boards".
@Api(name = "tictactoe", version = "v1", resource = "scores")
class TicTacToeBoards extends BoardsBase {  }
```

where `TicTacToeBoards` inherits the `resource` property value `boards` from `BoardsBase`, thus overriding the `resource` property setting (`scores`) in its `@Api` annotation. Remember that if any class has specified the resource property in the `@Api` annotation, all of the classes need to specify that same setting in the `@Api` annotation; this inheritance technique lets you override that `@Api` property.

### Using @ApiReference Inheritance

The `@ApiReference` annotation provides an alternate way to specify annotation inheritance. A class that uses `@ApiReference` to specify another class with `@Api` or `@ApiClass` annotations will behave as if annotated with the same properties. For example:

```
@Api(name = "tictactoe", version = "v1")
class TicTacToeBase {  }

// TicTacToeA behaves as if it has the same @Api annotation as TicTacToeBase
@ApiReference(TicTacToeBase.class)
class TicTacToeA {  }
```

If both Java inheritance and the `@ApiReference` are used, the annotations will inherit through the `@ApiReference` annotation only. `@Api` and `@ApiClass` annotations on the class inherited through Java inheritance will be ignored. For example:

```
@Api(name = "tictactoe", version = "v1")
class TicTacToeBaseA {  }
@Api(name = "tictactoe", version = "v2")
class TicTacToeBaseB {  }

// TicTacToe will behave as if annotated the same as TicTacToeBaseA, not TicTacToeBaseB.
// The value of the "version" property will be "v1".
@ApiReference(TicTacToeBaseA.class)
class TicTacToe extends TicTacToeBaseB {  }
```

### Overriding Inherited Configuration

Whether inheriting configuration using Java inheritance or `@ApiReference`, the inherited configuration can be overridden using a new `@Api` or `@ApiClass` annotation. Only configuration properties specified in the new annotation are overridden. Properties that are unspecified are still inherited. For example:

```
@Api(name = "tictactoe", version = "v2")
class TicTacToe {  }

// Checkers will behave as if annotated with name = "checkers" and version = "v2"
@Api(name = "checkers")
class Checkers extends TicTacToe {  }
```

Overriding inheritance works for `@ApiClass` too:

```
@Api(name = "tictactoe", version = "v1")
@ApiClass(resource = "boards", clientIds = { "c1" })
class Boards {  }

// Scores will behave as if annotated with resource = "scores" and clientIds = { "c1" }
@ApiClass(resource = "scores")
class Scores {  }
```

Overriding also works when inheriting through `@ApiReference`:

```
@Api(name = "tictactoe", version = "v2")
class TicTacToe {  }

// Checkers will behave as if annotated with name = "checkers" and version = "v2"
@ApiReference(TicTacToe.class)
@Api(name = "checkers")
class Checkers {  }
```

### Inheriting @ApiMethod Annotations

The `@ApiMethod` annotation can be inherited from overridden methods. For example:

```
class TicTacToeBase {
  @ApiMethod(httpMethod = "POST")
  public Game setGame(Game game) {  }
}
@Api(name = "tictactoe", version = "v1")
class TicTacToe extends TicTacToeBase {
  // setGame behaves as if annotated with the @ApiMethod from TicTacToeBase.setGame.
  // Thus the "httpMethod" property will be "POST".
  @Override
  public Game setGame(Game game) {  }
}
```

Similarly to `@Api` and `@ApiClass` annotation inheritance, if multiple methods overriding each other have `@ApiMethod` annotations, individual properties can be overridden. For example:

```
class TicTacToeBase {
  @ApiMethod(httpMethod = "POST", clientIds = { "c1" })
  public Game setGame(Game game) {  }
}
@Api(name = "tictactoe", version = "v1")
class TicTacToe extends TicTacToeBase {
  // setGame behaves as if annotated with httpMethod = "GET" and clientIds = { "c1"}.
  @ApiMethod(httpMethod = "GET")
  @Override
  public Game setGame(Game game) {  }
}
```

There is no `@ApiReference` annotation or equivalent for methods, so `@ApiMethod` is always inherited through Java inheritance, not through `@ApiReference`.

## Inheritance and Precedence Rules

To synopsize the preceding discussion, the follow table shows the inheritance rules and order of precedence.

<table>
<thead>
<tr class="header">
<th>Annotation/inheritance</th>
<th>Rule</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>@Api</code></td>
<td>Must be identical for all classes.</td>
</tr>
<tr class="even">
<td><code>@ApiClass</code></td>
<td>Specified for a class to override <code>@Api</code> properties.</td>
</tr>
<tr class="odd">
<td>Java inheritance</td>
<td>Class inherits <code>@Api</code> and <code>@ApiClass</code> of base class.</td>
</tr>
<tr class="even">
<td><code>@ApiReference</code></td>
<td>Class inherits <code>@Api</code> and <code>@ApiClass</code> of referenced class.</td>
</tr>
<tr class="odd">
<td>Using <code>@ApiReference</code> on a class (Java) inheriting from a base class</td>
<td>Class inherits the <code>@Api</code> and <code>@ApiClass</code> of referenced class, <em>not</em> from the base class.</td>
</tr>
</tbody>
</table>

## Common Use Cases for Annotation Inheritance

The following are examples of the typical use cases for inheritance:

For API versioning:

```
@Api(name = "tictactoe", version = "v1")
class TicTacToeV1 {  }
@Api(version = "v2")
class TicTacToeV2 extends TicTacToeV1 {  }
```

For multiclass APIs:

```
@Api(name = "tictactoe", version = "v1")
class TicTacToeBase {}
@ApiClass(resource = "boards")
class TicTacToeBoards extends TicTacToeBase {  }
@ApiClass(resource = "scores")
class TicTacToeScores extends TicTacToeBase {  }
```

For testing different versions of the same API:

```
@Api(name = "tictactoe", version = "v1")
class TicTacToe {
  protected Foo someMethod() {
    // Do something real;
  }

  public Foo getFoo() {  }
}

@Api(version="v1test")
class TicTacToeTest extends TicTacToe {
  protected Foo someMethod() {
    // Stub out real action;
  }
}
```

where `someMethod` might return pre-determined responses, avoid calls with side-effects, skip a network or datastore request, and so forth.

# Query Strings

  

*A wise owl sat up high in a birch,  
and he said as he gazed from his perch.  
I feel so world-weary, think I'll write a query:  
"the meaning of life"  
and go `search()`.  
     - Life, the Universe, and App Engine*

A query string contains Unicode characters. The maximum length of a query string is 2000 characters. All query strings contain at least one field value. It's recommended to write field values in lower case, because searches on atom, text, and HTML fields are case insensitive, and a query string can also contain the boolean operators AND, OR, and NOT, which are recognized by writing them in upper case.

**Note:** A comma can appear in a query string only if it is used to separate the arguments of a function: distance(home, geopoint(35.2, 40.5)) &gt; 100 or it's part of a quoted string.

A query string can take many forms. There are two main ways to construct a query: with and without field names. A *global search* uses a query string that contains only field values:

```
String query = "blue";
String query = "NOT white";
String query = "blue OR red";
String query = "blue guitar";
```

A *field search* uses a query string that contains one or more expressions specifying field names and field values:

```
String query = "model:gibson date < 1965-01-01";
String query = "title:\"Harry Potter\" AND pages<500";
String query = "beverage:wine color:(red OR white) NOT country:france";
```

This document describes how to construct query strings for global and field searches, and how the search logic works in each case.

1.  [Global search](#Java_Global_search)
    -   [One-value queries](#Java_One-value_queries)
    -   [Multi-value queries](#Java_Multi-value_queries)
    -   [Boolean operators](#Java_Boolean_operators)
    -   [Stemming](#Java_Stemming)
    -   [Tokenization](#Java_Tokenization)
2.  [Field search](#Java_Field_search)
    -   [Queries on atom fields](#Java_Queries_on_atom_fields)
    -   [Queries on text and HTML fields](#Java_Queries_on_text_and_HTML_fields)
    -   [Queries on number fields](#Java_Queries_on_number_fields)
    -   [Queries on date fields](#Java_Queries_on_date_fields)
    -   [Queries on geopoint fields](#Java_Queries_on_geopoint_fields)
    -   [Queries on multiple fields](#Java_Queries_on_multiple_fields)
3.  [Mixing global and field searches](#Java_Mixing_global_and_field_searches)

## Global search

Global search offers the ability to search for documents by specifying values that might appear in *any* document field. To perform a global search you write a query string that contains one or more field values. The search algorithm recognizes the type of each value and searches all the document fields that could contain that value.

### One-value queries

Brevity is the soul of wit, and a global search with one value is the epitome of brevity. A search with a query string that contains a single value is handled according to these rules:

If the query string is a word ("red") or a quoted string ("\\red rose\\"), search retrieves all documents in an index that have:

-   a text or HTML field that contains that word or quoted string (matching is case insensitive)
-   an atom field with a value that matches the word or quoted string (matching is case insensitive)

If the query string is a number ("3.14159"), search retrieves all documents that have:

-   a number field with a value equal to the number in the query (a number field with the value 5 will match the query "5" and "5.0")
-   a text or HTML field that contains a token that matches the number as it appears in the query (the text field "he took 5 minutes" will match the query "5" but not "5.0")
-   an atom field that literally matches the number as it appears in the query

If the query string is a date in yyyy-mm-dd form, search retrieves all documents that have:

-   a date field whose value equals that date (leading zeros in the query string are optional, "2012-07-04" and "2012-7-4" are the same date)
-   a text or HTML field that contains a token that literally matches the date as it appears in the query
-   an atom field that literally matches the date as it appears in the query

You can prepend the NOT boolean operator (upper case) to a one word query. The result is a list of documents that do not have any fields that match the query value, according to the same rules. So the query "NOT red" will retrieve all documents that don't have any text or HTML fields that contain "red", or any atom fields with the value "red".

You may have noticed that geopoint fields have not been mentioned. At this time, you cannot specify a raw geopoint value as a string, so geopoints cannot appear in global searches.

### Multi-value queries

You can specify multiple values (separated by spaces) in a global search string. The white space between words, quoted strings, numbers, and dates is treated as an implicit AND operator. The two search strings below are *almost* the same; they differ in how global search treats atom fields, which is explained below:

```
query = "small red"
query = "small AND red"
```

When performing a global search with multiple values, field matching is done independently on each value in the string, and atom field matching is handled differently, in particular:

-   Query values can appear in any order in a text or HTML field
-   Different values can appear in different fields
-   Atom fields are searched only when the query string does not contain any boolean operators (AND, OR, NOT). The entire query string must match an atom field.

Note the third rule dealing with atom fields. The query string "red small" does not contain the boolean AND (even though it's implied), so search will attempt to find matching atom fields. The string "red AND small" does contain a boolean operator, so search will not try to match the query string against atom fields.

The following example shows four documents that were retrieved using the query string "rose bud." Each document has two text fields and one atom field. The comment column explains why each document satisfies the query.

<table>
<thead>
<tr class="header">
<th>Doc ID</th>
<th>Text Field 1</th>
<th>Text Field 2</th>
<th>Atom Field</th>
<th>Comment</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>mighty like a <strong>rose</strong></td>
<td>one <strong>bud</strong> to bind them all</td>
<td>thorn bush</td>
<td>Matching values can appear in different fields</td>
</tr>
<tr class="even">
<td>2</td>
<td>wide like a river</td>
<td>like a <strong>bud</strong> on a <strong>rose</strong></td>
<td>tumble weed</td>
<td>Matching values can appear in any order in the same text or HTML field, with intervening text</td>
</tr>
<tr class="odd">
<td>3</td>
<td>deep like the ocean</td>
<td>the <strong>rose bud</strong> boys</td>
<td>blue bonnet</td>
<td>Matching values can appear in any order in the same text or HTML field</td>
</tr>
<tr class="even">
<td>4</td>
<td>tall like a mountain</td>
<td>the beautiful garden</td>
<td><strong>rose bud</strong></td>
<td>Atom field matches because its value is the same as the entire query string</td>
</tr>
</tbody>
</table>

Note that if you reversed the values in the query and searched for "bud rose" instead, documents 1, 2, and 3 would still be returned, but document 4 would not. To search for an exact character string in atom, text, and HTML fields, quote the string in the query string. A search for "\\rose bud\\" would return only documents 3 and 4 in the example.

### Boolean operators

You can specify a more complex global search by using the boolean operator NOT before a value, and the operators AND and OR between values. Note that these operators must be written in upper case. If they appear within a quoted string they are treated as part of the field value, not as operators. You can use parentheses in a query string make the logic clear.

The boolean operator precedence, from highest to lowest, is: NOT, OR, AND:

```
NOT cat AND dogs OR horses --> (NOT cat) AND (dogs OR horses)
NOT cat OR dogs AND horses --> ((NOT cat) OR dogs) AND horses
```

### Stemming

To search for common variations of a word, like plural forms and verb endings, use the `~` stem operator (the tilde character). This is a prefix operator which must precede a value with no intervening space. The value `~cat` will match "cat" or "cats," and likewise `~dog` matches "dog" or "dogs." The stemming algorithm is not fool-proof. The value `~care` will match "care" and "caring," but not "cares" or "cared." Stemming is only used when searching text and HTML fields.

### Tokenization

When a document is indexed, its fields are [tokenized](https://web.archive.org/web/20160424225934/https://cloud.google.com/appengine/docs/java/search/#Java_Tokenizing_string_fields). Similarly, the values in a query string are also tokenized. This means that what may appear to be a one-value query is actually treated as a multi-value query. For example:

```
"real-time" --> "real time"
"2001-12-15" --> "2001 12 15"
"1.5" --> "1 5"
```

## Field search

A field search looks for values in specific document fields, by field name. A field search query string is composed of one or more expressions that specify a *field name*, a *relational operator*, and a *field value*. The available relational operators depend on the type of the field. The equality operator, represented by either a colon or the equals sign, can be used for all field types. Here are some field query strings for different types of fields:

```
query = "pet = dog"
query = "author = \"Ray Bradbury\""
query = "color:red"
query = "NOT color:red"
query = "price < 500"
query = "birthday>=2011-05-10"
```

Note that the use of whitespace on either side of the relational operator is optional. As with global search query strings, the value of a text, HTML, or atom field can be enclosed in quotes to specify a string, and an expression for a field value can be negated by prepending an uppercase NOT.

### Queries on atom fields

The value of an atom field is a character string. Queries on atom fields are case insensitive. If your query specifies a field value with whitespace or punctuation, be sure to quote the value within the query string. The only valid relational operator for atom fields is the equality operator. The complete contents of an atom field must match the query value; this includes any Unicode combining characters or accented characters in the field. Stemming is not supported for atom fields.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Query String</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>"weather=stormy"</code><br />
<code>"weather: stormy" </code></td>
<td>Either form of the equality operator is valid. Retrieves documents with a weather field that equals "stormy."</td>
</tr>
<tr class="even">
<td><code>"Title: \"Tom&amp;Jerry\""</code><br />
<code>"Couple: \"Fred and Ethel\""</code><br />
<code>"Version = \"1HCP(21.3)\""</code><br />
</td>
<td>If you are searching for an atom field that contains whitespace or special characters, enclose the value in quotes.</td>
</tr>
<tr class="odd">
<td><code>"Color = (red OR blue)"</code><br />
<code>"Color = (\"dark red\" OR \"bright blue\")"</code><br />
</td>
<td>You can use parentheses and the logical operator OR to specify a list of alternate field values.</td>
</tr>
</tbody>
</table>

### Queries on text and HTML fields

The only valid relational operator for text and HTML fields is equality. In this case the operator means "field includes value" not "field equals value." You can use the [stemming operator](#Java_Stemming) to search for variants on a word. You can also use the OR and AND operators to specify complex boolean expressions for the field value. If a boolean operator appears within a quoted string, it is not treated specially, it's just another piece of the character string to be matched. Remember that when searching HTML fields, the text within HTML markup tags is ignored. Queries on text and HTML fields are case insensitive. When these fields are indexed, any Unicode combining characters and accented characters in them are "normalized" to their unaccented equivalents. Combining characters and accents are also normalized in query strings on these fields, so a query can include the accented forms or not, and will match the fields in either case.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Query String</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>"Comment = great"</code><br />
<code>"Comment: great"</code></td>
<td>Either form of the equality operator is valid. Retrieves documents with a comment field that contains at least one occurrence of the word "great" in the Comment field.</td>
</tr>
<tr class="even">
<td><code> "Comment = (great big ball)"</code><br />
<code>"Comment = (great AND big AND ball)" </code></td>
<td>To search for two or more words in a field, in any order, enclose the words in parentheses. This query string retrieves documents with a Comment field that includes all three words in any order with any number of other words between them. The space between words implies a logical AND; the second form makes this explicit.</td>
</tr>
<tr class="odd">
<td><code>"Comment = \"insanely great\""</code></td>
<td>To search for a specific string of text, enclose the string in quotes. This query will retrieve documents whose Comment field contains the phrase "insanely great" (and also "insanely-great" which is tokenized to the same thing).</td>
</tr>
<tr class="even">
<td><code>"pet = ~dog"</code></td>
<td>The stemming operator will match variants of the word "dog" in the pet field.</td>
</tr>
<tr class="odd">
<td><code>"Color = (red OR blue)"</code></td>
<td>To search for a match from a list of alternatives, enclose the list in parentheses with the keyword OR between alternatives. This query retrieves documents whose Color field includes either "red" or "blue" or both.</td>
</tr>
<tr class="even">
<td><code>"weather = ((rain OR snow) AND cold)"</code></td>
<td>You can use the logical operators OR and AND, along with parentheses, to specify a more complex field value.</td>
</tr>
<tr class="odd">
<td><code>"weather = \"rain OR shine\""</code></td>
<td>Because the logical OR is embedded in a quoted string, it is not treated as a relational operator. This query string retrieves documents with a weather field that contains the string "rain or shine"</td>
</tr>
</tbody>
</table>

### Queries on number fields

A number field value can be written as an integer, a decimal, or an exponential. The valid relational operators for number fields are the equality operators, along with the less than/greater than operators (&lt;, &lt;=, &gt;, &gt;=). Note that there is no inequality (!=) operator. Here are some example query strings for number fields:

```
"quantity = 10000"
"size: 4"
"price < 9.99"
"theta > 1.5E-2"
```

### Queries on date fields

A date field value must be written in yyyy-mm-dd form. Leading zeros are optional for one-digit months and days. The valid relational operators for date fields are the equality operators, along with the less than/greater than operators (&lt;, &lt;=, &gt;, &gt;=). Note that there is no inequality operator. You can prepend the NOT operator to an expression to negate it. Here are some example query strings for date fields:

```
"start_date: 2012-05-20"
"end_date: 2013-5-1"
"birthday >= 2000-12-31"
"NOT birthday = 2000-12-25"
```

### Queries on geopoint fields

There are no relational operators that work with geopoint fields, so geopoint fields cannot be named directly in a query string. The Search API provides two special functions that can be used for queries involving geopoint fields:

###### `geopoint(lat,long)`

Defines a geopoint given a latitude and longitude.

###### `distance(point1, point2)`

Computes the distance in meters between two geopoints. Each point can be specified by using the name of a geopoint field or an invocation of the geopoint function. Note that you cannot provide two field names as arguments to this function. At least one argument must be a constant.

These functions can be used to construct queries that retrieve locations relative to a constant position. The following examples assume that the index contains documents with geopoint fields named "survey\_marker" and "home."

<table>
<thead>
<tr class="header">
<th>Query String</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>"distance(survey_marker, geopoint(35.2, 40.5)) &lt; 100"</code></td>
<td>Search for markers less than 100 meters from a given geopoint.</td>
</tr>
<tr class="even">
<td><code>"distance(home, geopoint(35.2, 40.5)) &gt; 100"</code></td>
<td>Search for homes more than 100 meters from a given geopoint.</td>
</tr>
</tbody>
</table>

Applications that use geolocation typically receive information from the browser. If the user allows, location can be inferred from their IP address, or they can enter a postal code. Location can also come from other APIs like the [Google Maps Geolocation API](https://web.archive.org/web/20160424225934/https://developers.google.com/maps/documentation/business/geolocation).

### Queries on multiple fields

You can combine multiple field query expressions in one query by listing them in sequence separated by whitespace. This puts an implied AND between each expression, so all of them must be satisfied to retrieve a document. You can explicitly add AND and OR operators between expressions, and use parentheses to make the logic clear.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Query String</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>"product=piano manufacturer=steinway"</code><br />
<code>"product=piano AND manufacturer=steinway"</code></td>
<td>These queries retrieve all Steinway pianos. The space between the terms implies a logical AND; the second form makes this explicit.</td>
</tr>
<tr class="even">
<td><code>"product=piano AND NOT manufacturer=steinway"</code></td>
<td>Retrieves all non-Steinway pianos.</td>
</tr>
<tr class="odd">
<td><code>"product=piano AND price&lt;2000"</code></td>
<td>This query retrieves inexpensive pianos.</td>
</tr>
</tbody>
</table>

## Mixing global and field searches

A query string can contain any number of global search expressions and field search expressions. Spaces between expressions are treated as AND. You can also use OR and AND explicity, along with parentheses. Each expression will be handled according to the rules associated with that kind of term.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Query String</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>"keyboard great price&lt;5000"</code><br />
<code>"keyboard AND great AND price&lt;5000"</code></td>
<td>Retrieves documents where the words "great" and "keyboard" appear in any text, HTML, or atom fields, and there is a price field less than 5000. The AND is implied, the second form is equivalent.</td>
</tr>
<tr class="even">
<td><code>"keyboard OR product=piano"</code></td>
<td>Retrieves documents with a product field that contains piano, or documents with any text, HTML, or atom field that contains keyboard.</td>
</tr>
</tbody>
</table>

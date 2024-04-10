# Documents and Indexes

  

[Python](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/python/search/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/go/search/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425050508im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425050508/http://stackoverflow.com/questions/tagged/google-app-engine+search)

[](https://web.archive.org/web/20160425050508/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bsearch)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425050508/http://stackoverflow.com/questions/tagged/google-app-engine+search?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

The Search API provides a model for indexing documents that contain structured data. You can search an index, and organize and present search results. The API supports full text matching on string fields. Documents and indexes are saved in a separate persistent store optimized for search operations. The Search API can index any number of documents. The App Engine Datastore may be more appropriate for applications that need to retrieve very large result sets.

1.  [Overview](#Java_Overview)
2.  [Documents and fields](#Java_Documents_and_fields)
3.  [Creating a document](#Java_Creating_a_document)
4.  [Working with an index](#Java_Working_with_an_index)
    -   [Putting documents in an index](#Java_Putting_documents_in_an_index)
    -   [Retrieving documents by doc\_id](#Java_Retrieving_documents_by_doc_ids)
    -   [Searching for documents by their contents](#Java_Searching_for_documents_by_their_contents)
    -   [Deleting documents from an index](#Java_Deleting_documents_from_an_index)
    -   [Eventual consistency](#Java_Consistency)
    -   [Determining the size of an index](#Java_Determining_the_size_of_an_index)
5.  [Index schemas](#Java_Index_schemas)
6.  [Viewing indexes in the Google Cloud Platform Console](#Java_Viewing_indexes_in_the_Dev_Console)
7.  [Search API quotas](#Java_Search_API_quotas)
8.  [Search API pricing](#Java_Search_API_pricing)

## Overview

The Search API is based on four main concepts: documents, indexes, queries, and results.

### Documents

A document is an object with a unique ID and a list of fields containing user data. Each field has a name and a type. There are several types of fields, identified by the kinds of values they contain:

-   Atom Field - an indivisible character string
-   Text Field - a plain text string that can be searched word by word
-   HTML Field - a string that contains HTML markup tags, only the text outside the markup tags can be searched
-   Number Field - a floating point number
-   Date Field - a date object
-   Geopoint Field - a data object with latitude and longitude coordinates

The maximum size of a document is 1 MB.

### Indexes

An index stores documents for retrieval. You can retrieve a single document by its ID, a range of documents with consecutive IDs, or all the documents in an index. You can also search an index to retrieve documents that satisfy given criteria on fields and their values, specified as a query string. You can manage groups of documents by putting them into separate indexes.

There is no limit to the number of documents in an index or the number of indexes you can use. The total size of all the documents in a single index is limited to 10GB by default but may be increased to up to 200GB by [submitting a request](https://web.archive.org/web/20160425050508/https://support.google.com/cloud/contact/cloud_search_api_index_size_increase_request_form).

### Queries

To search an index, you construct a query, which has a query string and possibly some additional options. A query string specifies conditions for the values of one or more document fields. When you search an index you get back only those documents in the index with fields that satisfy the query.

The simplest query, sometimes called a "global search" is a string that contains only field values. This search uses a string that searches for documents that contain the words "rose" and "water":

```
index.search("rose water");
```

This one searches for documents with date fields that contain the date July 4, 1776, or text fields that include the string "1776-07-04":

```
index.search("1776-07-04");
```

A query string can also be more specific. It can contain one or more terms, each naming a field and a constraint on the field's value. The exact form of a term depends on the type of the field. For instance, assuming there is a text field called "product", and a number field called "price", here's a query string with two terms:

```
// search for documents with pianos that cost less than $5000
index.search("product = piano AND price < 5000");
```

Query options, as the name implies, are not required. They enable a variety of features:

-   Control how many documents are returned in the search results.
-   Specify what document fields to include in the results. The default is to include all the fields from the original document. You can specify that the results only include a subset of fields (the original document is not affected).
-   Sort the results.
-   Create "computed fields" for documents using [`FieldExpressions`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/search/options#Java_Writing_expressions) and abridged text fields using [snippets](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/search/options#Java_Snippets).
-   Support paging through the search results by returning only a portion of the matched documents on each query (using offsets and cursors)

### Search results

A call to `search()` can only return a limited number of matching documents. Your search may find more documents than can be returned in a single call. Each search call returns an instance of the [`Results`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Results) class, which contains information about how many documents were found and how many were returned, along with the list of returned documents. You can repeat the same search, using [cursors](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/search/results#Java_Using_cursors) or [offsets](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/search/results#Java_Using_offsets) to retrieve the complete set of matching documents.

### Additional training material

In addition to this documentation, you can read the [two-part training class](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/training/fts_intro/) on the Search API at the Google Developer's Academy. (Although the class uses the Python API, you may find the additional discussion of the Search concepts useful.)

## Documents and fields

The [Document](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Document) class represents documents. Each document has a *document identifier* and a list of *fields*.

### Document identifier

Every document in an index must have a unique document identifier, or `doc_id`. The identifier can be used to retrieve a document from an index without performing a search. By default, the Search API automatically generates a `doc_id` when a document is created. You can also specify the `doc_id` yourself when you create a document. A `doc_id` must contain only visible, printable ASCII characters (ASCII codes 33 through 126 inclusive) and be no longer than 500 characters. A document identifier cannot begin with an exclamation point ('!'), and it can't begin and end with double underscores ("\_\_").

While it is convenient to create readable, meaningful unique document identifiers, you cannot include the `doc_id` in a search. Consider this scenario: You have an index with documents that represent parts, using the part's serial number as the `doc_id`. It will be very efficient to retrieve the document for any single part, but it will be impossible to search for a range of serial numbers along with other field values, such as purchase date. Storing the serial number in an atom field solves the problem.

### Document fields

A document contains fields that have a *name*, a *type*, and a single *value* of that type. Two or more fields can have the same name, but different types. For instance, you can define two fields with the name "age": one with a text type (the value "twenty-two"), the other with a number type (value 22).

#### Field names

Field names are case sensitive and can only contain ASCII characters. They must start with a letter and can contain letters, digits, or underscore. A field name cannot be longer than 500 characters.

#### Multi-valued fields

A field can contain only one value, which must match the field's type. Field names do not have to be unique. A document can have multiple fields with the same name and same type, which is a way to represent a field with multiple values. (However, date and number fields with the same name can't be repeated.) A document can also contain multiple fields with the same name and *different* field types.

#### Field types

There are three kinds of fields that store `java.lang.String` character strings; collectively we refer to them as *string fields*:

-   Text Field: A string with maximum length 1024\*\*2 characters.
-   HTML Field: An HTML-formatted string with maximum length 1024\*\*2 characters.
-   Atom Field: A string with maximum length 500 characters.

There are also three field types that store non-textual data:

-   Number Field: A double precision floating point value between -2,147,483,647 and 2,147,483,647.
-   Date Field: A `java.util.Date`.
-   Geopoint Field: A point on earth described by latitude and longitude coordinates

The field types are specified using the [`Field.FieldType`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Field.FieldType) enums `TEXT`, `HTML`, `ATOM`, `NUMBER`, `DATE`, and `GEO_POINT`.

### Special treatment of string and date fields

When a document with date, text, or HTML fields is added to an index, some special handling occurs. It's helpful to understand what's going on "under the hood" in order to use the Search API effectively.

#### Tokenizing string fields

When an HTML or text field is indexed, its contents are *tokenized*. The string is split into tokens wherever whitespace or special characters (punctuation marks, hash sign, etc.) appear. The index will include an entry for each token. This enables you to search for keywords and phrases comprising only part of a field's value. For instance, a search for "dark" will match a document with a text field containing the string "it was a dark and stormy night", and a search for "time" will match a document with a text field containing the string "this is a real-time system".

In HTML fields, text within markup tags is not tokenized, so a document with an HTML field containing "it was a &lt;strong&gt;dark&lt;/strong&gt; night" will match a search for "night", but not for "strong". If you want to be able to search markup text, store it in a text field.

Atom fields are not tokenized. A document with an atom field that has the value "bad weather" will only match a search for the entire string "bad weather". It will not match a search for "bad" or "weather" alone.

##### Tokenizing Rules

-   The underscore (\_) and ampersand (&) characters do not break words into tokens.

-   These whitespace characters always break words into tokens: space, carriage return, line feed, horizontal tab, vertical tab, form feed, and NULL.

-   These characters are treated as punctuation, and will break words into tokens:

<table>
<tbody>
<tr class="odd">
<td>!</td>
<td>"</td>
<td>%</td>
<td>(</td>
<td>)</td>
</tr>
<tr class="even">
<td>*</td>
<td>,</td>
<td>-</td>
<td>|</td>
<td>/</td>
</tr>
<tr class="odd">
<td>[</td>
<td>]</td>
<td>]</td>
<td>^</td>
<td>`</td>
</tr>
<tr class="even">
<td>:</td>
<td>=</td>
<td>&gt;</td>
<td>?</td>
<td>@</td>
</tr>
<tr class="odd">
<td>{</td>
<td>}</td>
<td>~</td>
<td>$</td>
<td></td>
</tr>
</tbody>
</table>

-   The characters in the following table *usually* break words into tokens, but they can be handled differently depending on the context in which they appear:

<table>
<thead>
<tr class="header">
<th>Character</th>
<th>Rule</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>&lt;</td>
<td>In an HTML field the "less than" sign indicates the start of an HTML tag which is ignored.</td>
</tr>
<tr class="even">
<td>+</td>
<td>A string of one or more "plus" signs is treated as a part of the word if it appears at the end of the word (C++).</td>
</tr>
<tr class="odd">
<td>#</td>
<td>The "hash" sign is treated as a part of the word if it is preceded by a, b, c, d, e, f, g, j, or x (a# - g# are musical notes; j# and x# are programming language, c# is both.) If a term is <em>preceded</em> by '#' (#google), it is treated as a hashtag and the hash becomes part of the word.</td>
</tr>
<tr class="even">
<td>'</td>
<td>Apostrophe is a letter if it precedes the letter "s" followed by a word-break, as in "John's hat".</td>
</tr>
<tr class="odd">
<td>.</td>
<td>If a decimal point appears between digits, this is part of a number (i.e., the decimal-separator). This can also be part of a word if used in an acronym (A.B.C).</td>
</tr>
<tr class="even">
<td>-</td>
<td>The dash is part of a word if used in an acronym (I-B-M).</td>
</tr>
</tbody>
</table>

-   All other 7-bit characters other than letters and digits ('A-Z', 'a-z', '0-9') are handled as punctuation and break words into tokens.

-   Everything else is parsed as a UTF-8 character.

**Note:** Non-western languages, like Japanese and Chinese, use other tokenization rules.

##### Acronyms

Tokenization uses special rules to recognize acronyms (strings like "I.B.M.", "a-b-c", or "C I A"). An acronym is a string of single alphabetic characters, with the same separator character between all of them. The valid separators are the period, dash, or any number of spaces. The separator character is removed from the string when an acronym is tokenized. So the example strings mentioned above become the tokens "ibm", "abc", and "cia". The original text remains in the document field.

When dealing with acronyms, note that:

-   An acronym cannot contain more than 21 letters. A valid acronym string with more than 21 letters will be broken into a series of acronyms, each 21 letters or less.
-   If the letters in an acronym are separated by spaces, all the letters must be the same case. Acronyms constructed with period and dash can use mixed case letters.
-   When searching for an acronym, you can enter the canonical form of the acronym (the string without any separators), or the acronym punctuated with either the dash or the dot (but not both) between its letters. So the text "I.B.M" could be retrieved with any of the search terms "I-B-M", "I.B.M", or "IBM".

#### Date field accuracy

When you create a date field in a document you set its value to a `java.util.Date`. For the purpose of indexing and searching the date field, any time component is ignored and the date is converted to the number of days since 1/1/1970 UTC. This means that even though a date field can contain a precise time value a date query can only specify a date field value in the form yyyy-mm-dd. This also means the sorted order of date fields with the same date is not well-defined.

### Other document properties

The *rank* of a document is a positive integer which determines the default ordering of documents returned from a search. By default, the rank is set at the time the document is created to the number of seconds since January 1, 2011. You can set the rank explicitly when you create a document. (It's a bad idea to assign the same rank to many documents, and you should never give more than 10,000 documents the same rank.) If you specify [sort options](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/search/options#Java_SortOptions), you can use the rank as a sort key. Note that when rank is used in a [sort expression](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/search/options#Java_Sorting_on_multiple_keys) or [field expression](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/search/options#Java_FieldExpressions) it is referenced as `_rank`.

The locale property specifies the language in which the fields are encoded.

See the [`Document`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Document) class reference page for more details about these attributes.

### Linking from a document to other resources

You can use a document's `doc_id` and other fields as links to other resources in your application. For example, if you use [Blobstore](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/blobstore/) you can associate the document with a specific blob by setting the `doc_id` or the value of an Atom field to the BlobKey of the data.

## Creating a document

To create a document, request a new builder using the [`Document.newBuilder()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Document#newBuilder()) method. Once the application has access to a builder, it can specify an optional document identifier and add fields.

Fields, like documents, are created using a builder. The [`Field.newBuilder()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Field#newBuilder()) method returns a field builder that allows you to specify a field's name and value. The field type is automatically specified by choosing a specific set method. For example, to indicate that a field holds plain text, call [`setText()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Field.Builder#setText(java.lang.String)). The following code builds a document with fields representing a guestbook greeting.

```
import com.google.appengine.api.search.Document;
import com.google.appengine.api.search.Field;
import com.google.appengine.api.users.User;
import com.google.appengine.api.users.UserServiceFactory;

...

User currentUser = UserServiceFactory.getUserService().getCurrentUser();
String myDocId = "PA6-5000";
Document doc = Document.newBuilder()
    .setId(myDocId) // Setting the document identifer is optional. If omitted, the search service will create an identifier.
    .addField(Field.newBuilder().setName("content").setText("the rain in spain"))
    .addField(Field.newBuilder().setName("email")
        .setText(currentUser.getEmail()))
    .addField(Field.newBuilder().setName("domain")
        .setAtom(currentUser.getAuthDomain()))
    .addField(Field.newBuilder().setName("published").setDate(new Date()))
    .build();
```
**Note:** When you create a document you must specify all of its attributes using the [`Document.Builder`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Document.Builder) class method. You cannot add, remove, or delete fields, nor change the identifier or any other attribute once the document has been created. Date and geopoint fields must be assigned a non-null value. Atom, text, HTML, and number fields can be empty.

To access fields within the document, use [`getOnlyField()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Document#getOnlyField(java.lang.String)):

```
Document document = ...

String coverLetter = document.getOnlyField("coverLetter").getText();
String resume = document.getOnlyField("resume").getHTML();
String fullName = document.getOnlyField("fullName").getAtom();
Date submissionDate = document.getOnlyField("submissionDate").getDate();
```

## Working with an index

### Putting documents in an index

When you put a document into an index, the document is copied to persistent storage and each of its fields is indexed according to its name, type, and the `doc_id`.

The following code example shows how to access an Index and put a document into it. The steps are:

-   Create an [`IndexSpec`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/IndexSpec)
-   Create a [`SearchService`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/SearchService)
-   Call [`SearchService.getIndex()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/SearchService#getIndex(com.google.appengine.api.search.IndexSpec)) to create an instance of Index.
-   Call [`Index.put()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#put(com.google.appengine.api.search.Document...)) to add the document to the index.

```
import com.google.appengine.api.search.Index;
import com.google.appengine.api.search.IndexSpec;
import com.google.appengine.api.search.SearchServiceFactory;
import com.google.appengine.api.search.PutException;
import com.google.appengine.api.search.StatusCode;

public void IndexADocument(String indexName, Document document) {
    IndexSpec indexSpec = IndexSpec.newBuilder().setName(indexName).build(); 
    Index index = SearchServiceFactory.getSearchService().getIndex(indexSpec);
    
    try {
        index.put(document);
    } catch (PutException e) {
        if (StatusCode.TRANSIENT_ERROR.equals(e.getOperationResult().getCode())) {
            // retry putting the document
        }
    }
}
```
You can pass up to 200 documents at a time to the `put()` method. Batching puts is more efficient than adding documents one at a time.

When you put a document into an index and the index already contains a document with the same `doc_id`, the new document replaces the old one. No warning is given. You can call [`Index.get(id)`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#get(java.lang.String)) before creating or adding a document to an index to check whether a specific `doc_id` already exists.

Note that creating an instance of the `Index` class does not guarantee that a persistent index actually exists. A persistent index is created the first time you add a document to it with the `put` method. If you want to check whether or not an index actually exists before you start to use it, use the [`SearchService.getIndexes()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/SearchService#getIndexes(com.google.appengine.api.search.GetIndexesRequest)) method.

#### Updating documents

A document cannot be changed once you've added it to an index. You can't add or remove fields, or change a field's value. However, you can replace the document with a new document that has the same `doc_id`.

### Retrieving documents by doc\_id

There are two ways to retrieve documents from an index using document identifiers:

-   Use [`Index.get()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#get(java.lang.String)) to fetch a single document by its `doc_id`.
-   Use [`Index.getRange()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#getRange(com.google.appengine.api.search.GetRequest)) to retrieve a group of consecutive documents ordered by `doc_id`.

Each call is demonstrated in the example below.

```
IndexSpec indexSpec = IndexSpec.newBuilder().setName(indexName).build(); 
Index index = SearchServiceFactory.getSearchService().getIndex(indexSpec);
    
// Fetch a single document by its  doc_id
Document doc = index.get("AZ125");

// Fetch a range of documents by their doc_ids
GetResponse<Document> docs = index.getRange(GetRequest.newBuilder().setStartId("AZ125").setLimit(100).build());
```

### Searching for documents by their contents

To retrieve documents from an index, you construct a query string and call `Index.search()`. The query string can be passed directly as the argument, or you can include the string in a [Query](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Query) object which is passed as the argument. By default, `search()` returns matching documents sorted in order of decreasing rank. To control how many documents are returned, how they are sorted, or add computed fields to the results, you need to use a `Query` object, which contains a query string and can also specify other search and sorting options.

```
import com.google.appengine.api.search.Results;
import com.google.appengine.api.search.ScoredDocument;
import com.google.appengine.api.search.SearchException;

...

try {
    String queryString = "product: piano AND price &lt; 5000";
    Results<ScoredDocument> results = getIndex().search(queryString);

    // Iterate over the documents in the results
    for (ScoredDocument document : results) {
        // handle results
    }
} catch (SearchException e) {
    if (StatusCode.TRANSIENT_ERROR.equals(e.getOperationResult().getCode())) {
        // retry
    }
}
```

### Deleting documents from an index

You can delete documents in an index by specifying the `doc_id` of one or more documents you wish to delete to the [`delete()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#delete(java.lang.Iterable)) method. To get a range of document ids in an index, invoke the [`setReturningIdsOnly()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/GetRequest.Builder#setReturningIdsOnly(boolean)) method of the [`GetRequest.Builder`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/GetRequest.Builder), which is then given to the [`getRange()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#getRange(com.google.appengine.api.search.GetRequest)) method. When you invoke this method, the API returns document objects populated only with the `doc_id`. You can then delete the documents by passing those document identifiers to the `delete()` method:

```
import com.google.appengine.api.search.Document;
import com.google.appengine.api.search.GetRequest;
import com.google.appengine.api.search.GetResponse;
...
try {
    // looping because getRange by default returns up to 100 documents at a time
    while (true) {
        List<String> docIds = new ArrayList<String>();
        // Return a set of doc_ids.
        GetRequest request = GetRequest.newBuilder().setReturningIdsOnly(true).build();
        GetResponse<Document> response = getIndex().getRange(request);
        if (response.getResults().isEmpty()) {
            break;
        }
        for (Document doc : response) {
            docIds.add(doc.getId());
        }
        getIndex().delete(docIds);
    }
} catch (RuntimeException e) {
    LOG.log(Level.SEVERE, "Failed to delete documents", e);
}
```
You can pass up to 200 documents at a time to the `delete()` method. Batching deletes is more efficient than handling them one at a time.

### Eventual consistency

When you put, update, or delete a document in an index, the change propagates across multiple data centers. This usually happens quickly, but the time it takes is variable. The Search API guarantees [eventual consistency](https://web.archive.org/web/20160425050508/http://en.wikipedia.org/wiki/Eventual_consistency). This means that in some cases if you perform a search or retrieve one or more documents by id, the results may not reflect the most recent change.

### Determining the size of an index

An index stores documents for retrieval. You can retrieve a single document by its ID, a range of documents with consecutive IDs, or all the documents in an index. You can also search an index to retrieve documents that satisfy given criteria on fields and their values, specified as a query string. You can manage groups of documents by putting them into separate indexes. There is no limit to the number of documents in an index or the number of indexes you can use. The total size of all the documents in a single index is limited to 10GB by default but may be increased to up to 200GB by [submitting a request](https://web.archive.org/web/20160425050508/https://support.google.com/cloud/contact/cloud_search_api_index_size_increase_request_form). (The method [Index.getStorageLimit()](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#getStorageLimit()) returns the maximum allowable size of an index.)

The method [`Index.getStorageUsage()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/Index#getStorageUsage()) is an estimate of the amount of storage space used by an index. This number is an estimate because the index monitoring system does not run continuously; the actual usage is computed periodically. The `storage_usage` is adjusted between sampling points by accounting for document additions, but not deletions.

## Index schemas

Every index has a schema that shows all the field names and field types that appear in the documents it contains. You cannot define a schema yourself. Schemas are maintained dynamically; they are updated as documents are added to an index. A simple schema might look like this, in JSON-like form:

```
{'comment': ['TEXT'], 'date': ['DATE'], 'author': ['TEXT'], 'count': ['NUMBER']}
```

Each key in the dictionary is the name of a document field. The key's value is a list of the field types used with that field name. If you have used the same field name with different field types the schema will list more than one field type for a field name, like this:

```
{'ambiguous-integer': ['TEXT', 'NUMBER', 'ATOM']}
```

Once a field appears in a schema it can never be removed. There is no way to delete a field, even if the index no longer contains any documents with that particular field name.

You can view the schemas for your indexes like this:
```
import com.google.appengine.api.search.Field.FieldType;
import com.google.appengine.api.search.Index;
import com.google.appengine.api.search.GetIndexesRequest;
import com.google.appengine.api.search.GetIndexesResponse;
import com.google.appengine.api.search.Schema;

GetResponse<Index> response = SearchServiceFactory.getSearchService().getIndexes(
    GetIndexesRequest.newBuilder().setSchemaFetched(true).build());

// List out elements of each Schema
for (Index index : response) {
    Schema schema = index.getSchema();
    for (String fieldName : schema.getFieldNames()) {
        List<FieldType> typesForField = schema.getFieldTypes(fieldName);
    }
}
```
Note that a call to [`GetIndexes()`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/?com/google/appengine/api/search/GetIndexesRequest.html) cannot return more than 1000 indexes. To retrieve more indexes, call the method repeatedly, using `setStartIndexName()` along with [`GetIndexesRequest.Builder`](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/search/GetIndexesRequest.Builder).

A schema does not define a "class" in the object-programming sense. As far as the Search API is concerned, every document is unique and indexes can contain different kinds of documents. If you want to treat collections of objects with the same list of fields as instances of a class, that's an abstraction you must enforce in your code. For instance, you could insure that all documents with the same set of fields are kept in their own index. The index schema could be seen as the class definition, and each document in the index would be an instance of the class.

## Viewing indexes in the Google Cloud Platform Console

In the Cloud Platform Console, you can <a href="https://web.archive.org/web/20160425050508/https://console.cloud.google.com//project/_/appengine/search" data-track-metadata-end-goal="viewIndexes" data-track-metadata-position="body" data-track-name="consoleLink" data-track-type="tasks">view information about your application's indexes and the documents they contain</a>. Clicking an index name displays the documents that index contains. You'll see all the defined schema fields for the index; for each document with a field of that name, you'll see the field's value. You can also issue queries on the index data directly from the console.

## Search API quotas

The Search API has several free quotas:

<table>
<thead>
<tr class="header">
<th>Resource or API call</th>
<th>Free Quota</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Total storage (documents and indexes)</td>
<td>0.25 GB</td>
</tr>
<tr class="even">
<td>Queries</td>
<td>1000 queries per day</td>
</tr>
<tr class="odd">
<td>Adding documents to indexes</td>
<td>0.01 GB per day</td>
</tr>
</tbody>
</table>

The Search API imposes these limits to ensure the reliability of the service. These apply to both free and paid apps:

<table>
<thead>
<tr class="header">
<th>Resource</th>
<th>Safety Quota</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum query usage</td>
<td>100 aggregated minutes of query execution time per minute</td>
</tr>
<tr class="even">
<td>Maximum documents added or deleted</td>
<td>15,000 per minute</td>
</tr>
<tr class="odd">
<td>Maximum size per index (unlimited number of indexes allowed)</td>
<td>10 GB</td>
</tr>
</tbody>
</table>

API usage is counted in different ways depending on the type of call:

-   `Index.search()`: Each API call counts as one query; execution time is equivalent to the latency of the call.
-   `Index.put()`: When you add documents to indexes the size of each document and the number of documents counts towards the indexing quota.
-   All other Search API calls are counted based on the number of operations they involve:
    -   `SearchService.getIndexes()`: 1 operation is counted for each index actually returned, or 1 operation if nothing is returned.
    -   `Index.get()` and `Index.getRange()`: 1 operation counted for each document actually returned, or 1 operation if nothing is returned.
    -   `Index.delete()`: 1 operation counted for each document in the request, or 1 operation if the request is empty.

The quota on query throughput is imposed so that a single user cannot monopolize the search service. Because queries can execute simultaneously, each application is allowed to run queries that consume up to 100 minutes of execution time per one minute of clock time. If you are running many short queries, you probably will not reach this limit. Once you exceed the quota, subsequent queries will fail until the next time slice, when your quota is restored. The quota is not strictly imposed in one minute slices; a variation of the [leaky bucket](https://web.archive.org/web/20160425050508/http://en.wikipedia.org/wiki/Leaky_bucket) algorithm is used to control search bandwidth in five second increments.

More information on quotas can be found on the [Quotas](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/docs/quotas) page. When an app tries to exceed these amounts, an insufficient quota error is returned.

Note that although these limits are enforced by the minute, the console displays the daily totals for each. Customers with [Silver, Gold, or Platinum support](https://web.archive.org/web/20160425050508/https://cloud.google.com/support/packages) can request higher throughput limits by contacting their support representative.

## Search API pricing

If you enable billing for your app you will be charged for additional usage beyond free quotas. The following charges apply to billed apps:

<table>
<thead>
<tr class="header">
<th>Resource</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Total storage (documents and indexes)</td>
<td>$0.18 per GB per month</td>
</tr>
<tr class="even">
<td>Queries</td>
<td>$ 0.50 per 10K queries</td>
</tr>
<tr class="odd">
<td>Indexing searchable documents</td>
<td>$2.00 per GB</td>
</tr>
</tbody>
</table>

Additional information on pricing is on the [Pricing](https://web.archive.org/web/20160425050508/https://cloud.google.com/appengine/pricing) page.

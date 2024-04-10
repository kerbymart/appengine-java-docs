# Using the Java Development Server

  

The Java development server runs on your local machine and emulates most of the Search API's capabilities. However, a few features are not currently available on the server. For the moment, you should not attempt to use the following features when you run on the development server:

### Stemming

Stemming of field values (for example "~cat") is not implemented.

### Tokenization

String fields in Asian languages are not tokenized on the development server.

### Scoring

The match scoring mechanism is not implemented.

### Diacriticals

Diacritical marks (such as accent marks) cannot be used in atom, text, and HTML fields.

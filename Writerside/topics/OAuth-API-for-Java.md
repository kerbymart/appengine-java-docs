# OAuth API for Java

  

[Python](https://web.archive.org/web/20160425100012/https://cloud.google.com/appengine/docs/python/oauth/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color:lightgray" title="This page is not available in the PHP runtime">PHP</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425100012/https://cloud.google.com/appengine/docs/go/oauth/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425100012im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425100012/http://stackoverflow.com/questions/tagged/google-app-engine+oauth)

[](https://web.archive.org/web/20160425100012/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Boauth)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425100012/http://stackoverflow.com/questions/tagged/google-app-engine+oauth?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

The App Engine OAuth API uses the OAuth protocol and provides a way for your app to authenticate users who are requesting access without asking for their credentials (username and password).

1.  [OAuth 2.0](#oauth2)
    1.  [Generating an access token](#oa2-access-token)
    2.  [Using the API](#oa2-api)

## OAuth 2.0

OAuth 2.0 access tokens supplied by the [Google Sign-In](https://web.archive.org/web/20160425100012/https://developers.google.com/identity/sign-in/) library and low-level [OAuth 2.0 endpoints](https://web.archive.org/web/20160425100012/https://developers.google.com/identity/protocols/OAuth2) can be used to authenticate clients with the OAuth API and retrieve user identity information.

### Generating an access token

Follow the steps outlined at [Using OAuth 2.0 to Access Google APIs](https://web.archive.org/web/20160425100012/https://developers.google.com/identity/protocols/OAuth2).

**Note:** Be sure you include the email scope (`https://www.googleapis.com/auth/userinfo.email`) in step \#2. This scope is required because the App Engine OAuth API accesses the user's email address.

The client should send the resulting OAuth 2.0 access token in the `Authorization: Bearer` HTTP Request Header on every request to your AppEngine app.

You can also obtain an access token using one of the [Google Sign-in](https://web.archive.org/web/20160425100012/https://developers.google.com/identity/) client libraries for [Android](https://web.archive.org/web/20160425100012/https://developers.google.com/identity/toolkit/android/), [iOS](https://web.archive.org/web/20160425100012/https://developers.google.com/identity/toolkit/ios/), or the [web](https://web.archive.org/web/20160425100012/https://developers.google.com/identity/toolkit/web/).

### Using the API

When a client sends a request to your app, the authorization header of the request includes an OAuth access token that has one or more scopes associated with it, indicating what APIs the client can access. Your app can retreive information about the user who granted the access token by running this code:

```
OAuthService oauth = OAuthServiceFactory.getOAuthService();
String scope = "https://www.googleapis.com/auth/userinfo.email";
Set<String> allowedClients = new HashSet<>();
allowedClients.add("407408718192.apps.googleusercontent.com"); // list your client ids here

try {
  User user = oauth.getCurrentUser(scope);
  String tokenAudience = oauth.getClientId(scope);
  if (!allowedClients.contains(tokenAudience)) {
    throw new OAuthRequestException("audience of token '" + tokenAudience
        + "' is not in allowed list " + allowedClients);
  }
  // proceed with authenticated user
  // ...
} catch (OAuthRequestException ex) {
  // handle auth error
  // ...
} catch (OAuthServiceFailureException ex) {
  // optionally, handle an oauth service failure
  // ...
}
```

`getCurrentUser` returns an object representing the user associated with the request. If the access token is invalid, the method returns an error.

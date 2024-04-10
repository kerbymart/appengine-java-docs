# The Task Queue REST API
## About Authorization

### Authorizing requests

Every request your application sends to the Google App Engine Task Queue must include an authorization token. The token also identifies your application to Google.

Your application must use [OAuth 2.0](https://web.archive.org/web/20160424230857/https://developers.google.com/identity/protocols/OAuth2) to authorize requests. No other authorization protocols are supported. If your application uses [Google Sign-In](https://web.archive.org/web/20160424230857/https://developers.google.com/identity/#google-sign-in), some aspects of authorization are handled for you.

## Authorizing requests with OAuth 2.0

All requests to the Google App Engine Task Queue must be authorized by an authenticated user.

The details of the authorization process, or "flow," for OAuth 2.0 vary somewhat depending on what kind of application you're writing. The following general process applies to all application types:

1.  When you create your application, you register it using the [Google Cloud Platform Console](https://web.archive.org/web/20160424230857/https://console.cloud.google.com/). Google then provides information you'll need later, such as a client ID and a client secret.
2.  Activate the Google App Engine Task Queue in the Google Cloud Platform Console. (If the API isn't listed in the Cloud Platform Console, then skip this step.)
3.  When your application needs access to user data, it asks Google for a particular **scope** of access.
4.  Google displays a **consent screen** to the user, asking them to authorize your application to request some of their data.
5.  If the user approves, then Google gives your application a short-lived **access token**.
6.  Your application requests user data, attaching the access token to the request.
7.  If Google determines that your request and the token are valid, it returns the requested data.

Some flows include additional steps, such as using **refresh tokens** to acquire new access tokens. For detailed information about flows for various types of applications, see Google's [OAuth 2.0 documentation](https://web.archive.org/web/20160424230857/https://developers.google.com/identity/protocols/OAuth2).

Here's the OAuth 2.0 scope information for the Google App Engine Task Queue:

| Scope | Meaning |
| --- | --- |
| `https://www.googleapis.com/auth/taskqueue` | Read/write access. |
| `https://www.googleapis.com/auth/taskqueue.consumer` | Read-only access. |
| `https://www.googleapis.com/auth/cloud-taskqueue` | Read/write access for  
[Compute Engine Service Accounts](https://web.archive.org/web/20160424230857/https://cloud.google.com/compute/docs/authentication#applications). |
| `https://www.googleapis.com/auth/cloud-taskqueue.consumer` | Read-only access for  
[Compute Engine Service Accounts.](https://web.archive.org/web/20160424230857/https://cloud.google.com/compute/docs/authentication#applications) |

To request access using OAuth 2.0, your application needs the scope information, as well as information that Google supplies when you register your application (such as the client ID and the client secret).

**Tip:** The Google APIs client libraries can handle some of the authorization process for you. They are available for a variety of programming languages; check the [page with libraries and samples](https://web.archive.org/web/20160424230857/https://cloud.google.com/appengine/docs/java/taskqueue/rest/libraries) for more details.

Except as otherwise noted, the content of this page is licensed under the [Creative Commons Attribution 3.0 License](https://web.archive.org/web/20160424230857/http://creativecommons.org/licenses/by/3.0/), and code samples are licensed under the [Apache 2.0 License](https://web.archive.org/web/20160424230857/http://www.apache.org/licenses/LICENSE-2.0). For details, see our [Site Policies](https://web.archive.org/web/20160424230857/https://cloud.google.com/site-policies).

Last updated January 14, 2016.

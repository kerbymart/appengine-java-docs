# Standard Query Parameters

  

The following query parameters can be used with all methods and all resources in the Google App Engine Task Queue.

Query parameters that apply to all Google App Engine Task Queue operations are shown in the table below.

**Notes (on API keys and auth tokens):**

1.  The `key` parameter is required with every request, **unless you provide an OAuth 2.0 token with the request.**
2.  You **must** send an authorization token with every request that is marked <span class="authrequired">(AUTHENTICATED)</span>. [OAuth 2.0](https://web.archive.org/web/20160424230140/https://cloud.google.com/accounts/docs/OAuth2) is the preferred authorization protocol.
3.  You can provide an OAuth 2.0 token with any request in one of two ways:
    -   Using the `access_token` query parameter like this: `?access_token=``oauth2-token`
    -   Using the HTTP `Authorization` header like this: `Authorization: Bearer` `oauth2-token`

All parameters are optional except where noted.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameter</th>
<th>Meaning</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td id="oa_token"><code>access_token</code></td>
<td>OAuth 2.0 token for the current user.</td>
<td><ul>
<li><a href="https://web.archive.org/web/20160424230140/https://cloud.google.com/accounts/docs/OAuth2WebServer#callinganapi">One possible way</a> to provide an OAuth 2.0 token.</li>
</ul></td>
</tr>
<tr class="even">
<td id="callback"><code>callback</code></td>
<td>Callback function.</td>
<td><ul>
<li>Name of the JavaScript callback function that handles the response.</li>
<li>Used in JavaScript <a href="https://web.archive.org/web/20160424230140/http://en.wikipedia.org/wiki/JSONP">JSON-P</a> requests.</li>
</ul></td>
</tr>
<tr class="odd">
<td id="fields"><code>fields</code></td>
<td>Selector specifying a subset of fields to include in the response.</td>
<td><ul>
<li>For more information, see the <a href="https://web.archive.org/web/20160424230140/https://cloud.google.com/appengine/docs/java/taskqueue/rest/performance.html#partial">partial response</a> section in the Performance Tips document.</li>
<li>Use for better performance.</li>
</ul></td>
</tr>
<tr class="even">
<td id="key"><code>key</code></td>
<td>API key. <span class="required">(REQUIRED*)</span><a href="https://web.archive.org/web/20160424230140/https://console.cloud.google.com/"></a></td>
<td><ul>
<li>*Required unless you provide an OAuth 2.0 token.</li>
<li>Your <a href="https://web.archive.org/web/20160424230140/https://cloud.google.com/console/help/using-keys">API key</a> identifies your project and provides you with API access, quota, and reports.</li>
<li>Obtain your project's API key from the <a href="https://web.archive.org/web/20160424230140/https://console.cloud.google.com/">Google Cloud Platform Console</a>.</li>
</ul></td>
</tr>
<tr class="odd">
<td id="pp"><code>prettyPrint</code></td>
<td><p>Returns response with indentations and line breaks.</p></td>
<td><ul>
<li>Returns the response in a human-readable format if <code>true</code>.</li>
<li>Default value: <code>true</code>.</li>
<li>When this is <code>false</code>, it can reduce the response payload size, which might lead to better performance in some environments.</li>
</ul></td>
</tr>
<tr class="even">
<td id="quotaUser"><code>quotaUser</code></td>
<td>Alternative to <code>userIp</code>.</td>
<td><ul>
<li>Lets you enforce per-user quotas from a server-side application even in cases when the user's IP address is unknown. This can occur, for example, with applications that run cron jobs on App Engine on a user's behalf.</li>
<li>You can choose any arbitrary string that uniquely identifies a user, but it is limited to 40 characters.</li>
<li>Overrides <code>userIp</code> if both are provided.</li>
<li>Learn more about <a href="https://web.archive.org/web/20160424230140/https://cloud.google.com/console/help/capping-usage">capping usage</a>.</li>
</ul></td>
</tr>
<tr class="odd">
<td id="userIp"><code>userIp</code></td>
<td>IP address of the end user for whom the API call is being made.</td>
<td><ul>
<li>Lets you enforce per-user quotas when calling the API from a server-side application.</li>
<li>Learn more about <a href="https://web.archive.org/web/20160424230140/https://cloud.google.com/console/help/capping-usage">capping usage</a>.</li>
</ul></td>
</tr>
</tbody>
</table>

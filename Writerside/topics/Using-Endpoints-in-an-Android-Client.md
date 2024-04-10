# Using Endpoints in an Android Client

  

**Note:** These instructions assume familiarity with Android development and concepts including project setup, Activities and AsyncTasks classes, shared preferences, Android permissions, and Intents. If you would like to learn more about these concepts, refer to the [Android documentation and examples](https://web.archive.org/web/20160424225324/http://developer.android.com/).

The recommended way to add a backend to a new or existing Android app is to use Android Studio's [backend templates for endpoints](https://web.archive.org/web/20160424225324/https://cloud.google.com/mobile/endpoints/add_module) feature. In addition to supporting the addition of an Endpoints backend, Android Studio also provides code validation of your backend, lets you debug the backend along with the Android app, and deploys your backend to production App Engine.

You can find the following complete instructions for using this feature to add a backend to your Android project in the README at this GitHub repo:

-   [Simple Hello Endpoints](https://web.archive.org/web/20160424225324/https://github.com/GoogleCloudPlatform/gradle-appengine-templates/tree/master/HelloEndpoints): shows the basics of adding a backend and connecting it to your Android app.

## Making authenticated calls

**Important:** For information on authentication from the perspective of keeping the backend secure, see the blog post [Verifying Back-End Calls from Android Apps](https://web.archive.org/web/20160424225324/https://android-developers.blogspot.com/2013/01/verifying-back-end-calls-from-android.html).

These instructions cover only the client coding you need to add. They assume that you have already added the Endpoints support for auth as described in [Adding Authentication to Endpoints](https://web.archive.org/web/20160424225324/https://cloud.google.com/appengine/docs/java/endpoints/auth).

If your Android client is making calls to an Endpoint that requires authentication, you must:

-   [Configure your Android Client](#Configuring_your_Android_client_to_provide_credentials) to provide credentials to the [service object](#creating_the_service_object).
-   Use the [account picker](#using_the_account_picker) to support user choice of login accounts.

### Configuring Your Android client to provide credentials

To support requests to an Endpoint that requires authentication, your Android client needs to get user credentials and pass them to the [service object](#creating_the_service_object).

Getting the user credentials and using the [account picker](#using_the_account_picker) requires you to have the following Android permissions:

```
<uses-permission android:name="android.permission.GET_ACCOUNTS" />
<uses-permission android:name="android.permission.USE_CREDENTIALS" />
```

To get the user credentials, you call `GoogleAccountCredential.usingAudience` as follows:

```
// Inside your Activity class onCreate method
settings = getSharedPreferences(
    "TicTacToeSample", 0);
credential = GoogleAccountCredential.usingAudience(this,
    "server:client_id:1-web-app.apps.googleusercontent.com");
```

where the second parameter to the call is the prefix `server:client_id` prepended to the web client ID of the backend API.

#### Sample code: getting credentials for the service object

The following code shows how to get credentials and pass it to the service object:

```
// Inside your Activity class onCreate method
settings = getSharedPreferences("TicTacToeSample", 0);
credential = GoogleAccountCredential.usingAudience(this,
    "server:client_id:1-web-app.apps.googleusercontent.com");
setSelectedAccountName(settings.getString(PREF_ACCOUNT_NAME, null));

Tictactoe.Builder builder = new Tictactoe.Builder(
    AndroidHttp.newCompatibleTransport(), new GsonFactory(),
    credential);
service = builder.build();

if (credential.getSelectedAccountName() != null) {
  // Already signed in, begin app!
} else {
  // Not signed in, show login window or request an account.
}

// setSelectedAccountName definition
private void setSelectedAccountName(String accountName) {
  SharedPreferences.Editor editor = settings.edit();
  editor.putString(PREF_ACCOUNT_NAME, accountName);
  editor.commit();
  credential.setSelectedAccountName(accountName);
  this.accountName = accountName;
}
```

The above sample code looks up any shared preferences stored by the Android app and attempts to find the name of the account the user would like to use to authenticate to your application. If successful, the code creates a credentials object and passes it into your service object. These credentials allow your Android application to pass the appropriate token to your backend.

Notice that the code sample checks to see whether or not the Android app already knows which account to use. If it does, the logical flow can continue on and run the Android app. If the app doesn't know which account to use, the app should display a login screen or prompt the user to [pick an account](#using_the_account_picker).

Finally, the sample creates a credentials object and passes it into the service object. These credentials allow your Android application to pass the appropriate token to your App Engine web app.

### Using the account picker

Android provides an intent to select a user account. You can invoke this as follows:

```
static final int REQUEST_ACCOUNT_PICKER = 2;

void chooseAccount() {
  startActivityForResult(credential.newChooseAccountIntent(),
      REQUEST_ACCOUNT_PICKER);
}
```

A handler for interpreting the result of this intent is shown here:

```
@Override
protected void onActivityResult(int requestCode, int resultCode,
   Intent data) {
 super.onActivityResult(requestCode, resultCode, data);
 switch (requestCode) {
   case REQUEST_ACCOUNT_PICKER:
     if (data != null && data.getExtras() != null) {
       String accountName =
           data.getExtras().getString(
               AccountManager.KEY_ACCOUNT_NAME);
       if (accountName != null) {
         setSelectedAccountName(accountName);
         SharedPreferences.Editor editor = settings.edit();
         editor.putString(PREF_ACCOUNT_NAME, accountName);
         editor.commit();
         // User is authorized.
       }
     }
     break;
  }
}
```

## Testing an Android client against a local development server

You can test your client against a backend API running in production App Engine at any time without making any changes. However, if you want to test your client against a backend API running on the local development server, you'll need to change one line of code in the client to point to the IP address of the machine running the local development server.

**Note:** You can use either an Android AVD emulator, or a physical device. However, physical devices must use the same Wi Fi as the system running the local development server.

To make the required changes and test using the local dev server:

1.  Note the IP address of the machine that is running the local dev server: you need it when you add code to the Android client.

2.  Start the local development server, as described in [Running and testing API backends locally](https://web.archive.org/web/20160424225324/https://cloud.google.com/appengine/docs/java/endpoints/test_deploy#running_and_testing_api_backends_locally). You may need to specify the address `0.0.0.0` as described at that link.

3.  In your Android Studio client project, locate the code that gets the handle to the backend API service; typically, this code will use a `Builder` to set up the API request.

4.  Override the root URL on the Builder object (this is the URL the Android client connects to in the backend API call) by adding the line:

    ```
    yourBuilderObject.setRootUrl("http://<your-machine-IP-address>:8080/_ah/api");
    ```

    For example:

    ```
    public static Helloworld getApiServiceHandle(@Nullable GoogleAccountCredential credential) {
      // Use a builder to help formulate the API request.
      Helloworld.Builder helloWorld = new Helloworld.Builder(AppConstants.HTTP_TRANSPORT,
          AppConstants.JSON_FACTORY,credential);

      helloWorld.setRootUrl("http://<your-machine-IP-address>:8080/_ah/api");
      return helloWorld.build();
    }
    ```

    Be sure to replace `<your-machine-IP-address>` with your system's actual IP value.

5.  Rebuild your Android client project.

6.  If you want to run your client app on an AVD emulator:

    1.  In Android Studio, select **Tools** &gt; **Android** &gt; **AVD Manager** and start an existing AVD if you have one, otherwise create one first and start it.
    2.  Select **Run** &gt; **Debug `<your-project-name>`**.
    3.  Select your AVD when prompted to choose a device.
    4.  Test your client as desired.

7.  If you want to run your client app on a physical Android device,

    1.  Make sure your Android device is [enabled for debugging](https://web.archive.org/web/20160424225324/http://developer.android.com/tools/device.html#setting-up).
    2.  In Android Studio, select **Run** &gt; **Debug `<your-project-name>`**.
    3.  Select your physical Android device when prompted to choose a device.
    4.  Test your client as desired.

**Important:** If the backend API is Auth-protected, your client must provide support as described above under <span hre="##Adding_authentication_support_with_OAuth_20">Adding Authentication Support with OAuth 2.0</span>. However, when running against the local dev server, no credentials page is displayed in response to a user's attempt to sign in. Instead, the sign-on simply occurs, granting the user access to the protected API, *and the user returned is always `example@example.com`*.

## Sample client source code

For a complete sample client that illustrates the concepts described above, see the [Android client tutorial](https://web.archive.org/web/20160424225324/https://cloud.google.com/appengine/docs/java/endpoints/getstarted/clients/android/).

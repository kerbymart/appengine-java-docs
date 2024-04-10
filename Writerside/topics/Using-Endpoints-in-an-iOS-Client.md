# Using Endpoints in an iOS Client

  

**Note:** This document assumes you familiar with Apple iOS development, including Objective-C®, Xcode®, and general project setup. If you are unfamiliar with these, refer to the documentation and examples on the Apple [iOS developer site](https://web.archive.org/web/20160424225755/https://developer.apple.com/devcenter/ios/index.action).

To use Endpoints in an iOS app, you need to:

1.  Have access to the backend API project that the client is going to access.
2.  Have a system running Mac OSX and Xcode.
3.  [Compile the client library generator and generate your library](#Java_Compiling_the_client_library_generator_and_generating_your_library).
4.  [Add required files](#Java_Adding_required_files_to_your_iOS_project) to your iOS project.
5.  [Create the service object](#Java_Creating_the_service_object) to be used for making requests.
6.  [Call the API](#Java_Calling_the_Endpoint_API_unauthenticated) exposed by the Endpoint.
7.  Optionally, [add support for authenticated calls](#Java_Adding_a_sign-in_dialog_to_your_iOS_client).

## Compiling the client library generator and generating your library

To compile the library generator and then use it to generate your client library:

1.  On the system where you have your backend API project, generate the discovery document from your backend API in RPC format following the instructions provided in [Generating a Discovery Doc](https://web.archive.org/web/20160424225755/https://cloud.google.com/appengine/docs/java/endpoints/endpoints_tool#generating_a_discovery_doc).

2.  On a system running a recent version of Mac OSX that has the [Xcode](https://web.archive.org/web/20160424225755/https://developer.apple.com/devcenter/ios/index.action) development environment installed, check out a read-only copy of the Google APIs Client Library as follows:

    ```
    svn checkout \
        http://google-api-objectivec-client.googlecode.com/svn/trunk/ \
        google-api-objectivec-client-read-only
    ```

    The Google APIs Client Library for Objective-C project contains all of the code for the Objective-C client library as well as the client library generator.

3.  Start Xcode, and in the above download of the Google APIs Client source, locate `google-api-objectivec-client-read-only/Source/Tools/ServiceGenerator/ServiceGenerator.xcodeproj`. This project contains a single target, `ServiceGenerator`. Build this target (**Project-&gt;Build** or **?B)**.

4.  After you build that target, in the Xcode **Project Navigator**, expand the `ServiceGenerator` project and `Products` arrows to display the output binary. Click on the binary and get the full path from the File Inspector.

5.  Open a new Terminal window and invoke `ServiceGenerator`, pasting in the path to the `ServiceGenerator` binary. As arguments, include the path to your discovery document in RPC format that you created previously. Use the following invocation:

    ```
    /Users/foo/Library/Developer/Xcode/DerivedData/ServiceGenerator-abc/Build/Products/Debug/ServiceGenerator \
        ~/src/tictactoe-java/war/WEB-INF/tictactoe-v1-rpc.discovery \
        --outputDir ~/API
    ```

    replacing the paths and application name with those for your own project.

6.  After you successfully complete these procedures, you now have an API directory containing all of the supplementary files you’ll need to access your API (on top of the existing client library). We'll show you how to add these files to your project in the next section.

**Note:** If you modify your Endpoints backend API, you must repeat the above process after regenerating the discovery document.

## Adding required files to your iOS project

To add the required files:

1.  If you are *not* using the Google+ iOS SDK, add the client library to your iOS project in Xcode. (If you *are* using the Google+ iOS SDK, you must skip this step.) There are two ways to add the client library:
    -   The first way is to copy the source files for the library into your project and compile them directly. To do this,
        1.  Open `google-api-objectivec-client-read-only/Source/GTL.xcodeproj` and expand GTLSource and Common.
        2.  Select all the files in the subgroups, excluding the OAuth 2.0/Mac group.
        3.  Drag the selected files into the **Project Navigator** of your own open project. (You may wish to put them into their own group for organizational purposes, but that is not required.)
        4.  Select your project in the **Add to targets** selection area.
        5.  If your project uses Automatic Reference Counting (ARC), disable it for the client library files you include.
        6.  Under the target **Build Phases** tab for your project, and expand the **Compile Sources** build phase.
        7.  Select the library source files, then press Enter to open the edit field., and type `-fno-objc-arc` as the compiler flag for those files.
    -   The second way to add the client library is to compile and link to the static iOS framework, and is described in the [objectivec client](https://web.archive.org/web/20160424225755/https://github.com/google/google-api-objectivec-client/wiki/BuildingTheLibrary) documentation.  

2.  If you are using the Google+ iOS SDK, set up your Xcode project using the [instructions provided by that SDK](https://web.archive.org/web/20160424225755/https://developers.google.com/+/mobile/ios/getting-started).

3.  Your project needs to include the `foo.h` & `foo_Sources.m` files generated from ServiceGenerator, which you ran previously, where `foo` is your API name. Disabling ARC is not required for these files.

## Creating the service object

The service object is required for making requests to the Endpoint API. If your app makes unauthenticated requests, you construct a service object as follows:

```
static GTLServiceTictactoe *service = nil;
if (!service) {
  service = [[GTLServiceTictactoe alloc] init];
  service.retryEnabled = YES;
  [GTMHTTPFetcher setLoggingEnabled:YES];
}
```

For authenticated calls, you'll need to [supply credentials to the service object](#Java_Adding_a_sign-in_dialog_to_your_iOS_client).

## Calling the Endpoint API (unauthenticated)

After you create a service object, calling your API is straightforward, as shown by this example:

```
GTLQueryTictactoe *query = [GTLQueryTictactoe queryForScoresList];
[service executeQuery:query completionHandler:^(GTLServiceTicket *ticket, GTLTictactoeScores *object, NSError *error) {
  NSArray *items = [object items];
  // Do something with items.
}];
```

In the above sample code, the call requests a list of all `Score` objects on the server and passes the result to a callback. If `queryForScoresList` required parameters, or a request body, you would set them on the query object. (Xcode’s hinting is very helpful here).

If your iOS app is making calls to an Endpoint that requires authentication, you must [Add a Sign-in Dialog](#Java_Adding_a_sign-in_dialog_to_your_iOS_client) to your iOS client.

For more information on making authenticated Endpoints calls, see [Adding Authentication to Endpoints](https://web.archive.org/web/20160424225755/https://cloud.google.com/appengine/docs/java/endpoints/auth).

### Adding a sign-in dialog to your iOS client

The client library provides a "ready made" sign-in user interface, which you present to the user by means of the following code:

```
static NSString *const kKeychainItemName = @"Your App Name";
NSString *kMyClientID = @"your_client_id";
// Client secret is not required, and can be left blank.
NSString *kMyClientSecret = @"your_client_secret";

GTMOAuth2ViewControllerTouch *viewController;
viewController = [[GTMOAuth2ViewControllerTouch alloc] initWithScope:scope
    clientID:kMyClientID
    clientSecret:kMyClientSecret
    keychainItemName:kKeychainItemName
    delegate:self
    finishedSelector:@selector(viewController:finishedWithAuth:error:)];

[self presentModalViewController:viewController
    animated:YES];
```

You place this code within a method of your view controller. When the user signs in, this code triggers a callback called `finishedWithAuth`. An example implementation follows:

```
- (void)viewController:(GTMOAuth2ViewControllerTouch *)viewController
      finishedWithAuth:(GTMOAuth2Authentication *)auth
                 error:(NSError *)error {
    [self dismissModalViewControllerAnimated:YES];

    if (error != nil) {
        // Authentication failed
    } else {
        // Authentication succeeded
        [[self tictactoeService] setAuthorizer:auth];
        // Make some API calls
    }
}

- (GTLServiceTictactoe *)tictactoeService {
    static GTLServiceTictactoe *service = nil;
    if (!service) {
        service = [[GTLServiceTictactoe alloc] init];

        service.retryEnabled = YES;

        [GTMHTTPFetcher setLoggingEnabled:YES];
    }
    return service;
}
```

As a result of the above code, the service object is set to the user that signed into your application.

## Sample client source code

For a complete sample client that illustrates the concepts described above, see the [iOS client](https://web.archive.org/web/20160424225755/https://github.com/GoogleCloudPlatform/appengine-endpoints-tictactoe-ios) sample, which uses the sample [Tic Tac Toe web backend](https://web.archive.org/web/20160424225755/https://github.com/GoogleCloudPlatform/appengine-endpoints-tictactoe-java).

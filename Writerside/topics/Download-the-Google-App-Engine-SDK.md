# Download the Google App Engine SDK

  

**Note:** The App Engine SDK is under active development; please keep this in mind as you explore its capabilities. See the [Java](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/java/release-notes) | [Python](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/python/release-notes) | [Go](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/go/release-notes) | [PHP](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/php/release-notes) SDK Release Notes for information on the most recent changes to the App Engine SDK. If you discover any issues, feel free to notify us via our [Issue Tracker](https://web.archive.org/web/20160424230233/http://code.google.com/p/googleappengine/issues/list).  
  
By downloading, you agree to be bound by the [Terms](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/terms) that govern use of the App Engine SDK.

## Contents

1.  [Google App Engine SDK for Python](#Google_App_Engine_SDK_for_Python)
2.  [Google App Engine SDK for Java](#Google_App_Engine_SDK_for_Java)
3.  [Google App Engine SDK for PHP](#Google_App_Engine_SDK_for_PHP)
4.  [Google App Engine SDK for Go](#Google_App_Engine_SDK_for_Go)
5.  [Previous SDK versions](#Previous_SDK_versions)

## Google App Engine SDK for Python

<table>
<thead>
<tr class="header">
<th>Platform</th>
<th>Version</th>
<th>Package</th>
<th>Size</th>
<th>SHA1 Checksum</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Windows</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/GoogleAppEngine-1.9.36.msi">GoogleAppEngine-1.9.36.msi</a></td>
<td>53.8 MB</td>
<td>960cfe2157c6e984802db4b0224cfe8273d727dc</td>
</tr>
<tr class="even">
<td>Mac OS X</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/GoogleAppEngineLauncher-1.9.36.dmg">GoogleAppEngineLauncher-1.9.36.dmg</a></td>
<td>58.7 MB</td>
<td>88f267fedca0f02f951107389f85aa3350d71137</td>
</tr>
<tr class="odd">
<td>Linux/Other Platforms</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.36.zip">google_appengine_1.9.36.zip</a></td>
<td>40.2 MB</td>
<td>8920584d79332098c2a8248d908a6e27ad7f5697</td>
</tr>
</tbody>
</table>

#### Installing on Linux

To install on Linux:

1.  Unzip the App Engine SDK file you downloaded (*google\_appengine\_1.9.36.zip*), for example:

    ```
    unzip google_appengine_1.9.36.zip 
    ```

    There is no App Engine installation script that you need to run after unzipping the files.

2.  Add the `google_appengine` directory to your PATH:

    ```
    export PATH=$PATH:/path/to/google_appengine/
    ```

3.  Make sure Python 2.7 is installed on your machine using the following command:

    ```
    /usr/bin/env python -V
    ```

    The output should look like this: `Python 2.7.<number>`. If Python 2.7 isn't installed, install it now using the installation instructions for your Linux distribution.

#### Installing on Mac OS X

To install the SDK on Mac OS X:

1.  In the Finder, click **Go &gt; Applications** to open the Applications folder.

2.  Double click the App Engine SDK file you downloaded (*GoogleAppEngineLauncher-1.9.36.dmg*) to open it, then drag the **GoogleAppEngineLauncher** icon over to the Applications folder.

3.  Double-click **GoogleAppEngineLauncher** in the Application folder.

4.  When prompted to *Make command symlinks*, click **OK**. The symlinks allow you to run important SDK command-line tools in any terminal window.

    **Important:** The GoogleAppEngineLauncher is a convenient UI-based tool for running and deploying App Engine apps, but it *does not* provide all the features you'll need. You will need to use the command-line equivalent, `appcfg.py`, for many of the features you'll want to use.

5.  Notice that the installation process above unpacks the contents of the App Engine SDK at the location:

    ```
    /usr/local/google_appengine
    ```

6.  The App Engine SDK requires Python 2.7, which is installed by default on Mac OS X 10.6 (Lion) or later. Verify your Mac's Python installation using the following command:

    ```
    /usr/bin/env python -V
    ```

    If the output looks like `Python 2.7.<number>` then you already have the correct Python version installed. Otherwise you can download and install Python 2.7 from [the Python web site](https://web.archive.org/web/20160424230233/https://www.python.org/download/releases/2.7.4).

#### Installing on Windows

To install the SDK on Windows:

1.  Double-click the SDK file you downloaded (*GoogleAppEngine-1.9.36.msi*) and follow the prompts to install the SDK.
2.  You will need Python 2.7 to use the App Engine SDK, because the [Development Server](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/php/tools/devserver) is a Python application. Download Python 2.7.X (don't use a higher version) from [the Python web site](https://web.archive.org/web/20160424230233/https://www.python.org/download/releases/2.7.4).

See the [Python documentation](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/python/tools/libraries27) for more information on developing for Google App Engine with Python.

## Google App Engine SDK for Java

<table>
<thead>
<tr class="header">
<th>Version</th>
<th>Package</th>
<th>Size</th>
<th>SHA1 Checksum</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/appengine-java-sdk-1.9.36.zip">appengine-java-sdk-1.9.36.zip</a></td>
<td>204.6 MB</td>
<td>2aca5d41f3c6f6bb0af074f2e39c9e06b94ad61e</td>
</tr>
</tbody>
</table>

#### Installing on Linux

To install on Linux:

1.  Unzip the App Engine SDK file you downloaded (*appengine-java-sdk-1.9.36.zip*) to a directory of your choice. For example:

    ```
    unzip appengine-java-sdk-1.9.36.zip 
    ```

    There is no App Engine installation script that you need to run after unzipping the files.

2.  Add the `appengine-java-sdk-1.9.36` directory to your PATH:

    ```
    export PATH=$PATH:/path/to/appengine-java-sdk-1.9.36/bin/
    ```

3.  The App Engine Java SDK requires Java 7 bytecode level. You can use either Java 7 or Java 8; be sure to set the javac compiler flags to generate 1.7 bytecode:

    ```
      -source 1.7 -target 1.7
    ```

    If you don't have Java or the correct Java version installed, download and install the [Java SE Development Kit (JDK)](https://web.archive.org/web/20160424230233/http://java.sun.com/javase/downloads/index.jsp) for Linux.

#### Installing on Mac OS X

To install the SDK on Mac OS X:

1.  Double click the App Engine SDK `.zip` file you downloaded (*appengine-java-sdk-1.9.36.zip*) to extract the SDK into a folder.

2.  The App Engine Java SDK requires Java 7 bytecode level. You can use either Java 7 or Java 8; be sure to set the javac compiler flags to generate 1.7 bytecode:

    ```
    -source 1.7 -target 1.7
    ```

    If you don't have Java or the correct Java version installed, download and install the [Java SE Development Kit (JDK)](https://web.archive.org/web/20160424230233/http://java.sun.com/javase/downloads/index.jsp) for Mac OS X.

3.  Add the `appengine-java-sdk-1.9.36` directory to your PATH:

    ```
    export PATH=$PATH:/path/to/appengine-java-sdk-1.9.36/bin/
    ```

#### Installing on Windows

To install the SDK on Windows:

1.  Double click the App Engine SDK `.zip` file you downloaded (*appengine-java-sdk-1.9.36.zip*) to extract the SDK into a folder.

2.  The App Engine Java SDK requires Java 7 bytecode level. You can use either Java 7 or Java 8; be sure to set the javac compiler flags to generate 1.7 bytecode:

    ```
      -source 1.7 -target 1.7
    ```

    If you don't have Java or the correct Java version installed, download and install the [Java SE Development Kit (JDK)](https://web.archive.org/web/20160424230233/http://java.sun.com/javase/downloads/index.jsp) for Windows.

## Google App Engine SDK for PHP

<table>
<thead>
<tr class="header">
<th>Platform</th>
<th>Version</th>
<th>Package</th>
<th>Size</th>
<th>SHA1 Checksum</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Windows</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/GoogleAppEngine-1.9.36.msi">GoogleAppEngine-1.9.36.msi</a></td>
<td>53.8 MB</td>
<td>960cfe2157c6e984802db4b0224cfe8273d727dc</td>
</tr>
<tr class="even">
<td>Mac OS X</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/GoogleAppEngineLauncher-1.9.36.dmg">GoogleAppEngineLauncher-1.9.36.dmg</a></td>
<td>58.7 MB</td>
<td>88f267fedca0f02f951107389f85aa3350d71137</td>
</tr>
<tr class="odd">
<td>Linux/Other Platforms</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/google_appengine_1.9.36.zip">google_appengine_1.9.36.zip</a></td>
<td>40.2 MB</td>
<td>8920584d79332098c2a8248d908a6e27ad7f5697</td>
</tr>
</tbody>
</table>

#### Installing on Linux

To install on Linux:

1.  Unzip the App Engine PHP file you downloaded (google\_appengine\_1.9.36.zip). For example:

    ```
    unzip google_appengine_1.9.36.zip 
    ```

    There is no App Engine installation script that you need to run after unzipping the files.

2.  Add the `google_appengine` directory to your PATH:

    ```
    export PATH="$PATH:/path/to/google_appengine/"
    ```

3.  Make sure Python 2.7 is installed on your machine using the following command:

    ```
    /usr/bin/env python -V
    ```

    The output should look like this: `Python 2.7.<number>`. If Python 2.7 isn't installed, install it now using the installation instructions for your Linux distribution.

4.  Build and install the [PHP interpreter](https://web.archive.org/web/20160424230233/https://github.com/GoogleCloudPlatform/appengine-php) and [App Engine PHP extension](https://web.archive.org/web/20160424230233/https://github.com/GoogleCloudPlatform/appengine-php-extension). Specify the path to `php-cgi` and `gae_runtime_module.so` when [running the development server](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/php/tools/devserver#PHP_Running_a_custom_php_install).

5.  Install MySQL on your local machine. (You will need this to test your app locally.) On Debian-based Linux systems, you can use the following command:

    ```
    sudo apt-get install mysql-server-5.5
    ```

    You will be prompted to enter a password for the MySQL root user; make sure that you remember it.

#### Installing on Mac OS X

To install the SDK on Mac OS X:

1.  In the Finder, click **Go &gt; Applications** to open the Applications folder.

2.  Double click the App Engine SDK file you downloaded (*GoogleAppEngineLauncher-1.9.36.dmg*) to open it, then drag the **GoogleAppEngineLauncher** icon over to the Applications folder.

3.  Double-click **GoogleAppEngineLauncher** in the Application folder.

4.  When prompted to *Make command symlinks*, click **OK**. The symlinks allow you to run important SDK command-line tools in any terminal window.

    **Important:** The GoogleAppEngineLauncher is a convenient UI-based tool for running and deploying App Engine apps, but it *does not* provide all the features you'll need. You will need to use the command-line equivalent, `appcfg.py`, for many of the features you'll want to use.

5.  Notice that the installation process above unpacks the contents of the App Engine SDK at the location:

    ```
    /usr/local/google_appengine
    ```

6.  The App Engine PHP SDK requires Python 2.7, which is installed by default on Mac OS X 10.6 (Lion) or later. Verify your Mac's Python installation using the following command:

    ```
      /usr/bin/env python -V
    ```

    If the output looks like `Python 2.7.<number>` then you already have the correct Python version installed. Otherwise you can download and install Python 2.7 from [the Python web site](https://web.archive.org/web/20160424230233/https://www.python.org/download/releases/2.7.4).

7.  Most PHP apps use a MySQL backend, which is not packaged with the App Engine SDK. To setup MySQL, visit the [MySQL Downloads page](https://web.archive.org/web/20160424230233/http://dev.mysql.com/downloads/); the "Community Server" edition will suffice for your local development.

#### Installing on Windows

To install the SDK on Windows:

1.  Double-click the SDK file you downloaded (*GoogleAppEngine-1.9.36.msi*) and follow the prompts to install the SDK.

2.  You will need Python 2.7 to use the App Engine PHP SDK, because the [Development Server](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/php/tools/devserver) is a Python application. Download Python 2.7.X (don't use a higher version) from [the Python web site](https://web.archive.org/web/20160424230233/https://www.python.org/download/releases/2.7.4).

    **Note:** The PHP SDK includes binaries for the PHP 5.4 runtime, including all [enabled extensions](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/php/#PHP_Enabled_extensions), so there is no need to download PHP separately for the purposes of developing with App Engine -- you just need Python.

3.  Most PHP apps use a MySQL backend, which is not packaged with the App Engine SDK. To set up MySQL, visit the [MySQL Downloads page](https://web.archive.org/web/20160424230233/http://dev.mysql.com/downloads/); the "Community Server" edition will suffice for your local development.

See the [PHP documentation](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/php) for more information on developing for Google App Engine with PHP.

## Google App Engine SDK for Go

<table>
<thead>
<tr class="header">
<th>Platform</th>
<th>Version</th>
<th>Package</th>
<th>Size</th>
<th>SHA1 Checksum</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Linux 64-bit</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/go_appengine_sdk_linux_amd64-1.9.36.zip">go_appengine_sdk_linux_amd64-1.9.36.zip</a></td>
<td>59.7 MB</td>
<td>91ce2e3608c25e8c1ac3155ab5041abeb537c8e6</td>
</tr>
<tr class="even">
<td>Linux 32-bit</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/go_appengine_sdk_linux_386-1.9.36.zip">go_appengine_sdk_linux_386-1.9.36.zip</a></td>
<td>57.8 MB</td>
<td>f52e31cba487373f0605f5d5e37257e4c7fddc42</td>
</tr>
<tr class="odd">
<td>Mac OS X 64-bit</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/go_appengine_sdk_darwin_amd64-1.9.36.zip">go_appengine_sdk_darwin_amd64-1.9.36.zip</a></td>
<td>59.5 MB</td>
<td>e2d015107f2fe46554e7d53e3ebaa7b0685b70a6</td>
</tr>
<tr class="even">
<td>Mac OS X 32-bit</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/go_appengine_sdk_darwin_386-1.9.36.zip">go_appengine_sdk_darwin_386-1.9.36.zip</a></td>
<td>57.6 MB</td>
<td>e392bc77d847b1ca14669e68d0ba6b4ca8755a47</td>
</tr>
<tr class="odd">
<td>Windows 64-bit</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/go_appengine_sdk_windows_amd64-1.9.36.zip">go_appengine_sdk_windows_amd64-1.9.36.zip</a></td>
<td>60.2 MB</td>
<td>c63764c8dec821ec818c613aa35b8bedf4518c9f</td>
</tr>
<tr class="even">
<td>Windows 32-bit</td>
<td>1.9.36 - 2016-04-18</td>
<td><a href="https://web.archive.org/web/20160424230233/https://storage.googleapis.com/appengine-sdks/featured/go_appengine_sdk_windows_386-1.9.36.zip">go_appengine_sdk_windows_386-1.9.36.zip</a></td>
<td>58.2 MB</td>
<td>7e256e20fe5c2a2f42fbfd1a1d553106704f16b4</td>
</tr>
</tbody>
</table>

#### Installing on Linux

To install on Linux 32-bit or 64-bit:

1.  Unzip the App Engine SDK file you downloaded (*go\_appengine\_sdk\_linux\_amd64-1.9.36.zip* or *go\_appengine\_sdk\_linux\_386-1.9.36.zip*) to a directory of your choice. For example:

    ```
    unzip go_appengine_sdk_linux_amd64-1.9.36.zip 
    ```

    There is no App Engine installation script that you need to run after unzipping the files.

2.  Add the `go_appengine` directory to your PATH:

    ```
    export PATH=$PATH:/path/to/go_appengine/
    ```

3.  Go requires Python 2.7.x; don't use a higher version. (The Go SDK uses tools from the App Engine Python SDK, so Python is required.) Make sure Python 2.7 is installed on your machine using the following command:

    ```
    /usr/bin/env python -V
    ```

    The output should look like this: `Python 2.7.<number>`. If Python 2.7 isn't installed, install it now using the installation instructions for your Linux distribution.

#### Installing on Mac OS X

To install the SDK on Mac OS X 32-bit or 64-bit:

1.  Double-click the App Engine SDK file you downloaded (*go\_appengine\_sdk\_darwin\_amd64-1.9.36.zip* or *go\_appengine\_sdk\_darwin\_386-1.9.36.zip*) to extract the SDK to a folder. It doesn't matter where you install the SDK. We suggest adding the folder to your PATH:

    ```
    export PATH=/path/to/go_appengine:$PATH
    ```

2.  Go requires Python 2.7.x; don't use a higher version. (The Go SDK uses tools from the App Engine Python SDK, so Python is required.) Make sure Python 2.7 is installed on your machine using the following command:

    ```
    /usr/bin/env python -V
    ```

    The output should look like this: `Python 2.7.<number>`. If Python 2.7 isn't installed, install it now from the [Python web site](https://web.archive.org/web/20160424230233/https://www.python.org/download/releases/2.7.4).

**Note:** The Google App Engine Launcher does not work with Go apps.

#### Installing on Windows

To install the SDK on Windows 64-bit or 32-bit:

1.  Double-click the App Engine SDK file you downloaded (*go\_appengine\_sdk\_windows\_amd64-1.9.36.zip* or *go\_appengine\_sdk\_windows\_386-1.9.36.zip*) to extract the SDK files to a location of your choice. It doesn't matter where you install the SDK; however you might want to add the location to your %PATH%.

2.  Go requires Python 2.7.x; don't use a higher version. (The Go SDK uses tools from the App Engine Python SDK, so Python is required.) Make sure Python 2.7 is installed on your machine using the following command:

    ```
      python -V
    ```

    The output should look like this: `Python 2.7.<number>`. If Python 2.7 isn't installed, install it now from the [Python web site](https://web.archive.org/web/20160424230233/https://www.python.org/download/releases/2.7.4).

3.  Optionally, to simplify development and deployment, consider adding the App Engine Go SDK directory to your PATH environment variable by finding the "Environment Variables" in the Advanced System Settings dialog and setting it there.

See the [Go documentation](https://web.archive.org/web/20160424230233/https://cloud.google.com/appengine/docs/go) for more information on developing for Google App Engine with Go.

## Previous SDK versions

SDKs for previous versions of App Engine can be accessed at [https://console.cloud.google.com/storage/appengine-sdks/deprecated/](https://web.archive.org/web/20160424230233/https://console.cloud.google.com/storage/appengine-sdks/deprecated/). You will need to log in with your Google credentials to access this page.

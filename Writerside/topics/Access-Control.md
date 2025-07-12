# Access Control

  

[Python](https://web.archive.org/web/20160424230301/https://cloud.google.com/appengine/docs/python/console/access-control "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230301/https://cloud.google.com/appengine/docs/php/console/access-control "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230301/https://cloud.google.com/appengine/docs/go/console/access-control "View this page in the Go runtime")

**Beta**

This is a Beta release of IAM access control. This feature is not covered by any SLA or deprecation policy and may be subject to backward-incompatible changes.

This document describes the access control options available to you in App Engine, including [IAM](https://web.archive.org/web/20160424230301/https://cloud.google.com/iam/docs/overview#roles) curated controls.

## Contents

1.  [Overview](#overview)
2.  [Choosing the right access control](#choosing_the_right_access_control)
3.  [Setting access control](#setting_access_control)
4.  [Primitive Roles](#primitive_roles)
5.  [Curated App Engine roles](#curated_app_engine_roles)
    1.  [Curated roles comparison matrix](#curated_roles_comparison_matrix)
    2.  [Permissions the Curated Roles do NOT Grant](#permissions_the_curated_roles_do_not_grant)
    3.  [Using Curated Roles with Deployments](#using_curated_roles_with_deployments)
6.  [Limitations and restrictions](#limitations_and_restrictions)

## Overview

In App Engine, you use the Google Cloud Platform Console to set access control using roles at the project level for both [primitive roles](#primitive_roles) and [App Engine curated roles](#curated_app_engine_roles). In general, primitive roles provide broader access, whereas curated roles provide a more focused and restricted access.

For more information about IAM access control, see the [IAM overview](https://web.archive.org/web/20160424230301/https://cloud.google.com/iam/docs/overview#roles) page.

## Choosing the right access control

If you are just experimenting with App Engine, the simplest approach to access control is to grant the Editor role to all people involved with the project, following the [console instructions](#setting_access_control_in_the_console) below. Keep in mind that only an Owner can add other people to the project.

When your project is ready for more fine-grained access controls, we recommend that you do the following:

1.  Identify all the different job functions that need access to the project.

2.  [Set up a Google Group](https://web.archive.org/web/20160424230301/https://support.google.com/groups/answer/2464926?hl=en) for each of these job functions.

3.  Add members as desired to each Google Group.

4.  Follow the [console instructions](#setting_access_control_in_the_console) below to add each Google Group as member of the project and set roles on each group.

## Setting access control

To set access controls:

1.  Visit the [Permissions page](https://web.archive.org/web/20160424230301/https://console.cloud.google.com/permissions/projectpermissions) for your project.

2.  Click **Add member** to add new members to the project and set their roles using the dropdown menu. You can add an individual user email or you can supply a Google Group email (`example-google-group@googlegroups.com`) if you use [Google Groups](#choosing_the_right_access_control) to manage group roles as described above.

    ![Add a Group](https://web.archive.org/web/20160424230301im_/https://cloud.google.com/appengine/docs/images/add-group.png)

    Assign one of the primitive roles *Owner*, *Editor*, *Viewer*, or assign one of the App Engine curated roles for more fine-grained control: *App Engine Admin*, *App Engine Viewer*, *App Engine Deployer*, and *App Engine ServiceAdmin*.

    You'll notice that there are [other curated roles](https://web.archive.org/web/20160424230301/https://cloud.google.com/iam/docs/overview#roles) in the dropdown menu. Those apply to other Google Cloud Platform products. We'll cover only the roles that apply to App Engine in this document.

## Primitive Roles

The primitive roles for App Engine and their permissions are the following:

<table>
<thead>
<tr class="header">
<th>Role</th>
<th>Permissions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Viewer</td>
<td>Provides READ access to all resources in the project.</td>
</tr>
<tr class="even">
<td>Editor</td>
<td>Provides <em>Viewer</em> access, plus the ability to write to or modify any resources, upload and roll back application code, and edit application settings.</td>
</tr>
<tr class="odd">
<td>Owner</td>
<td>Provides <em>Editor</em> access, plus the ability to change membership and permissions in the project, disable serving on the app, delete the app, and manage billing.</td>
</tr>
</tbody>
</table>

### Primitive roles and the admin user

Roles and Admin User

In your [application configuration](https://web.archive.org/web/20160424230301/https://cloud.google.com/appengine/docs/java/config/webxml#Security_and_Authentication) (`login:admin`), you can specify that only admin users can access a particular URL. Be aware that this means that any logged in user with Viewer, Editor, or Owner roles can access that URL.

Similarly, if you query the current user to determine whether the user has admin permissions, using [isUserAdmin](https://web.archive.org/web/20160424230301/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/users/UserService.html#isUserAdmin--), the query will return true if the current user has Viewer, Editor, or Owner roles.

## Curated App Engine roles

The curated roles for App Engine provide you with finer grained options for access control. Each role is listed with its targetted user, as follows:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Role</th>
<th>Capabilities</th>
<th>Target User</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>App Engine Admin</td>
<td>Read/Write/Modify access to all application configuration and settings.</td>
<td><ul>
<li>Application owner/administrator</li>
<li>On-call engineer</li>
<li>sysadmin</li>
</ul></td>
</tr>
<tr class="even">
<td>App Engine Viewer</td>
<td>Read-only access to all application configuration and settings.</td>
<td><ul>
<li>User needing visibility into application, but not to modify it.</li>
<li>Audit job checking App Engine configuration for policy compliance.</li>
</ul></td>
</tr>
<tr class="odd">
<td>App Engine Deployer</td>
<td><ul>
<li>Read-only access to all application configuration and settings.</li>
<li>Write access only to create a new version; cannot modify existing versions other than deleting versions that are not receiving traffic.</li>
</ul></td>
<td><ul>
<li>Deployment account</li>
<li>Release engineer</li>
</ul></td>
</tr>
<tr class="even">
<td>App Engine Service Admin</td>
<td><ul>
<li>Read-only access to all application configuration and settings.</li>
<li>Write access to module-level and version-level settings. Cannot deploy a new version.</li>
</ul>
<p><strong>Note:</strong> Service Admin is basically a Module Admin role. However, because we expect to rename Modules to Services soon, this role uses the new name.</p></td>
<td><ul>
<li>Release engineer</li>
<li>DevOps</li>
<li>On-call engineer</li>
<li>Sys Admin</li>
</ul></td>
</tr>
</tbody>
</table>

### Curated roles comparison matrix

The following table provides a complete comparison of the capabilities of each curated App Engine role.

<table>
<thead>
<tr class="header">
<th>Capability</th>
<th>App Engine Admin</th>
<th>App Engine Deployer</th>
<th>App Engine Service Admin</th>
<th>App Engine Viewer</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>List all modules, versions and instances</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr class="even">
<td>View application, module, version and instance settings - traffic splits, instance type, etc.</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr class="odd">
<td>View runtime metrics such as resource usage, load information, and error information</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr class="even">
<td>Deploy a new version of the application</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
</tr>
<tr class="odd">
<td>Change traffic splits, change default version</td>
<td>Yes</td>
<td>No</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr class="even">
<td>Start and stop a version</td>
<td>Yes</td>
<td>No</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr class="odd">
<td>Delete a version</td>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr class="even">
<td>Delete an entire module</td>
<td>Yes</td>
<td>No</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr class="odd">
<td>Shut down an instance</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
<td>No</td>
</tr>
<tr class="even">
<td>Update dispatch rules</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
<td>No</td>
</tr>
<tr class="odd">
<td>Update dos settings</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
<td>No</td>
</tr>
<tr class="even">
<td>Update default cookie expiration</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
<td>No</td>
</tr>
<tr class="odd">
<td>Update referrers</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
<td>No</td>
</tr>
<tr class="even">
<td>Update Email API Authorized Senders</td>
<td>Yes</td>
<td>No</td>
<td>No</td>
<td>No</td>
</tr>
</tbody>
</table>

**Note:** The curated roles are enforced in the Google Cloud Platform Console, the [Admin API](https://web.archive.org/web/20160424230301/https://cloud.google.com/appengine/docs/admin-api/), and other access methods, such as [appcfg](https://web.archive.org/web/20160424230301/https://cloud.google.com/appengine/docs/java/tools/uploadinganapp).

### Permissions the Curated Roles do NOT Grant

None of the curated roles listed above grant access to the following:

-   Disable and re-enable the application
-   View and download application logs
-   View Monitoring charts in the Cloud Platform Console
-   Enable and Disable billing
-   Set up a daily Spending Limit (formerly known as Budget) for App Engine and view dollar amount spent
-   SSH into a VM instance running in the flexible environment
-   View and edit custom domains and uploaded SSL certificates
-   Run Security Scans
-   Access configuration or data stored in Datastore, Task Queues, Memcache, Cloud Search or any other Cloud Platform storage product

We expect to control some of these features in the future by their own fine-grained roles.

### Using Curated Roles with Deployments

The App Engine Deployer role is the recommended role to grant to the account that is responsible for deploying a new version of a module. A user with Deployer permissions will be able to successfully run the following command:

`appcfg.sh update <directory-path>/my-application/my-module`

This command creates a new version of the application as specified in the `app.yaml`, but does not update any of the application-level settings specified in `dos.yaml` or `dispatch.yaml`, nor does it update any of the Task Queue, Cron and Datastore Index information specified in the `queues.yaml`, `cron.yaml` and `index.yaml` files, respectively.

-   If your deployment process involves updating `dos.yaml` or `dispatch.yaml`, you need to grant the App Engine Admin role to the account doing the deployment.

-   If your deployment process also involves updating `queues.yaml`, `cron.yaml` or `index.yaml`, your deployment account needs to be an Editor on the project.

#### Separating deployment and traffic routing duties

Many organizations prefer to separate the task of deploying an application version from the task of ramping up traffic to the newly created version, and to have these tasks done by different job functions. The Deployer and Service Admin roles enable this separation.

A user with only a Deployer role is limited to deploying new versions and deleting old versions that don’t have traffic routed to them. The Deployer-only user won’t be able to change which version gets traffic or change application-level settings such as dispatch rules or authentication domain.

The Service Admin role, on the other hand, cannot create a new version of the application or change application-level settings. However, it can change properties of existing modules and versions, including changing which versions get traffic. We suggest granting the Service Admin role to your Operations/IT department that handles ramping up traffic to newly deployed versions.

## Limitations and restrictions

The following limitations apply to these roles, and some may persist beyond the Beta period:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Restriction</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Apps running in the flexible environment are not supported.</td>
<td>The App Engine curated roles only work for apps running in the App Engine standard environment. This means, for example, that a user trying to deploy a module to the flexible environment must use the primitive Editor role on the project. The Deployer and App Admin roles do not currently grant sufficient permissions for a deployment to the the flexible environment to succeed.</td>
</tr>
<tr class="even">
<td>None of the roles allow access to pages that have a <code>login:admin</code> restriction.</td>
<td>To access a page designated as <code>login:admin</code> in the <code>app.yaml</code>, use an Owner, Viewer or Editor role.<br />
<br />
In the future, we expect that App Admin as well as Service Admin will allow this access.</td>
</tr>
<tr class="odd">
<td>You must use the Google Cloud Platform Console with curated roles.</td>
<td>Users granted these new curated roles must use the Google Cloud Platform Console to manage their applications, not any older, legacy consoles.</td>
</tr>
<tr class="even">
<td>The App Engine application continues to have Editor role on the project.</td>
<td>The application itself, at runtime, continues to have edit access to all the resources in the project. Reducing the permissions of the service account representing the identity of the application is not supported.</td>
</tr>
</tbody>
</table>

---
title: BetEngines. Administration Web Interface
---

## BetEngines

# Administration Web Interface

## Introduction

The Administration Web Interface (AWI) is for Excel files and user management.
Using AWI you can upload, delete and profile your files, as well as manage your users.

## Main panel

<img src="/images/user-guide/panel-eng.png" />

On the Main panel you can find links to the [user management](#user-management) and [file management](#file-management) pages. You can use the "Sign out" button to logout from your account.

{:#file-management}
## File management

<img src="/images/user-guide/files-eng.png" style="width: 1100px"/>

After signing in to AWI you will find yourself on the "Files" page. This page is for managing of your files. The following functions are available on this page:

  * [New file uploading](#file-uploading)
  * Uploaded files list
  * For uploaded files:
    * [File profiling](#file-profiling)
    * File downloading
    * [File updating](#file-updating)
    * File removing

{:#file-uploading}
### New file uploading

<img src="/images/user-guide/upload-file-eng.png" style="width: 700px"/>

The "Upload" page is for new file uploading.
In order to upload a new file, you should set:
 
  * **ID** - unique identifier of the new file. This is required integer field.
  * **File** - file to upload. This is required field.

The file will be uploaded after you click the "Submit" button.

<div class="well well-sm">
<b>Note!</b> Before you upload your file, you should make a few simple adjustments to it as described in the <a href="/en/doc/mengine/file-preparation/">manual</a>.
</div>

{:#file-profiling}
### File profiling

<img src="/images/user-guide/profile-eng.png"/>

<div class="well well-sm">
<b>Note!</b> This feature is for file performance checking. For production usage, you should use <a href="/en/doc/mengine/integration-guide/">MathEngine REST API</a>.
</div>

Using the "Profile" page you can set input parameters for the file manually and check the output value from it.
You can also check total and per-row calculation time.

<div class="well well-sm">
<b>Note!</b> ALWAYS use the "Finish" button after you end your profiling, this will free up computation resources.
</div>

Using the "Options" button you can export and import IN parameters to/from json format.

<img src="/images/user-guide/profile-options.png" />

{:#file-updating}
### File updating

<img src="/images/user-guide/replace.png" width="700px"/>

This page is for file updating. You should choose your new file and hit "Submit" in order to update it.

<div class="well well-sm">
<b>Note!</b> All active interactions with this file using <a href="/en/doc/mengine/integration-guide/">MathEngine REST API</a>
will continue with the old file version. 
All new interactions, with sessions created after the file update, will use the new file version.
</div>

{:#user-management}
## User management

<img src="/images/user-guide/users-eng.png" />

This page is for user management.
You can see a list of all available users on this page. The table contains the following columns:

  * **Login** - user login
  * **Name** - user name
  * **Status** - user status. Can be:
    * **active** - user can login to AWI
    * **inactive** - user is disabled and can't login to AWI
  * **Type** - user role. Can be either:
    * **admin** - this role has access to all AWI functions
    * **empty field** - guest access. A user with this role can only see the "Profiling" page

Actions available on this page:

  * [New user creation](#user-creation)
  * [User editing](#user-editing)
  * [Password change](#password-change)
  * User activation/deactivation
 
{:#user-creation}
### New user creation

<img src="/images/user-guide/create-user.png" width="700px"/>

In order to create a new user, you should fill in the following fields:

  * **Name** - new user's name (e.g. Johnny English)
  * **Login** - new user's login (e.g. johnny005). Login will be used for AWI access.
   Each user should have unique login name. Two users with the same login are not permitted
  * **Is Admin?** - select new user's role. **admin** if checked, **guest** otherwise
  * **Password** - new user's password
  * **Password confirmation** - password confirmation
  
Click "Submit" to create the user after you have filled in all form fields.

{:#user-editing}
### User editing

<img src="/images/user-guide/edit-user.png" width="700px"/>

Using this page you can change the user's name and role.

{:#password-change}
### Password change

<img src="/images/user-guide/user-password.png" width="700px"/>

Using this page you can change the user's password. You should confirm it using the appropriate field.


## Next steps

Now you can begin integration of new services to your system using the [integration guide](/en/doc/mengine/integration-guide/).


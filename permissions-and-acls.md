---
title: Permissions and ACLs
---
# Permissions and ACLs

This guide highlights some of the permission model features that are provided as part of the Catalyze Mobile Backend as a Service (BaaS). The key features covered are:

- Creating custom class entries and uploading files on behalf of another user within an app
- Granting and revoking permissions to custom classes and files
- Managing groups and their associated permissions

When going through this guide it is important to save the response output of the various calls made via the API. Keep it handy within a text document to refer back to in the event that you miss a step, lose an ID value, etc.

This guide assumes that you have already completed the Getting Started guide.

**Note:** For this section of the guide, only API calls can be made to showcase the features.

This step has you get session tokens within the app for Alice (a supervisor) and Bob (a regular user). The commands below assume that you did not change the usernames or passwords for Alice and Bob provided in the Getting Started guide. Change the values accordingly if you adjusted these values previously.

Run the following command to log Bob in:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Content-Type: application/json" https://api.catalyze.io/v2/auth/signin -X POST -d '{"username":"bob","password":"bob123"}'
```

Record the value of Bob's **sessionToken** for future use.

Run the following command to log Alice in:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Content-Type: application/json" https://api.catalyze.io/v2/auth/signin -X POST -d '{"username":"alice","password":"alice123"}'
```

Record the value of Alice's **sessionToken** for future use.

### Create a File for Another User

Often in the medical field and in clinical settings it is common that data is being collected and processed *on behalf* of the patient. For example, a radiologist may want to share image data with a patient to show them that they are cancer free. Of course this example involves PHI and Catalyze BaaS makes this scenario very easy and secure.

In the following examples, we will assume that Alice is the medical doctor and is sharing medical data, in the form of files, with Bob, her patient. During the **Getting Started** section we configured ExampleApp to have Alice and Bob as users, with Alice being a Supervisor. If you did not run through the Getting Started steps, please do so before continuing this guide.

Values needed to run this example:

Value               |DESCRIPTION
--------------------|--------
*filePath*          |The full path of a file to upload. Can be an image, PDF, etc.
*apiKey*            |The API key for ExampleApp
*aliceSessionToken* |a valid session token for Alice obtained from logging in using *apiKey*
*bobSessionToken*   |a valid session token for Bob obtained from logging in using *apiKey*
*aliceUsersId*      |Alice's *usersId*, obtained when signing Alice in to ExampleApp
*bobUsersId*        |Bob's *usersId*, obtained when signing Bob in to ExampleApp

#### Alice Uploads a File for Bob

As a supervisor, Alice has sufficient permission to upload a file and assign its ownership to Bob. The backend storage system will record Alice as the **author** and Bob as the **owner** for any files that Alice uploads to Bob.

Execute this command to upload the file to Bob:

```curl
    curl -F "phi=true" -F "file=@<filePath>" -H "X-Api-Key: browser <apiKey>" -H "Authorization: Bearer <aliceSessionToken>" -X POST https://api.catalyze.io/v2/users/<bobUsersId>/files
```

The call will return with an HTTP 200 along with a JSON body containing **filesId**. Store that value to test that Bob can retrieve the file and then delete it when he is done with it.

Verify that Alice can download the file she just created:

```curl
    curl -H "Accept: application/octet-stream" -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <aliceSessionToken>" -X GET https://api.catalyze.io/v2/users/files/<filesId> > img.png
```

#### Bob Receives the File

Once the file is uploaded to Bob, he has complete control over the file as he is the file's **owner**. Likewise, Alice has complete control over the file as she is the **author**. We will now verify that Bob can manipulate the file.

Execute the following command to return Bob's list of files:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <bobSessionToken>" -X GET https://api.catalyze.io/v2/users/files
```

Example result:

    [{"usersid":"e56d261f-8206-4921-8026-95cf55667b30","authorsid":"0ebfeab0-b0f7-41a1-9f7e-bac7390eb76c","filesid":"48e11b50-af79-49df-8744-4fa9717802e9","createdat":"Mon Jun 02 12:10:36 UTC 2014","phi":"true"}]

The **filesId** from the previous section should appear in the list.

Execute the following command to download the file as Bob:

```curl
    curl -H "Accept: application/octet-stream" -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <bobSessionToken>" -X GET https://api.catalyze.io/v2/users/files/<filesId> > img.png
```

Finally, changing the HTTP method to DELETE will allow Bob to delete the file.

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <bobSessionToken>" -X DELETE https://api.catalyze.io/v2/users/files/<filesId>
```
Example result:

```curl
    {"filesId":"a1925e2a-550c-4241-ba8e-b9b5b9e276b9","createdAt":"Mon Jun 02 12:21:32 UTC 2014","usersId":"e56d261f-8206-4921-8026-95cf55667b30","phi":"true","authorsId":"0ebfeab0-b0f7-41a1-9f7e-bac7390eb76c"}
```

Note that Alice's is listed as the author of the file.

#### Bob Uploads a File for Alice

Trying to have Bob upload a file to Alice will fail as Bob, a regular user, does not have the correct permissions to share a file with Alice. As a supervisor, Alice has complete control over the application's data, whereas Bob can only view and modify data that he has created or that has been shared with him by another user with sufficient permissions.

Here is an example call that will fail with a HTTP 403 and a JSON error message within the result body.

```curl
    curl -F "phi=true" -F "file=@<filePath>" -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <bobSessionToken>" -X POST https://api.catalyze.io/v2/users/<aliceUsersId>/files
```

Error message:

```curl
    {"errors":[{"message":"Insufficient privileges to perform the requested action.","code":403}]}
```

To make the above call work successfully, we will need to elevate Bob's permissions so that he can post files to Alice. See [section 7.3](acls_for_custom_classes_and_files.md) of this guide form information on how to work with Access Control Lists (ACLs).


### Create a Custom Class Entry for Another User

This section is very similar in purpose to the previous section, where a user was able to upload a file on behalf of another user. In that section, Alice, a doctor, shared image data with her patient, Bob, a regular user of the application. This section covers the same use case but here custom classes are used to exchange data instead of files.

Values needed to run this example:

Value               |Description
--------------------|------------
*apiKey*            |The API key for ExampleApp
*aliceSessionToken* |a valid session token for Alice obtained from logging in using *apiKey*
*bobSessionToken* |a valid session token for Bob obtained from logging in using *apiKey*
*bobUsersId* |Bob's *usersId*, obtained when signing Bob in to ExampleApp

#### Alice Creates an Entry for Bob

Run the command below to have Alice create an entry in ExampleClass for Bob.

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <aliceSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/classes/ExampleClass/entry/<bobUsersId> -d '{"content":{"name":"FromAlice","type":123}}
```

Example response:

```curl
    {"content":{"name":"FromAlice","type":123},"parentId":"e56d261f-8206-4921-8026-95cf55667b30","authorId":"0ebfeab0-b0f7-41a1-9f7e-bac7390eb76c","phi":false,"createdAt":1401793517168,"updatedAt":1401793517168,"entryId":"pWFz8uBYXILw3lWOK4OiisNo6M"}
```

Record the value of **entryId** to be used in the next section.

#### Bob Retrieves the Entry

Once Alice's has created the entry, Bob can manipulate it just like any entry he created, as he is the **owner**. Alice also retains control over the entry as she is the **author**.

Bob will see the new entry when he queries the custom class:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <bobSessionToken>" -H "Content-Type: application/json" -X GET 'https://api.catalyze.io/v2/classes/ExampleClass/query?field=name&searchBy=FromAlice'
```

Bob can fetch the new entry given its **entryId**:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <bobSessionToken>" -H "Content-Type: application/json" -X GET 'https://api.catalyze.io/v2/classes/ExampleClass/entry/<entryId>'
```

Bob can delete the entry given its **entryId**:
```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <bobSessionToken>" -H "Content-Type: application/json" -X DELETE 'https://api.catalyze.io/v2/classes/ExampleClass/entry/<entryId>'
```

#### Bob Creates an Entry for Alice

Trying to have Bob create a custom class entry for Alice will fail as Bob, a regular user, does not have the correct permissions to create entries on Alice's behalf. As a supervisor, Alice has complete control over the application's data, whereas Bob can only view and modify data that he has created or that has been shared with him by another user with sufficient permissions.

For Bob to be able to create data for Alice, we will need to elevate Bob's permissions. See section 7.3 of this guide form information on how to work with Access Control Lists (ACLs).


### ACLs for Custom Classes and Files

In this section, we will manipulate the access control lists (ACLs) within ExampleApp to allow a regular user, Bob, to perform various operations against other users that would otherwise be impossible with default permissions. In the case of a regular user like Bob, the only data that he can manipulate (create, read, update or delete - or CRUD) by default is data he created originally, or that has been shared with him by another user, as in the previous two sections. This section shows how we can leverage ACLs to provide users with additional permissions within the application.

Values needed to run this example:

Value                |Description
---------------------|------------
*adminApiKey*        |The transient API key for ExampleApp from calling **/auth/all** (see "Get an Admin-level Session token and API Key" under "Getting Started")
*adminSessionToken*  |The session token from **/auth/all**
*apiKey*             |The API key for ExampleApp
*aliceSessionToken*  |a valid session token for Alice obtained from logging in using *apiKey*
*bobSessionToken*    |a valid session token for Bob obtained from logging in using *apiKey*
*aliceUsersId*       |Alice's *usersId*, obtained when signing Alice in to ExampleApp
*bobUsersId*         |Bob's *usersId*, obtained when signing Bob in to ExampleApp

Within an application, a user can have create, read, update and delete permissions (CRUD) within files and custom classes.

Here is a summary of the permission types:

Permission |Description
-----------|---
*create*   |The ability to create data across the application.
*read*     |The ability to read data across the application. Can list and query data items as well.
*update*   |The ability to update any data item within the application.
*delete*   |The ability to delete any data item within the application.


#### Granting Create on Files

In [section 7.1](create_a_file_for_another_user.md), Bob attempted to upload a file to Alice but failed. To make this work, the application administrator can grant Bob create permission on files within the application.

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/acl/core/files/<bobUsersId> -d '["create"]'
```

Bob can now create a file for Alice:

```curl
    curl -F "phi=true" -F "file=@/Users/uphoff/Desktop/img.png" -H "X-Api-Key: <apiKy>" -H "Authorization: Bearer <bobSessionToken>" -X POST https://api.catalyze.io/v2/users/<aliceUsersId>/files
```

The drawback here is that Bob can now create files for **any** user, not just Alice. An upcoming release will allow entity-level permission granting, allowing Alice to grant Bob, and only Bob, the ability to create files against herself.

The key concept here is that ACLs currently function at the **model** level within an application. Currently a model is either a custom class or a file. If a user has been granted a permission on a model it is valid only for that model. We will be extending ACLs to our clinical data models (medications, problems, procedures, etc) in time.

#### Granting Full Permissions on Files

If we wanted to give Bob full control over files within the application we could extend his permissions to include read, update and delete permissions.

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/acl/core/files/<bobUsersId> -d '["read","update","delete"]'
```

Bob now can perform any operation against any other user's files within the application.

#### Granting Full Permissions on a Custom Class

Granting permission against specific custom classes is also possible. The route is changes slightly (**/core/files** becomes **/custom/&lt;className&gt;**) but otherwise the call is the same:

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/acl/custom/ExampleClass/<bobUsersId> -d '["create","read","update","delete"]'
```

The above call results in Bob having full permissions against the custom class named ExampleClass.

#### Revoking ACLs on Files

Revoking ACLs is a matter of simply changing the POST from the above examples into a DELETE. The following command will revoke Bob's create permissions on files.

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X DELETE https://api.catalyze.io/v2/acl/core/files/<bobUsersId> -d '["create"]'
```

Revoke the remaidner of the permissions with this command:

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X DELETE https://api.catalyze.io/v2/acl/core/files/<bobUsersId> -d '["create","read","update","delete"]'
```

After running the above commands Bob is back to having default permissions on files. He can only manipulate his own data however any files that he created while he held create permissions will still be accessible to him.

#### Revoking ACLs on Custom Classes

Revoking permissions on ExampleClass is almost the same as revoking file permissions, aside from a few changes to the route:

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X DELETE https://api.catalyze.io/v2/acl/custom/ExampleClass/<bobUsersId> -d '["create","read","update","delete"]'
```


### Managing Groups

In this section we will introduce **groups**, group management and granting permissions to groups. Groups are a feature that allows multiple users to be granted uniform access to resources along with the other members of the group. Groups can effectively be used to create new roles beyond the built-in administrator and supervisor roles that the API supports.

Values needed to run this example:

Value|Description
-----|-----------
*adminApiKey* | The transient API key for ExampleApp from calling **/auth/all** (see "Get an Admin-level Session token and API Key" under "Getting Started")
*adminSessionToken* | The session token from **/auth/all**
*apiKey* | The API key for ExampleApp
*aliceSessionToken* | a valid session token for Alice obtained from logging in using *apiKey* 
*bobSessionToken* | a valid session token for Bob obtained from logging in using *apiKey*
*aliceUsersId* | Alice's *usersId*, obtained when signing Alice in to ExampleApp
*bobUsersId* | Bob's *usersId*, obtained when signing Bob in to ExampleApp

#### Create a Group

Group creation is trivial but requires administrator level credentials. Run the following command to create the ExampleGroup.

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSession>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/groups -d '{"name":"ExampleGroup"}'
```

Example result:

    {"name":"ExampleGroup","groupsId":"200902c9-8e84-4863-8f7f-58c5d62b8837"}

Save the value of **groupsId** for future use.

#### Add Users to the Group

Either administrator or supervisor permissions are needed to add users to a group. In this example we will have Alice add Bob and herself to ExampleGroup.

Run this command to add Bob:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <aliceSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/groups/<groupsId>/users/<bobUsersId>
```

Run this command to add Alice:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <aliceApiKey>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/groups/<groupsId>/users/<aliceUsersId>
```

List the group's users to check that Alice and Bob are members of the group:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <aliceSessionToken>" -H "Content-Type: application/json" -X GET https://api.catalyze.io/v2/groups/<groupsId>/members
```

Example result:

```curl
    ["0ebfeab0-b0f7-41a1-9f7e-bac7390eb76c","e56d261f-8206-4921-8026-95cf55667b30"]
```

Alice's **usersId** and Bob's **usersId** should appear in the list returned.

#### Granting ExampleGroup Full Permissions on Files

The previous section shows how ACLs work for individual users. A nice feature of the API is that the ACL functionality works exactly the same for users and groups. The following command will give the group full permissions on files. The call is exactly the same as the one we used to give Bob these permissions but here we substitute Bob's **usersId** for the **groupsId* for the group we just created.

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/acl/core/files/<groupsId> -d '["create","read","update","delete"]'
```

Any member of the group now can perform any operation against any other user's files within the application. New members added to the group will receive these permissions as well. Removing a user from the group will revoke their permissions.

Note that Alice is in the group and thus has full permissions on files in two ways: her role (supervisor) and her group membership in ExampleGroup. These are independent permissions, meaning that if we remove Alice from the group she will still retain full control over files from her role as a supervisor.

#### Granting ExampleGroup Full Permissions on a Custom Class

The call to grant group permissions to ExampleClass is the same as in the previous section, again swapping Bob's usersId for the groupsId. Again, note that the route changes slightly from the previous call (**/core/files** becomes **/custom/&lt;className&gt;**).

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/acl/custom/ExampleClass/<groupsId> -d '["create","read","update","delete"]'
```

The above call results in Bob having full permissions against the custom class named ExampleClass. Alice effectively has the same permissions as she already had as a supervisor.

#### Removoke Group Permissions

Revoking group permissions works in the same manner as revoking user permissions. This call will revoke the group's delete permissions for files:

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X POST https://api.catalyze.io/v2/acl/core/files/<groupsId> -d '["delete"]'
```

Here we revoke delete permission but create, read and update are retained.

#### Remove a User from a Group

Removing users from groups is simply a matter of changing the method to DELETE in the call used above to add Bob to the group:

```curl
    curl -H "X-Api-Key: <apiKey>" -H "Authorization: Bearer <aliceSessionToken>" -H "Content-Type: application/json" -X DELETE https://api.catalyze.io/v2/groups/<groupsId>/users/<bobUsersId>
```

Bob is no longer a member of the group and will lose all associated permissions, unless he has obtained permissions in some other manner (i.e. through a role, membership in another group or individual user permission grants).

#### Delete a Group

Like creating a group, deleting a group requires administrator-level credentials. Run the following command to delete a group:

```curl
    curl -H "X-Api-Key: <adminApiKey>" -H "Authorization: Bearer <adminSessionToken>" -H "Content-Type: application/json" -X DELETE https://api.catalyze.io/v2/groups/<groupsId>
```

After the call, all the permissions inherited by the users of the group will be lost. In our example, Bob will no longer have full file permissions or full custom class permissions. Alice will still maintain these permission as she inherits them from her role as a supervisor.


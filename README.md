# Solr Setup and User Authentication usign basicAuth 

This guide outlines how to set up Apache Solr in cloud mode, enable user authentication, configure roles and permissions, and verify the configuration using `curl`.

# Table of Contents

1. [Install Necessary Packages](#1-install-necessary-packages)
2. [Download and Install Apache Solr](#2-download-and-install-apache-solr)
3. [Start Solr in Cloud Mode](#3-start-solr-in-cloud-mode)
4. [Enable Authentication in Solr](#4-enable-authentication-in-solr)
5. [Add Users and Set Roles](#5-add-users-and-set-roles)
6. [Changing the Password for User1](#6-change-the-password-for-user1)
7. [Create Collections](#7-create-collections)
8. [Test Authentication and Authorization](#8-test-authentication-and-authorization)
9. [Conclusion](#9-conclusion)

---


## 1. Install Necessary Packages

Update the system and install the required packages:

```bash
# Update package lists
sudo apt update

# Install Java Runtime Environment (JRE)
sudo apt install default-jre -y

# Install wget for downloading files
sudo apt-get install wget -y

# Install text editors (nano and vim)
sudo apt install nano -y
sudo apt install vim -y

# Install sudo if not installed
sudo apt install sudo -y
```

## 2. Download and Install Apache Solr

Download Solr version 8.11.2 and extract it:

```bash
# Download Solr tarball
wget https://archive.apache.org/dist/lucene/solr/8.11.2/solr-8.11.2.tgz

# Extract the Solr tarball
tar -xzf solr-8.11.2.tgz
```

## 3. Start Solr in Cloud Mode

Navigate to the extracted Solr directory and start Solr in cloud mode:

```bash
cd solr-8.11.2/

# Start Solr in cloud mode
bin/solr start -e cloud -force
```
This will start Solr in cloud mode. You will be prompted for the following:

    Node Numbers: Enter 1 (based on the requirements).
    Port: Enter 8983 (or any available port).
After a successful start, you will see a message: "Happy searching!" indicating that Solr is running successfully.

## 4. Enable Authentication in Solr

Now, enable Solr’s authentication mechanism using basic authentication. This will block unknown users from accessing Solr:

```bash
bin/solr auth enable -type basicAuth -prompt true -z localhost:9983 -blockUnknown true
```

This command will prompt you for a username and password:

    Enter username: admin
    Enter password: admin

This will enable basic authentication for Solr, and you’ll be able to add users and configure their roles and permissions.

## 5. Add Users and Set Roles

After authentication is enabled, you can add users and assign roles using Solr’s API.

### Add User

To add a new user `user1` with the password `user@123`:

```bash
curl --user admin:admin \
     -X POST \
     -H 'Content-type:application/json' \
     -d '{
           "set-user": {
             "user1": "user@123"
           }
         }' \
     http://localhost:8983/solr/admin/authentication
```
### Assign Roles to the User

Next, assign roles (reader, writer) to user1:
```bash
curl --user admin:admin \
     -X POST \
     -H 'Content-type:application/json' \
     -d '{
           "set-user-role": {
             "user1": ["reader", "writer"]
           }
         }' \
     http://localhost:8983/solr/admin/authorization
```
### Set Permissions for the User
Now, assign specific permissions to user1 for the collection mycollection. For example, to allow read access to mycollection:
```bash
curl --user admin:admin \
     -X POST \
     -H 'Content-type:application/json' \
     -d '{
           "set-permission": [
             {                  
               "name": "read-mycollection",
               "role": "reader",
               "collection": "mycollection",
               "path": "/select"
             }
           ]
         }' \
     http://localhost:8983/solr/admin/authorization
```
## 6. Change the Password for User1

If you need to change the password for `user1`, you can do so using the following command:

```bash
curl --user admin:admin \
     -X POST \
     -H 'Content-type:application/json' \
     -d '{
           "set-user": {
             "user1": "new_password"
           }
         }' \
     http://localhost:8983/solr/admin/authentication
```
Make sure to replace new_password with the new password you want for user1.

## 7. Create Collections

You can create two collections in Solr, `test` and `mycollection`, using the Solr Admin UI .

## 8. Test Authentication and Authorization

For `user1` (with password `user@123`):

### To check if `user1` has access to the `mycollection`:

```bash
curl --user user1:user@123 "http://localhost:8983/solr/mycollection/select?q=*:*"
```

This should return a response indicating that user1 can access the collection, even though the collection might be empty:
```
{
  "responseHeader": {
    "zkConnected": true,
    "status": 0,
    "QTime": 4,
    "params": {
      "q": "*:*",
      "_forwardedCount": "1"
    }
  },
  "response": {
    "numFound": 0,
    "start": 0,
    "numFoundExact": true,
    "docs": []
  }
}
```
### For Unauthorized Access to Another Collection (e.g., test):

Since user1 is not authorized to access the test collection, it should return an error like:
```bash
curl --user user1:user@123 "http://localhost:8983/solr/test/select?q=*:*"
```
The response should be something like:
```
{
  "error": {
    "msg": "Unauthorized request",
    "code": 403
  }
}
```

This confirms that Solr's authentication and authorization are working properly.

## 9. Conclusion

You have now successfully set up Apache Solr in cloud mode with authentication enabled. You’ve added users, assigned roles and permissions, and verified authentication and authorization using `curl`.

For further steps, you can:

- Add more users and roles.
- Fine-tune permissions for specific collections or operations.
- Explore Solr's features for scaling and fine-tuning your search system.









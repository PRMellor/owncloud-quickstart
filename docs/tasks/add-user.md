---
layout: default
title: Add User
navigation_weight: 4
---

## Create a new user
Add new users using an `OCC` `user:add` command.

> {{ site.note}} You can also manage users from the {{ site.product-name }} [**User Management** page](https://doc.owncloud.org/server/10.0/admin_manual/configuration/user/user_configuration.html), though the command line approach is considered the most convenient for experienced *Administrators*.

### Task
{{ site.task}} **Summary:**
Steps to add a user using the `user:add` command.

1. Create the user with the appropriate attributes.

   For example:

   ```
   sudo -u www-data php occ user:add --display-name="Layla Smith" \
  --group="users" --group="db-admins" --email=layla.smith@example.com layla
  Enter password:
  Confirm password:
  The user "layla" was created successfully
  Display name set to "Layla Smith"
  Email address set to "layla.smith@example.com"
  User "layla" added to group "users"
  User "layla" added to group "db-admins"
  ```

   Full set of attributes:

   - `uid` (username and login name)
   - `display` name (Full Name on the {{ site.product-name }} **User Management** page)
   - `email` address
   - `group` (Any groups that do not exist are created)
   - `login` name
   - `password`

2. Got to the **User Management** page to see the new user.

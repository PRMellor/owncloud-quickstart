---
layout: default
title: Install
navigation_weight: 2
---
### Install and configure an {{ site.product-name }} server
The steps assume a command line installation of the {{ site.product-name }} tarball using `OCC`.

You must run `OCC` as [a HTTP user](https://doc.owncloud.org/server/10.0/admin_manual/configuration/server/occ_command.html#http-user-label){:target="_blank"}.
> {{ site.note}} [More on using OCC commands](https://doc.owncloud.org/server/10.0/admin_manual/configuration/server/occ_apps_command.html?highlight=occ){:target="_blank"}

#### Steps

1. Check you have the [prerequisites](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#prerequisites-label){:target="_blank"} required.
2. [Download and unpack the {{ site.product-name }} tarball into a directory](https://owncloud.org/download/#owncloud-server-tar-ball){:target="_blank"}
3. Set your webserver as the owner of the {{ site.product-name }} directory:

    ```
    $ sudo chown -R www-data:www-data /var/www/owncloud/
    ```

4. Install {{ site.product-name }} from the `OCC` command line using:

    ```
    # Assuming you’ve unpacked the source to /var/www/owncloud/
    $ cd /var/www/owncloud/
    $ sudo -u www-data php occ maintenance:install \
       --database "mysql" --database-name "owncloud" \
       --database-user "root" --database-pass "password" \
       --admin-user "admin" --admin-pass "password"
   ```

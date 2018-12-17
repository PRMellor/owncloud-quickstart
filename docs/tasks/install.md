---
layout: default
title: Install
navigation_weight: 2
---

## Install and configure an {{ site.product-name }} server
Here we describe a command line installation of an {{ site.product-name }} tarball on a Linux server using out-of-box defaults.

> {{ site.product-name }} The steps assume familiarity with the **{{ site.product-name }} console** (`OCC`). If you prefer not to install from the command line, other options are [available](https://doc.owncloud.org/server/10.0/admin_manual/installation/deployment_considerations.html).

> {{ site.note}} Other [installation options](https://doc.owncloud.org/server/10.0/admin_manual/installation/) are available.

For the installation, you must run `OCC` as [a HTTP user](https://doc.owncloud.org/server/10.0/admin_manual/configuration/server/occ_command.html#http-user-label).

> {{ site.note}} More information on [using OCC commands](https://doc.owncloud.org/server/10.0/admin_manual/configuration/server/occ_apps_command.html?highlight=occ)

### Task
{{ site.task}} **Summary:**
Steps to install and configure an Owncloud server from the command line using `OCC`.

1. Check your server has the [prerequisites](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#prerequisites-label) required.

2. [Download and unpack the {{ site.product-name }} tarball](https://owncloud.org/download/#owncloud-server-tar-ball) into a suitable directory.
3. Set your webserver as the owner of the `{{ site.product-name }}` directory:

    ```text
    $ sudo chown -R www-data:www-data /var/www/owncloud/
    ```

4. Install {{ site.product-name }} from the `OCC` command line.

    For example, assuming you’ve unpacked the source to `/var/www/owncloud/`, the command is:

    ```text
    $ cd /var/www/owncloud/
    $ sudo -u www-data php occ maintenance:install \
       --database "mysql" --database-name "owncloud" \
       --database-user "root" --database-pass "password" \
       --admin-user "admin" --admin-pass "password"
   ```

5. Set [strong permissions](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#strong-perms-label).

    > {{ site.warning}} This step is **very** important and should be performed without delay as it protects your installation

    Copy and run the following script, replacing the paths and variables according to your `{{ site.product-name }}` directory. More details on configuring the script can be found [here](https://doc.owncloud.org/server/10.0/admin_manual/installation/installation_wizard.html#post-installation-steps-label).

    ```text
    #!/bin/bash

    ocpath='/var/www/owncloud'
    ocdata='/var/www/owncloud/data'
    ocapps_external='/var/www/owncloud/apps-external'
    oldocpath='/var/www/owncloud.old'
    linkdata="/var/mylinks/data"
    linkapps-external="/var/mylinks/apps-external"
    htuser='www-data'
    htgroup='www-data'
    rootuser='root'

    # Because the data directory can be huge or on external storage, an automatic chmod/chown can take a while.
    # Therefore this directory can be treated differently.
    # If you have already created an external data and apps-external directory which you want to link,
    # set the paths above accordingly. This script can link and set the proper rights and permissions
    # depending what you enter when running the script.
    # You have to run this script twice, one time to prepare installation and one time post installation

    # Example input
    # New install using mkdir:     n/n/n (create missing directories, setup permissions and ownership)
    # Upgrade using mkdir:         n/n/n (you move/replace data, apps-external and config.php manually, set setup permissions and ownership)
    # New install using links:     y/y/n (link existing directories, setup permissions and ownership)
    # Upgrade using links:         y/n/y (link existing directories, copy config.php, permissions and ownership are already ok)
    # Post installation/upgrade:   either n/n/n or y/y/n
    # Reset all perm & own:        either n/n/n or y/y/n

    echo
    read -p "Do you want to use ln instead of mkdir for creating directories (y/N)? " -r -e answer
    if echo "$answer" | grep -iq "^y"; then
      uselinks="y"
    else
      uselinks="n"
    fi

    read -p "Do you also want to chmod/chown these links (y/N)? " -r -e answer
    if echo "$answer" | grep -iq "^y"; then
      chmdir="y"
    else
      chmdir="n"
    fi

    read -p "If you upgrade, do you want to copy an existing config.php file (y/N)? " -r -e answer
    if echo "$answer" | grep -iq "^y"; then
      upgrdcfg="y"
    else
      upgrdcfg="n"
    fi

    printf "\nCreating or linking possible missing directories \n"
    mkdir -p $ocpath/updater
    # check if directory creation is possible and create if ok
    if [ "$uselinks" = "n" ]; then
      if [ -L ${ocdata} ]; then
        echo "Symlink for $ocdata found but mkdir requested. Exiting."
        echo
        exit
      else
        echo "mkdir $ocdata"
        echo
        mkdir -p $ocdata
      fi
      if [ -L ${ocapps_external} ]; then
        echo "Symlink for $ocapps_external found but mkdir requested. Exiting."
        echo
        exit
      else
        printf "mkdir $ocapps_external \n"
        mkdir -p $ocapps_external
      fi
    else
      if [ -d ${ocdata} ]; then
        echo "Directory for $ocdata found but link requested. Exiting."
        echo
        exit
      else
        printf "ln $ocdata \n"
        ln -sfn $linkdata $ocdata
      fi
      if [ -d ${ocapps_external} ]; then
        echo "Directory for $ocapps_external found but link requested. Exiting."
        echo
        exit
      else
        printf "ln $ocapps_external \n"
        ln -sfn $linkapps-external $ocapps_external
      fi
    fi

    # Copy if requested an existing config.php
    if [ "$upgrdcfg" = "y" ]; then
      if [ -f ${oldocpath}/config/config.php ]; then
        printf "\nCopy existing config.php file \n"
        cp ${oldocpath}/config/config.php ${ocpath}/config/config.php
      else
        printf "Skipping copy config.php, not found: ${oldocpath}/config/config.php \n"
      fi
    fi

    printf "\nchmod files and directories excluding data and apps-external directory \n"
    find -L ${ocpath} -path ${ocdata} -prune -o -path ${ocapps_external} -prune -o -type f -print0 | xargs -0 chmod 0640
    find -L ${ocpath} -path ${ocdata} -prune -o -path ${ocapps_external} -prune -o -type d -print0 | xargs -0 chmod 0750

    # no error messages on empty directories
    if [ "$chmdir" = "n" ] && [ "$uselinks" = "n" ]; then
      printf "chmod data and apps-external directory (mkdir) \n"
      if [ -n "$(ls -A $ocdata)" ]; then
        find ${ocdata}/ -type f -print0 | xargs -0 chmod 0640
      fi
      find ${ocdata}/ -type d -print0 | xargs -0 chmod 0750
      if [ -n "$(ls -A $ocapps_external)" ]; then
        find ${ocapps_external}/ -type f -print0 | xargs -0 chmod 0640
      fi
      find ${ocapps_external}/ -type d -print0 | xargs -0 chmod 0750
    fi

    if [ "$chmdir" = "y" ] && [ "$uselinks" = "y" ]; then
      printf "chmod data and apps-external directory (linked) \n"
      if [ -n "$(ls -A $ocdata)" ]; then
        find -L ${ocdata}/ -type f -print0 | xargs -0 chmod 0640
      fi
      find -L ${ocdata}/ -type d -print0 | xargs -0 chmod 0750
      if [ -n "$(ls -A $ocapps_external)" ]; then
        find -L ${ocapps_external}/ -type f -print0 | xargs -0 chmod 0640
      fi
      find -L ${ocapps_external}/ -type d -print0 | xargs -0 chmod 0750
    fi

    printf "\nchown files and directories excluding data and apps-external directory \n"
    find  -L $ocpath  -path ${ocdata} -prune -o -path ${ocapps_external} -prune -o -type d -print0 | xargs -0 chown ${rootuser}:${htgroup}
    find  -L $ocpath  -path ${ocdata} -prune -o -path ${ocapps_external} -prune -o -type f -print0 | xargs -0 chown ${rootuser}:${htgroup}

    # do only if the directories are present
    if [ -d ${ocpath}/apps/ ]; then
      printf "chown apps directory \n"
      chown -R ${htuser}:${htgroup} ${ocpath}/apps/
    fi
    if [ -d ${ocpath}/config/ ]; then
      printf "chown config directory \n"
      chown -R ${htuser}:${htgroup} ${ocpath}/config/
    fi
    if [ -d ${ocpath}/updater/ ]; then
      printf "chown updater directory \n"
      chown -R ${htuser}:${htgroup} ${ocpath}/updater
    fi

    if [ "$chmdir" = "n" ] && [ "$uselinks" = "n" ]; then
      printf "chown data and apps-external directories (mkdir) \n"
      chown -R ${htuser}:${htgroup} ${ocapps_external}/
      chown -R ${htuser}:${htgroup} ${ocdata}/
    fi
    if [ "$chmdir" = "y" ] && [ "$uselinks" = "y" ]; then
      printf "chown data and apps-external directories (linked) \n"
      chown -R ${htuser}:${htgroup} ${ocapps_external}/
      chown -R ${htuser}:${htgroup} ${ocdata}/
    fi

    printf "\nchmod occ command to make it executable \n"
    if [ -f ${ocpath}/occ ]; then
      chmod +x ${ocpath}/occ
    fi

    printf "chmod/chown .htaccess \n"
    if [ -f ${ocpath}/.htaccess ]; then
      chmod 0644 ${ocpath}/.htaccess
      chown ${rootuser}:${htgroup} ${ocpath}/.htaccess
    fi
    if [ -f ${ocdata}/.htaccess ];then
      chmod 0644 ${ocdata}/.htaccess
      chown ${rootuser}:${htgroup} ${ocdata}/.htaccess
    fi
    echo
    ```

#### What next?

- [Connect users to the {{ site.product-name }} server using the IP address and port 8080]({{ site.baseurl }}{% link tasks/connect.md %})
- [Add new user accounts]({{ site.baseurl }}{% link tasks/add-user.md %})

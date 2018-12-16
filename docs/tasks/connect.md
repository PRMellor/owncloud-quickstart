---
layout: default
title: Connect
navigation_weight: 3
---

## Specify a connection using an IP and port 8080
Users are allowed to log into {{ site.product-name }} server only when their browser is pointing to a whitelisted URL in the `trusted_domains` setting of the `config.php` file in `/var/www/owncloud/config/`.

Standard configuration:

```
'trusted_domains' => [
   0 => 'localhost',
   1 => 'server1.example.com',
   2 => '192.168.1.50',
],
```

### Task
{{ site.task}} **Summary:**
Steps to connect a user through a specific URL.

1. Access the `config.php` file and change the `trusted_domains` setting.

   For example, to connect users to the {{ site.product-name }} server using the server's IP address and port 8080, the port is added with the address.

   ```
   'trusted_domains' => [
      0 => 'localhost',
      1 => 'server1.example.com',
      2 => '192.168.1.50',
      3 => '192.168.1.50:8080'
   ],
   ```

2. Check the connection from a browser.   

   You can look in `owncloud.log` logfile in `/var/www/owncloud/data` for more information.

---
author:
  - Mo D Jabeen
date: 2023-10-01
title: Apache
---

# General

-   Need to set firewall to accept WWW
-   Add mods using a2enmod command
-   enable sites in sites available using soft link (ln -s)

# Proxy

Need to add proxy mods to enable proxy

```bash
<VirtualHost *:80>

ProxyRequests Off
ProxyPreserveHost On
ProxyVia Full

<Location /directory>
ProxyPass http://127.0.0.1:2021
ProxyPassReverse http://127.0.0.1:2021
</Location>

ErrorLog ${APACHE_LOG_DIR}/divination_proxy_error.log
CustomLog ${APACHE_LOG_DIR}/divination_proxy_access.log combined

</VirtualHost>
```


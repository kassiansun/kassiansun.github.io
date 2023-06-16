---
title: "Set Up OpenLDAP and phpldapadmin on Ubuntu 22.04"
date: 2023-06-16T15:49:24+08:00
---

## Install LDAP

Before install ldap, set-up a valid FQDN for your hostname:

- Edit /etc/hostname, for example `void.kassiansun.com`
- Restart the host

If you've installed ldap before, purge them all:

```bash
sudo apt-get remove --purge slapd ldap-utils -y
```

Now we can install the ldap packages:

```bash
sudo apt-get install slapd ldap-utils -y
```

During the installation, it will prompt to set the default password.

Test that you now have a valid LDAP tree:

```bash
# Output:
# dn:
# namingContexts: dc=kassiansun,dc=com
ldapsearch -H ldap://localhost -x -LLL -s base -b "" namingContexts
```

## Clean-Up Old apache2 and php installation

```bash
sudo apt-get remove --purge apache2 phpldapadmin php*
```

If you're not using apache or php on your machine, clean them all so we can get started from the scratch.

## Install phpldapadmin

Add php repository to install php 7.4:

```bash
sudo add-apt-repository ppa:ondrej/php
```

Install phpldapamin with:

```bash
sudo apt-get install php7.4 apache2 phpldapadmin php7.4-common php7.4-ldap php7.4-xml
```

The key point is that phpldapadmin is not compatible with php 8.x, but Ubuntu will install 8.x by default. To fix it, we need to overwrite the default package versions.

## Use phpldapadmin

Open http://localhost/phpldapadmin/ in your browser, now you should be able to login with the ldap password we've set before.

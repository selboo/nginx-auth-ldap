# LDAP Authentication module for nginx
LDAP module for nginx which supports authentication against multiple LDAP servers.

# How to install

## FreeBSD

```bash
cd /usr/ports/www/nginx && make config install clean
```

Check HTTP_AUTH_LDAP options


```
[*] HTTP_AUTH_LDAP        3rd party http_auth_ldap module
```

## Linux

```bash
cd ~ && git clone https://github.com/selboo/nginx-auth-ldap.git
```

in nginx source folder

```bash
./configure --add-module=path_to_http_auth_ldap_module
make install
```

# Example configuration
Define list of your LDAP servers with required user/group requirements:

```bash

    load_module modules/ngx_http_auth_ldap_module.so;

    http {
      ldap_server test1 {
        url ldap://192.168.0.1:3268/DC=test,DC=local?sAMAccountName?sub?(objectClass=person);
        binddn "TEST\\LDAPUSER";
        binddn_passwd LDAPPASSWORD;
        group_attribute uniquemember;
        group_attribute_is_dn on;
        require valid_user;
      }

      ldap_server test2 {
        url ldap://192.168.0.2:3268/DC=test,DC=local?sAMAccountName?sub?(objectClass=person);
        binddn "TEST\\LDAPUSER";
        binddn_passwd LDAPPASSWORD;
        group_attribute uniquemember;
        group_attribute_is_dn on;
        require valid_user;
      }

      auth_ldap_cache_enabled on;
      auth_ldap_cache_expiration_time 60000; # 60s
      auth_ldap_cache_size 1024;
    }
```

And add required servers in correct order into your location/server directive:
```bash
    server {
        listen       8000;
        server_name  localhost;

        auth_ldap "Forbidden";
        auth_ldap_servers test1;
		auth_ldap_servers test2;

        location / {
            root   html;
            index  index.html index.htm;
        }

    }
```

# Available config parameters

## url
expected value: string

Available URL schemes: ldap://, ldaps://

## binddn
expected value: string

## binddn_passwd
expected value: string

## group_attribute
expected value: string

## group_attribute_is_dn
expected value: on or off, default off

## require
expected value: valid_user, user, group

## satisfy
expected value: all, any

## max_down_retries
expected value: a number, default 0

Retry count for attempting to reconnect to an LDAP server if it is considered
"DOWN".  This may happen if a KEEP-ALIVE connection to an LDAP server times 
out or is terminated by the server end after some amount of time.  

This can usually help with the following error:

```
http_auth_ldap: ldap_result() failed (-1: Can't contact LDAP server)
```

## connections
expected value: a number greater than 0

## referral
expected value: on, off

LDAP library default is on. This option disables usage of referral messages from
LDAP server. Usefull for authenticating against read only AD server without access
to read write.


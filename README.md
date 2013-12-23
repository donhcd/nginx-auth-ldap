# LDAP Authentication module for nginx
LDAP module for nginx which supports:
* authentication against multiple LDAP servers
* per location authentication requirements without defining the same server again for different authorization requirement
* case sensitive user accounts, just like all other web servers have
* LDAP-users can be specified with UID or DN

# How to install

## Linux

```bash
cd ~ && git clone https://github.com/HQJaTu/nginx-auth-ldap.git
```

in nginx source folder

```bash
./configure --add-module=path_to_http_auth_ldap_module
make install
```

# Example configuration
Define list of your LDAP servers with required user/group requirements:

```bash
    ldap_server test1 {
      url ldap://192.168.0.1:3268/DC=test,DC=local?sAMAccountName?sub?(objectClass=person);
      binddn "TEST\\LDAPUSER";
      binddn_passwd LDAPPASSWORD;
      group_attribute uniquemember;
      group_attribute_is_dn on;
    }

    ldap_server test2 {
      url ldap://192.168.0.2:3268/DC=test,DC=local?sAMAccountName?sub?(objectClass=person);
      binddn "TEST\\LDAPUSER";
      binddn_passwd LDAPPASSWORD;
      group_attribute uniquemember;
      group_attribute_is_dn on;
    }
```

And add required servers in correct order into your location/server directive:
```bash
    server {
        listen       8000;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }

        location /any-authenticated-user {
            root   html;

            auth_ldap "Forbidden";
            auth_ldap_servers test1;
            auth_ldap_servers test2;
            require valid_user;
        }

        location /specific-user-DN {
            root   html;

            auth_ldap "Forbidden";
            auth_ldap_servers test1;
            auth_ldap_servers test2;
            auth_ldap_require group 'cn=AllowedUser,ou=Users,DC=test,DC=local';
        }

        location /specific-user-UID {
            root   html;

            auth_ldap "Forbidden";
            auth_ldap_servers test1;
            auth_ldap_servers test2;
            auth_ldap_require group 'alloweduser';
        }

        location /specific-group {
            root   html;

            auth_ldap "Forbidden";
            auth_ldap_servers test1;
            auth_ldap_servers test2;
            auth_ldap_require group 'cn=SecretAllowedGroup,ou=Group,DC=test,DC=local';
        }

}
```

Kerberos Backend configuration
==============================

# KDC configuration

```
[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
        NYUU.PRIV = {
                acl_file = /var/lib/krb5kdc/kadm5.acl
                admin_keytab = /var/lib/krb5kdc/kadm5.keytab
        }

[dbdefaults]
        ldap_kerberos_container_dn = cn=krbContainer,ou=services,dc=nyuu,dc=priv

[dbmodules]
        openldap_ldapconf = {
                db_library = kldap
                ldap_kdc_dn = "cn=kdc,ou=services,dc=nyuu,dc=priv"

                # this object needs to have read rights on
                # the realm container, principal container and realm sub-trees
                ldap_kadmind_dn = "cn=kdc,ou=services,dc=nyuu,dc=priv"

                # this object needs to have read and write rights on
                # the realm container, principal container and realm sub-trees
                ldap_service_password_file = /var/lib/krb5kdc/service.keyfile
                ldap_servers = ldapi:///
                ldap_conns_per_server = 5
        }

```

# Initialization

Run the following :

    kdb5_ldap_util -D cn=kdc,ou=services,dc=nyuu,dc=priv create -subtrees ou=people,dc=nyuu,dc=priv -r NYUU.PRIV -s -H ldapi:///

Warning: subtrees takes the DNs under which the users will be !

Then we'll pass to the KDC the password of the DN used to bind to LDAP :

    kdb5_ldap_util -D cn=kdc,ou=services,dc=nyuu,dc=priv stashsrvpw -f /var/lib/krb5kdc/service.keyfile cn=kdc,ou=services,dc=nyuu,dc=priv

The KDC is now ready, start it then run `kadmin.local` as root.

    addprinc -x dn="cn=jraffre,ou=people,dc=nyuu,dc=priv" jraffre

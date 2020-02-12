# README

## How to setup

* from a client, run `kadmin -p admin/admin` to logon to krb admin console on krb5-server. The admin password is 'admin' by default
* create 'sokoide' user in REALM.SOKOIDE.COM by `addprinc sokoide`
* create 'HTTP/$host' principal in the realm by `addprinc -randkey HTTP/sokoide-elastic` (or any HTTP/$hostname)
* export the HTTP/$host into a keytab file as below

```bash
ktadd -k /var/tmp/http.keytab HTTP/sokoide-elastic
quit
# quit the kadmin
mv /var/tmp/http.keytab /path/to/od_elastic/keytabs
```

* change `opendistro_security.kerberos.acceptor_principal` in elasticsearch.yml

## How to run

* kinit sokoide -> get TGT
* docker-compose up
* test connection

```
# with -v
curl -k -s -X GET --negotiate -u: https://sokoide-elastic:9200/ -v

# without v
curl -k -s -X GET --negotiate -u: https://sokoide-elastic:9200/
Open Distro Security not initialized.
```


## To configure roles (again)

```sh
# generate pem files under cert
# -> https://opendistro.github.io/for-elasticsearch-docs/docs/security-configuration/generate-certificates/
cd /path/to/od_elastic/cert

# Make sure to update opendistro_security.authcz.admin_dn in elasticsearch.yml
# to match the following ADMIN

# Root CA
openssl genrsa -out root-ca-key.pem 2048
openssl req -new -x509 -sha256 -key root-ca-key.pem -out root-ca.pem -days 3650 -subj "/C=JP/ST=Tokyo/L=Tokyo/O=Sokoide Inc/OU=IT/CN=sokoide-elastic"

# Admin cert
openssl genrsa -out admin-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
openssl req -new -key admin-key.pem -out admin.csr -days 3650 -subj "/C=JP/ST=Tokyo/L=Tokyo/O=Sokoide Inc/OU=IT/CN=ADMIN"
openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem

# Node cert
openssl genrsa -out node-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in node-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node-key.pem
openssl req -new -key node-key.pem -out node.csr -days 3650 -subj "/C=JP/ST=Tokyo/L=Tokyo/O=Sokoide Inc/OU=IT/CN=node1.sokoide-elastic"
openssl x509 -req -in node.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node.pem
# Cleanup
rm admin-key-temp.pem
rm admin.csr
rm node-key-temp.pem
rm node.csr

# logon to the container
docker exec -it odfe-node1 sh

cd /usr/share/elasticsearch/plugins/opendistro_security

chmod +x tools/securityadmin.sh
tools/securityadmin.sh -nhnv -cn docker-cluster -cacert /usr/share/elasticsearch/config/root-ca.pem -cd ./securityconfig/ -cert /usr/share/elasticsearch/config/admin.pem -key /usr/share/elasticsearch/config/admin-key.pem

# if you see below, all done
Clustername: docker-cluster
Clusterstate: GREEN
Number of nodes: 1
Number of data nodes: 1
.opendistro_security index already exists, so we do not need to create one.
Populate config from /usr/share/elasticsearch/plugins/opendistro_security/securityconfig
Will update '_doc/config' with ./securityconfig/config.yml
   SUCC: Configuration for 'config' created or updated
Will update '_doc/roles' with ./securityconfig/roles.yml
   SUCC: Configuration for 'roles' created or updated
Will update '_doc/rolesmapping' with ./securityconfig/roles_mapping.yml
   SUCC: Configuration for 'rolesmapping' created or updated
Will update '_doc/internalusers' with ./securityconfig/internal_users.yml
   SUCC: Configuration for 'internalusers' created or updated
Will update '_doc/actiongroups' with ./securityconfig/action_groups.yml
   SUCC: Configuration for 'actiongroups' created or updated
Will update '_doc/tenants' with ./securityconfig/tenants.yml
   SUCC: Configuration for 'tenants' created or updated
Done with success
```

## If something goes wrong

* stop the containers, `docker volume rm od_elastic-odfe-data1` and retry

# README

## How to setup

* create 'scott' user in SOKOIDE.REALM.COM
* create 'HTTP/ip-172...' principal in the realm
* export thhe HTTP/ip-172 into a keytab file


## How to run

* kinit scott -> get TGT
* docker-compose up
* test connection

```
curl -k -s -X GET --negotiate -u: https://ip-172-18-40-101:9200/ -v
```


## To configure role again

```sh
docker exec -it odfe-node1 sh

cd /usr/share/elasticsearch/plugins/opendistro_security

tools/securityadmin.sh -nhnv -cn odfe-cluster -cacert /usr/share/elasticsearch/config/root-ca.pem -cd ./securityconfig/ -cert /usr/share/elasticsearch/config/admin.pem -key /usr/share/elasticsearch/config/admin-key.pem
```

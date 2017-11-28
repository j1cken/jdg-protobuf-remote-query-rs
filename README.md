# Credits

Most of this code is derived from the JDG Quickstart _remote-query_. I've just RESTified it.

# How-To

## Install [OpenShift CDK](https://developers.redhat.com/products/cdk/overview/)

## create JDG 7.1 image stream
```
oc import-image -n openshift openshift/jboss-datagrid71-openshift --from=registry.access.redhat.com/jboss-datagrid-7/datagrid71-openshift --confirm
```

## create JDG 7.1 template
```
oc create -n openshift -f jdg71.template
```

## change context
```
oc project myproject
```

## create JDG 7.1 deployment
```
oc new-app datagrid71-basic -p CACHE_NAMES=default,preferences,addressbook -p INFINISPAN_CONNECTORS=hotrod,rest -p USERNAME=admin -p PASSWORD=admin123 -p APPLICATION_NAME=protobuf
```

## set env variables
```
oc env dc protobuf HOTROD_AUTHENTICATION=true CONTAINER_SECURITY_ROLES=admin=ALL CONTAINER_SECURITY_ROLE_MAPPER=identity-role-mapper ADDRESSBOOK_CACHE_SECURITY_AUTHORIZATION_ROLES=admin USERNAME=admin PASSWORD=admin123 ADMIN_GROUP=REST,admin,___schema_manager
```

## create and deploy the REST protobuf client
```
oc new-app openshift/jboss-eap70-openshift~https://github.com/j1cken/jdg-protobuf-remote-query-rs.git
```

## Curl Tests
```
curl -i -XPUT 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/createPerson?id=1&name=torben'
curl -i 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/queryPerson?pattern=tor%25'
```

> **Hint** %25 is the encoded Wildcard pattern '%' ... so '%25or%25' (encoded equivalent of '%or%') will match both 'Torben' and 'Thorsten'

```
╭─torben@f27 ~/development/jboss-datagrid-7.1.1-quickstarts/remote-query  ‹master›
╰─$ curl -i -XPUT 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/createPerson?id=1&name=torben'
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 29
Date: Tue, 28 Nov 2017 20:47:28 GMT
Set-Cookie: 1685d736bc46b8fba481fc8c85e629e3=29ca7fa37dd4c4f198fa9a852858895a; path=/; HttpOnly
Cache-control: private

{ 'id': '1', 'name':'torben'}%
╭─torben@f27 ~/development/jboss-datagrid-7.1.1-quickstarts/remote-query  ‹master›
╰─$ curl -i -XPUT 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/createPerson?id=2&name=thorsten'
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 31
Date: Tue, 28 Nov 2017 20:47:35 GMT
Set-Cookie: 1685d736bc46b8fba481fc8c85e629e3=29ca7fa37dd4c4f198fa9a852858895a; path=/; HttpOnly
Cache-control: private

{ 'id': '2', 'name':'thorsten'}%
╭─torben@f27 ~/development/jboss-datagrid-7.1.1-quickstarts/remote-query  ‹master›
╰─$ curl -i 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/queryPerson?pattern=thorsten'
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 54
Date: Tue, 28 Nov 2017 20:47:45 GMT
Set-Cookie: 1685d736bc46b8fba481fc8c85e629e3=29ca7fa37dd4c4f198fa9a852858895a; path=/; HttpOnly
Cache-control: private

Person{id=2, name='thorsten', email='null', phones=[]}%
╭─torben@f27 ~/development/jboss-datagrid-7.1.1-quickstarts/remote-query  ‹master›
╰─$ curl -i 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/queryPerson?pattern=torben'
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 52
Date: Tue, 28 Nov 2017 20:47:51 GMT
Set-Cookie: 1685d736bc46b8fba481fc8c85e629e3=29ca7fa37dd4c4f198fa9a852858895a; path=/; HttpOnly
Cache-control: private

Person{id=1, name='torben', email='null', phones=[]}%
╭─torben@f27 ~/development/jboss-datagrid-7.1.1-quickstarts/remote-query  ‹master›
╰─$ curl -i 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/queryPerson?pattern=%25or%25'
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 106
Date: Tue, 28 Nov 2017 20:52:34 GMT
Set-Cookie: 1685d736bc46b8fba481fc8c85e629e3=29ca7fa37dd4c4f198fa9a852858895a; path=/; HttpOnly
Cache-control: private

Person{id=2, name='thorsten', email='null', phones=[]}Person{id=1, name='torben', email='null', phones=[]}%
╭─torben@f27 ~/development/jboss-datagrid-7.1.1-quickstarts/remote-query  ‹master›
╰─$ curl -i 'http://jdg-protobuf-remote-query-rs-myproject.atmy.localopenshift.cloud/jboss-remote-query-quickstart/rest/protobuf/queryPerson?pattern=tor%25'
HTTP/1.1 200 OK
X-Powered-By: Undertow/1
Server: JBoss-EAP/7
Content-Type: application/json
Content-Length: 52
Date: Tue, 28 Nov 2017 20:52:56 GMT
Set-Cookie: 1685d736bc46b8fba481fc8c85e629e3=29ca7fa37dd4c4f198fa9a852858895a; path=/; HttpOnly
Cache-control: private

Person{id=1, name='torben', email='null', phones=[]}%
╭─torben@f27 ~/development/jboss-datagrid-7.1.1-quickstarts/remote-query  ‹master›
╰─$
```

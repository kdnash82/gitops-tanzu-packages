tanzu secret registry add tanzu-image-registry \
--server harbor.<FQDN> \
--username  \
--password  \
--yes --namespace hello-world

Bind to Service
Grab secrets for <service-name>-app-user-db-secrets
- username
- password
- hosts
- database
- port


cf create-user-provided-service <name-of-cf-service> \
-p '{"uri":"mysql://<username>:<password>@<hosts>:<port>/<database>"}'

cf bind-service <app-name> <service-name> 
cf restage <app-name>

Make change to spring-music via GUI/URL and save

- kubectl exec -it <name-of-mysql-pod> -- /bin/bash


mysql -u mysqlappuser -h localhost -p <RETURN and enter password when prompted>

You're now logged into mysql...
show databases;
USE <database-name>;
show tables;
select * from <table-name>;

See the change!
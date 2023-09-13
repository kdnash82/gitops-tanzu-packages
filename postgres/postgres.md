tanzu package repository add tanzu-data-services-repository \
    --url cs-s-harbor.avmc.army.mil/packages-for-vmware-tanzu-data-services/tds-packages:1.5.0 \
    --namespace tanzu-package-repo-global 

tanzu package install postgres \
    -p postgres-operator.sql.tanzu.vmware.com \
    --version 1.7.1 \
    --namespace tanzu-package-repo-global 

watch kubectl get all -n tanzu-package-repo-global  --selector app=postgres-operator 

apiVersion: sql.tanzu.vmware.com/v1
kind: Postgres
metadata:
  name: pg-mypostgres
  namespace: postgres-databases
spec:
  storageClassName: standard
  storageSize: 10G
  highAvailability:
    enabled: false

dbname: pg-mypostgres
username: pgappuser
password: 7H452Owd07z1ecafchpzqAE5t9q8bk
host: pg-mypostgres.dotnet-ns
port: 5432

psql -h pg-mypostgres.dotnet-ns -p 5432 -d pg-mypostgres -U pgappuser

pg-mypostgres=> \d
               List of relations
 Schema |       Name       | Type  |   Owner   
--------+------------------+-------+-----------
 public | CustomerProfiles | table | pgappuser
(1 row)

pg-mypostgres=> \d "CustomerProfiles"
          Table "public.CustomerProfiles"
  Column   | Type | Collation | Nullable | Default 
-----------+------+-----------+----------+---------
 Id        | text |           | not null | 
 FirstName | text |           |          | 
 LastName  | text |           |          | 
 Email     | text |           |          | 
Indexes:
    "PK_CustomerProfiles" PRIMARY KEY, btree ("Id")

SELECT * FROM "CustomerProfiles";
                  Id                  | FirstName | LastName |    Email     
--------------------------------------+-----------+----------+--------------
 6857b63d-b178-44d3-8698-6ebf66966692 | Jane      | Doe      | jane@doe.com
 796f3b54-ef1a-4267-906c-6fdd5b12ea84 | John      | Doe      | john@doe.com
(2 rows)

pg-mypostgres=> INSERT INTO "CustomerProfiles" ("Id", "FirstName", "LastName", "Email") VALUES ('101', 'TAP', 'VMware', 'tap@vmware.com');
INSERT 0 1

tanzu services resource-claim create helloworld-angular --resource-name pg-mypostgres --resource-kind Postgres --resource-api-version sql.tanzu.vmware.com/v1 --resource-namespace dotnet-ns

tanzu apps workload create helloworld \
  --git-repo https://gitlab.k8s.avmc.army.mil/tanzu-team/helloworld-angular.git \
  --git-branch main \
  --type web \
  --label app.kubernetes.io/part-of=helloworld \
  --annotation autoscaling.knative.dev/minScale=1 \
  --service-ref="services.apps.tanzu.vmware.com/v1alpha1:ResourceClaim:helloworld-angular" \
  --env SPRING_PROFILES_ACTIVE=postgres
  --param scanning_source_policy="lax-scan-policy"
  --param scanning_image_policy="lax-scan-policy"



tanzu apps workload update helloworld \
  --git-repo https://gitlab.k8s.avmc.army.mil/tanzu-team/helloworld-angular.git \
  --git-branch main \
  --type web \
  --label app.kubernetes.io/part-of=helloworld \
  --annotation autoscaling.knative.dev/minScale=1 \
  --service-ref="db=services.apps.tanzu.vmware.com/v1alpha1:ResourceClaim:helloworld-angular" \
  --env SPRING_PROFILES_ACTIVE=postgres \
  --param scanning_source_policy="lax-scan-policy" \
  --param scanning_image_policy="lax-scan-policy" 


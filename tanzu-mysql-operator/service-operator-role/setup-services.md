## Roles
### There is a clear separation of concerns across the various user roles.

The life cycle of workloads is determined by application developers.

The life cycle of resource claims is determined by application operators.

The life cycle of service instances is determined by service operators.

The life cycle of service bindings is implicitly tied to the life cycle of workloads.

## Service Operator Role - Setup Services
https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.2/tap/GUID-getting-started-set-up-services.html


## Identify Permissions for Application with Cluster Role
`kubectl apply -f <path-to>/tanzu-mysql-operator/service-operator-role/resource-claims-mysql.yaml`

## Verify Cluster Role
`kubectl get clusterrole resource-claims-mysql`

## Provide a list of services for Application Operators/Developers
`kubectl apply -f <path-to>/tanzu-mysql-operator/service-operator-role/mysql-clusterinstancesclass.yaml`

## Verify Service List
`tanzu services class list`

## Create MySql Service Instance
`kubectl apply -f <path-to>/tanzu-mysql-operator/service-operator-role/mysql-1-service-instance.yaml`

## Verify MySql Service Instance Created
`kubectl get po -n service-instances`


## Create Claim Policy for Application Operators/Developers
`kubectl apply -f <path-to>/tanzu-mysql-operator/service-operator-role/mysql-claim-policy.yaml`

## Verify Claim Policies
`kubectl get resourceclaimpolicy -A`
or
`kubectl get resourceclaimpolicy -n <namespace>`

- Note: resource-namespace is where the resource-name instance resides.  namespace is the namespace of the application.
```
tanzu service claim create mysql-1 \
  --resource-name mysql-1 \
  --resource-namespace service-instances \
  --resource-kind MySQL \
  --resource-api-version with.sql.tanzu.vmware.com/v1
  --namespace spring-music
```
### Verify Claim
`tanzu services claims get mysql-1 --namespace <application-namespace>`

## Application Operator Role - Consume Services
https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.2/tap/GUID-getting-started-consume-services.html

### List available classes
`tanzu services class list`

### List Available Claims
`tanzu services claims list -A -o wide`

Note: Provide the value of `CLAIM REF` to Application-Developer


## Application Developer Role - Consume Services

### Create the Workload with Services
```
tanzu apps workload create pet-clinic-with-mysql \
  --git-repo https://<GIT_URL>/<ORG>/<PLATFORM>/spring-petclinic.git \
  --git-branch main \
  --type web \
  --label app.kubernetes.io/part-of=pet-clinic \
  --annotation autoscaling.knative.dev/minScale=1 \
  --service-ref="mysql-1=services.apps.tanzu.vmware.com/v1alpha1:ResourceClaim:mysql-1" \
  --env SPRING_PROFILES_ACTIVE=mysql
```
Note:  add --dry-run flag to the end of this command see the output

### Update the Workload with Services
```
tanzu apps workload update pet-clinic-with-mysql \
  --git-repo https://<GIT_URL>/<ORG>/<PLATFORM>/spring-petclinic.git \
  --git-branch main \
  --type web \
  --label app.kubernetes.io/part-of=pet-clinic \
  --annotation autoscaling.knative.dev/minScale=1 \
  --service-ref="mysql-1=services.apps.tanzu.vmware.com/v1alpha1:ResourceClaim:mysql-1" \
  --env SPRING_PROFILES_ACTIVE=mysql
```

## Troubleshooting
### Logs for service-binding
`kubectl logs -n service-bindings -l role=manager`

## Other Use Cases
https://docs.vmware.com/en/Services-Toolkit-for-VMware-Tanzu-Application-Platform/0.7/svc-tlk/GUID-usecases-consuming_aws_rds_with_crossplane.html#claim-the-rds-postgresql-instance-and-connect-to-it-from-the-tanzu-application-platform-workload-8

https://docs.vmware.com/en/VMware-Tanzu-SQL-with-MySQL-for-Kubernetes/1.6/tanzu-mysql-k8s/GUID-creating-service-bindings.html

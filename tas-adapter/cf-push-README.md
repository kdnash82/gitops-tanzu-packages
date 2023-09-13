cf api <tas-adapter-url> --skip-ssl-validation
cf login
select the cluster you wish to deploy applications to

cf curl /whoami

cat <<EOF | kubectl apply -f -
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cf-admin
  namespace: cf
  annotations:
    cloudfoundry.org/propagate-cf-role: "true"
subjects:
- kind: User
  name: kubernetes-admin #<VALUE-TO-REPLACE-FROM-PREVIOUS-COMMAND>
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: korifi-controllers-admin
  apiGroup: rbac.authorization.k8s.io
EOF

git clone https://github.com/cloudfoundry-samples/spring-music
cd spring-music
cf create-org <name-of-org>
cf target -o <name-of-org>
cf create-space <name-of-space>
cf target -o <name-of-org> -s <name-of-space>
cf push

Troubleshooting
- You may need to provide the path to the manifest file if it is not in the root directory
`cf push -f <path-to-manifest>`
- You may receive an error related to not having a .jar file for the application
`./gradlew clean assemble` will build a jar file.
- You may receive an error related to not having the java buildpacks
```
sudo add-apt-repository ppa:openjdk-r/ppa -y
sudo apt-get update
sudo apt-cache policy openjdk-8-jdk
sudo apt-get install openjdk-8-jdk -y
sudo update-alternatives --config java
sudo update-alternatives --config javac
```
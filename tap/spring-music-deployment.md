# Deploy Spring Music Cloud Foundry Application with TAP

## Create Empty Git Repository
- Create an empty git repo with the project name of `spring-music`

## Generate Accelerator
- Browse to the tap-gui website and click the `Create` button
- Select `Choose` under `TAP Project Configuration`
- Enter a name for the project: i.e. `spring-music`
- Enter Name of the target project: i.e. `spring-music`
- Enter the path to the empty git repo: i.e. `https://gitlab.<FQDN>/<project-org>/spring-music.git`
- Enter the branch you wish to target: i.e. `main`
- Enter the source repository prefix: i.e. `.` 
Note:  This may not be best practice and needs to be researched further.
- Click `Next` - `Generate Accelerator` - `Download Zip File`
- Copy zip file to your jumpbox or workstation
- Unzip the file in your working directory and cd to the unzipped directory
- Follow the procedures in the README from your empty repo to initialize, commit, and push the TAP Generated spring-music files.

## Clone Cloud Foundry Spring Music Repo
Note:  There may be a cleaner way of doing this.
- From a `scratch` or `temp` directory, `git clone https://github.com/cloudfoundry-samples/spring-music`
- Ensure you are in the `TAP Project spring-music` directory
- `cp -R <path-to><scratch/temp>/spring-music/* .`
- `git add .`
- `git commit -m "Adding cloud foundry resources for spring-music"`
- `git push`

## Create Spring Music Namespace
`kubectl create ns spring-music`

## Apply github-ssh-secret to Spring Music Namespace
`sops -d tap/git-secret.sops.yaml | kubectl apply -n spring-music -f -`

## Apply Single User Permissions to Spring Music Namespace
Note: You may be able to copy the value from .dockerconfigjson within the tap-registry secret for <YOUR-REGISTRY-CREDS-BASE64>. `kubectl get secret -n tap-install tap-registry -o yaml`
```
cat <<EOF | kubectl -n dotnet-ns apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: registry-credentials
  annotations:
    secretgen.carvel.dev/image-pull-secret: ""
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJjcy1zLWhhcmJvci5hdm1jLmFybXkubWlsIjp7InVzZXJuYW1lIjoidGFwLWFkbWluIiwicGFzc3dvcmQiOiJNYWdpY0Nsb3VkMjAyMyEhQEAifX19 
---
apiVersion: v1
kind: Secret
metadata:
  name: tap-registry
  annotations:
    secretgen.carvel.dev/image-pull-secret: ""
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJjcy1zLWhhcmJvci5hdm1jLmFybXkubWlsIjp7InVzZXJuYW1lIjoidGFwLWFkbWluIiwicGFzc3dvcmQiOiJNYWdpY0Nsb3VkMjAyMyEhQEAifX19
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
secrets:
  - name: registry-credentials
  - name: github-ssh-secret
imagePullSecrets:
  - name: registry-credentials
  - name: tap-registry
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default-permit-deliverable
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: deliverable
subjects:
  - kind: ServiceAccount
    name: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default-permit-workload
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: workload
subjects:
  - kind: ServiceAccount
    name: default
EOF
```
## Depoy Spring Music App
```
tanzu apps workload create <name-of-app> \
--git-repo https://gitlab.<FQDN>/<project-org>/<name-of-spring-music-project>.git \
--git-branch <branch-name> \
-n <namespace-for-app> \
--type web \
--label app.kubernetes.io/part-of=<name-of-app> 
```

## View Logs for Spring Music Deployment
`tanzu apps workload tail <name-of-app> --namespace <namespace-of-app>`

## View Deployment in TAP GUI
- Browse to tap-gui website
- Select the `Supply Chains` icon
- Click the link for your application
Note:  The section for `Pull Config` is the last step in the supply chain and will be in an error state until it is compeleted.

## Access Application
`tanzu apps workload get <name-of-app> --namespace <namespace-of-app>`
- From this command, there should be a URL that has been created.
- Browse to that URL.
Note: A DNS record may need to be added for the application.  The URL should point to the External-IP of the envoy service.
`kubectl get svc -n tanzu-system-ingress envoy`

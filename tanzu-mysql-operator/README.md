# Install MySQL Operator 
## Switch Context to TAP Cluster

## Export Variables
```
export INSTALL_REGISTRY_USERNAME=
export INSTALL_REGISTRY_PASSWORD=
export INSTALL_REGISTRY_HOSTNAME=
export TDS_INSTALL_REPO=tap-services
export TDS_VERSION=1.4.0
```

## Login Tanzu Registry and Private Registry
`docker login registry.tanzu.vmware.com`
`docker login $INSTALL_REGISTRY_HOSTNAME`

## Copy TDS Images to Private Registry
```
imgpkg copy -b registry.tanzu.vmware.com/packages-for-vmware-tanzu-data-services/tds-packages:${TDS_VERSION}  \
--to-repo ${INSTALL_REGISTRY_HOSTNAME}/${TDS_INSTALL_REPO}/${TDS_VERSION}
```

## Create tanzu-mysql-for-kubernetes-system namespace
`kubectl create ns tanzu-mysql-for-kubernetes-system`

## Create tanzu-image-registry Secret
```
tanzu secret registry add tanzu-image-registry \
  --username ${INSTALL_REGISTRY_USERNAME} --password ${INSTALL_REGISTRY_PASSWORD} \
  --server ${INSTALL_REGISTRY_HOSTNAME} \
  --yes --namespace tanzu-mysql-for-kubernetes-system 
```

## Add TDS Repository
```
tanzu package repository add tanzu-data-services-repository \
--url <INSTALL_REGISTRY_HOSTNAME>/tap-services/1.4.0:1.4.0 \
--namespace tanzu-mysql-for-kubernetes-system
```

## Verify Repository Added

`tanzu package available list --namespace tanzu-mysql-for-kubernetes-system`

## Update Values As Needed in cert-manager directory

## Import sops-key
- Copy value from `tkg/sandbox/tkg-sandbox-tap/sops-key` in vault and save to file
- `gpg --import <sops-key-filename>`

## Retrieve sops-key ID
`gpg --list-keys`

## Create sops-key Secret
```
gpg --export-secret-keys \
--armor <key-id> | \
kubectl create secret generic sops-key \
-n flux-system \
--from-file=sops.asc=/dev/stdin
```

## Update git-secret.sops.yaml - Optional
- Secrets have been encrypted using the sops-key
- Optional- If using vscode, it may be helpful to install the `@signageos/vscode-sops` extension.  This will decrypt sops files when opened and encrypt when saved.

## Override Operator Values - Optional
Create file operator-override-values.yaml and paste in the following:
```
certManagerClusterIssuerName: custom-issuer
imagePullSecretName: custom-secret
resources:
  limits:
    cpu: 500m
    memory: 300Mi
  requests:
    cpu: 500m
    memory: 300Mi
```

## Copy git-secret-REDACTED.yaml
`cp git-secret-REDACTED.yaml git-secret.sops.yaml`

## Update values in git-secret.sops.yaml
- GIT_URL, GIT_USERNAME, GIT_PASSWORD

## Encrypt git-secret.sops.yaml
`sops -e -i -pgp <GPG-KEY-ID> <path-to>/git-secret.sops.yaml`

## Update Git Resource
Edit `url` in `flux-config/tanzu-packages-source.yaml`

## Commit and Push All Updated Files

## Apply Git Secret
`sops -d tanzu-mysql-operator/git-secret.sops.yaml | kubectl apply -n flux-system -f -`
`sops -d tanzu-mysql-operator/git-secret.sops.yaml| kubectl apply -n tanzu-mysql-for-kubernetes-system -f -`

## Apply Git Resource
`kubectl apply -f flux-config/tanzu-packages-source.yaml`

## Verify Git Resource 
`kubectl get gitrepository -n flux-system`

## Apply Kustomization Resource
`kubectl apply -f flux-config/tanzu-packages-mysql-operator.yaml`

## Verify Kustomization Resource
`kubectl get kustomization -n flux-system`

## Verify MySQL Package Install
`kubectl get packageinstalls.packaging.carvel.dev -n tanzu-mysql-for-kubernetes-system`
`kubectl get apps -n tanzu-mysql-for-kubernetes-system`
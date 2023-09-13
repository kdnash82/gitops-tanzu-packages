# Install TAS ADAPTER
## Switch Context to TAP Cluster

## Export Variables
```
export INSTALL_REGISTRY_USERNAME=
export INSTALL_REGISTRY_PASSWORD=
export INSTALL_REGISTRY_HOSTNAME=
export TAS_INSTALL_REPO=tas-adapter
export TAS_VERSION=0.9.0
```

## Login Tanzu Registry and Private Registry
`docker login registry.tanzu.vmware.com`
`docker login $INSTALL_REGISTRY_HOSTNAME`

## Copy TAS Images to Private Registry
```
imgpkg copy -b registry.tanzu.vmware.com/app-service-adapter/tas-adapter-package-repo:0.9.0 \
--to-repo ${INSTALL_REGISTRY_HOSTNAME}/${TAS_INSTALL_REPO}/${TAS_VERSION}
```

## Create tas-adapter-install namespace
`kubeclt create ns tas-adapter-install`

## Create tap-registry Secret
```
tanzu secret registry add tanzunet-registry \
  --username ${INSTALL_REGISTRY_USERNAME} --password ${INSTALL_REGISTRY_PASSWORD} \
  --server ${INSTALL_REGISTRY_HOSTNAME} \
  --export-to-all-namespaces --yes --namespace tas-adapter-install
```

## Add TAS-Adapter Repository
```
tanzu package repository add tas-adapter-repository \
--url <INSTALL_REGISTRY_HOSTNAME>/tas-adapter/0.9.0:0.9.0 \
--namespace tas-adapter-install
```

## Verify Repository Added

`tanzu package available list --namespace tas-adapter-install`

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

## Copy tas-adapter-values-REDACTED.yaml
`cp tas-adapter-values-REDACTED.yaml tas-adapter-values.sops.yaml`

## Update values in tas-adapter-values.sops.yaml

## Encrypt tap-values.sops.yaml
`sops -e -i -pgp <GPG-KEY-ID> <path-to>/tap-values.sops.yaml`

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
`sops -d tas-adapter/git-secret.sops.yaml | kubectl apply -n flux-system -f -`
`sops -d tas-adapter/git-secret.sops.yaml| kubectl apply -n tas-adapter-install -f -`

## Apply Git Resource
`kubectl apply -f flux-config/tanzu-packages-source.yaml`

## Verify Git Resource 
`kubectl get gitrepository -n flux-system`

## Apply Kustomization Resource
`kubectl apply -f flux-config/tanzu-packages-tas.yaml`

## Verify Kustomization Resource
`kubectl get kustomization -n flux-system`

## Verify TAP Install
`kubectl get packageinstalls.packaging.carvel.dev -n tas-adapter-install`
`kubectl get apps -n tas-adapter-install`
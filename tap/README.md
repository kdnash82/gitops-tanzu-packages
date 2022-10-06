# Install TAP
## Create Cluster
## Switch Context to New Cluster

## Export Variables
```
export INSTALL_REGISTRY_USERNAME=
export INSTALL_REGISTRY_PASSWORD=
export INSTALL_REGISTRY_HOSTNAME=
export TAP_INSTALL_REPO=tap
export TAP_VERSION=1.2.1
export TBS_INSTALL_REPO=tbs
export TBS_VERSION=1.6.1
```

## Login Tanzu Registry
`docker login registry.tanzu.vmware.com`

## Copy TAP Images to Private Registry
```
imgpkg copy -b registry.tanzu.vmware.com/tanzu-application-platform/tap-packages:${TAP_VERSION} \
--to-repo ${INSTALL_REGISTRY_HOSTNAME}/${TAP_INSTALL_REPO}/${TAP_VERSION}
```

## Create tap-install namespace
`kubectl create ns tap-install`

## Create tap-registry Secret
```
tanzu secret registry add tap-registry \
  --username ${INSTALL_REGISTRY_USERNAME} --password ${INSTALL_REGISTRY_PASSWORD} \
  --server ${INSTALL_REGISTRY_HOSTNAME} \
  --export-to-all-namespaces --yes --namespace tap-install
```
## Add TAP Package Repo
```
tanzu package repository add tanzu-tap-repository \
  --url ${INSTALL_REGISTRY_HOSTNAME}/${TAP_INSTALL_REPO}/${TAP_VERSION}:${TAP_VERSION} \
  --namespace tap-install
```

## Copy TBS Images to Private Registry
```
imgpkg copy -b registry.tanzu.vmware.com/tanzu-application-platform/full-tbs-deps-package-repo:${TBS_VERSION} \
  --to-repo ${INSTALL_REGISTRY_HOSTNAME}/${TBS_INSTALL_REPO}/${TBS_VERSION}
```

## Veriy TAP Package Repo
`kubectl get package -n tap-install`

## Clone TAP Repo
`git clone https://<GITLAB_URL>/<PATH-TO>/gitops-tanzu-packages.git`

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

## Copy tap-values-REDACTED.yaml
`cp tap-values-REDACTED.yaml tap-values.sops.yaml`

## Update values in tap-values.sops.yaml
- Certain packages installed by TAP may already be on the cluster. ie load-balancing packages such as contour or istio, cert-manager, or flux.
- You can exclude these items from being deployed with tap using `exclude-packages`
- If you do not wish to exclude any packages, remove this from the `tap-values.sops.yaml` file.

ex:
```
excluded_packages:
    - fluxcd.source.controller.tanzu.vmware.com
    - cert-manager.tanzu.vmware.com
    - contour.tanzu.vmware.com
```
- All packages for TBS in ${INSTALL_REGISTRY_HOSTNAME}/${TBS_INSTALL_REPO}/${TBS_VERSION} are the full dependency versions unless you copied over the light versions.  The lite versions are not part of this README.  See VMware documenation for lite versions for TBS.  The lite version requires an open internet connection to pull down dependencies as needed.

## Encrypt tap-values.sops.yaml
`sops -e <path-to>/tap-values.sops.yaml`

## Copy git-secret-REDACTED.yaml
`cp git-secret-REDACTED.yaml git-secret.sops.yaml`

## Update values in git-secret.sops.yaml
- GIT_URL, GIT_USERNAME, GIT_PASSWORD

## Encrypt git-secret.sops.yaml
`sops -e <path-to>/git-secret.sops.yaml`

## Update Git Resource
Edit `url` in `flux-config/tanzu-packages-source.yaml`

## Commit and Push All Updated Files

## Apply Git Secret
`sops -d tap/git-secret.sops.yaml | kubectl apply -n flux-system -f -`
`sops -d tap/git-secret.sops.yaml | kubectl apply -n tap-install -f -`

## Apply Git Resource
`kubectl apply -f flux-config/tanzu-packages-source.yaml`

## Verify Git Resource 
`kubectl get gitrepository -n flux-system`

## Apply Kustomization Resource
`kubectl apply -f flux-config/tanzu-packages-tap.yaml`

## Verify Kustomization Resource
`kubectl get kustomization -n flux-system`

## Verify TAP Install
`kubectl get packageinstalls.packaging.carvel.dev -n tap-install`
`kubectl get apps -n tap-install`

## Add DNS Entry for TAP
- If using `contour` retrieve the load-balancer endpoint
`kubectl get svc -n tanzu-system-ingress`
- Add that value to DNS along with the `tap-gui` url provided in the `tap-values.sops.yaml` file

## Install Full TBS Repository 
```
tanzu package repository add tbs-full-deps-repository \
--url ${INSTALL_REGISTRY_HOSTNAME}/${TBS_INSTALL_REPO}/${TBS_VERSION}:${TBS_VERSION} \
--namespace tap-install
```

`tanzu package install full-tbs-deps -p full-tbs-deps.tanzu.vmware.com -v ${TBS_VERSION} -n tap-install`

## Verify Full TBS Install
`kubectl get apps -n tap-install full-tbs-deps`

## Troubleshooting


## Extra Commands

### Pause Kustomiztion Objects
`flux suspend kustomization -n <namespace> <name-of-kustomization>`



# Manual Procedures to Install TAP
## Deploy Cluster Essentials
https://docs.vmware.com/en/Cluster-Essentials-for-VMware-Tanzu/1.2/cluster-essentials/GUID-deploy.html
## Deploy onto Cluster
https://docs.vmware.com/en/Cluster-Essentials-for-VMware-Tanzu/1.2/cluster-essentials/GUID-deploy.html#deploy-onto-cluster-5
## Installing the Tanzu Appplication Platform Package and Profiles (Online)
https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.2/tap/GUID-install.html

## Installing the Tanzu Appplication Platform Package and Profiles (AirGapped)
https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.2/tap/GUID-install-air-gap.html
# TAP-GUI Configuration

##  Register Entity
- Copy the URL for the tap-catalog/catalog.yaml file from your git repo: i.e. `https://<GIT_URL>/<ORG-OPTIONAL>/<PROJECT>/-/blob/main/catalog-info.yaml`
- Browse to the tap-gui url.  You can obtain the URL by running `kubectl get httpproxy -n tap-gui`
- From the Home page, select `Register Entity`
- Paste the tap-catalog/catalog.yaml URL and click `Analyze`
- Review and Accept

## Troubleshooting
### Accelerator Page is Empty
Network Policies have been set for Flux that prevent Accelerators from being pulled in.

- From the Management Cluster
`flux suspend kustomization -n <name-of-cluster> cluster-services`
- From the TAP Workload Cluster
`kubectl get networkpolicy -n flux-system`
- Delete all network policies in the flux-system namespace
- Refresh the tap-gui page and verify if the Accelerators page has Accelerators


### 509 Error Within the Supply Chain
The Tekton ClusterTask/image-writer which is executed by the supply-chain config-writer when the image of the bundle is pushed to the registry fails as the command imgpkg copy -b <bundle> will get a x509: Certificate signed by unknow authority ... as the private CA certificate is not passed as parameter. See issue-16.

This problem must be fixed manually (till someone will find how to patch the TektonTask !)
```
kubectl patch pkgi tap -n tap-install -p '{"spec":{"paused":true}}' --type=merge
kubectl patch pkgi ootb-templates -n tap-install -p '{"spec":{"paused":true}}' --type=merge
kubectl edit ClusterTask/image-writer
```
and change the following line to pass --registry-verify-certs='False'
```
...
      imgpkg push --registry-verify-certs='False' -b $(params.bundle) -f .
      cat ./.imgpkg/images.yml
    securityContext:
      runAsUser: 0
```
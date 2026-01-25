# Cilium Helm vs Manifest

When installing the Talos cluster, there is an option for a CNI manifest. So the question naturally becomes why install Cilium through Helm as an extra step in the bootstrapping process when we could simply provide a manifest? There's two main reasons, the first being ease of upgrading and the second being storage of these manifests.

## Ease of upgrading

When installing Cilium through helm, there is a straightforward and simple upgrade process when it's time to upgrade to a new version. You can simply run `helm upgrade cilium cilium/cilium ...` with whatever parameters are needed. That's the whole process, which also makes it simple for tools like [ArgoCD](), [FluxCD](), or [Werf]() to add on to this process. When you are doing it through manifests files and having Talos manage the deployment it's easier on cluster bootstrapping but more cumbersome later. The upgrade process involves gathering the manifest files and storing them somewhere for Talos to access them, and then updating the talos manifests to point to the new location, and running a `talosctl apply`.

### Deployment drift

Another problem that helm solves indirectly is how best to ensure a configuration of an application stays in sync. By deploying through helm, those same mentioned above can be used to ensure the state of the application stays in sync, while deploying it through the talos manifests will only check it on node boot time, or after a `talosctl apply`.

## Storage of manifests

The other main benefit of using helm is the ability to store manifests in more types of locations. The talos manifests must be able to be reached over unauthenticated HTTP, which is quite limiting as to where the manifests can exist safely. For example if the manifest does need to be in a location that requires authentication, say due to a security policy, a talos manifest would be unable to read that but all 3 tools listed above would be able to read and manage the manifests without problem. Another problem is you need to store the whole manifests which is unweildy to read for most CNIs on the market as there are numerous CRDs, deployments, and configurations that need to be maintained. A helm chart's value file is already bordering on un-readable for some applications, the manifests those generate are untenable for a human to maintain directly. Thus they are usually stored in some other format and then generated to the destination manifests for talos to consume, so why not just deploy them from their stored state.


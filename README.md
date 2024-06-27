# Experimental test repo (Cozystack + Flux-Operator)

We are using `kustomize.toolkit.fluxcd.io/ssa: merge` annotation so that I can
add our own `spec.sync` to the `FluxInstance`.

This is not known to be reliable and you may cause a major outage, already I've
seen my Flux config nuked once I'm not sure how.

Beware!

Cozystack now (as of 0.8.0) optionally provides a `flux-operator` installation
as a Cluster addon, and installs Flux as a `FluxInstance` in the `cozy-fluxcd`
namespace. The `spec.sync` section is not written by the system, so users can
override it at runtime.

## What did we do

Add a `FluxInstance` in `clusters/test-cluster/flux-system/flux-instance.yaml`
as I have done, leaving alone the `cluster` and `distribution` settings which
are pre-configured.

Add your own config to the `sync` section:

```yaml
...
metadata:
  ...
  annotations:
    kustomize.toolkit.fluxcd.io/ssa: merge
spec:
  cluster: {}
  distribution: {}
  sync:
    kind: GitRepository
    url: "ssh://git@github.com/kingdon-ci/cozy-fleet.git"
    ref: "refs/heads/main"
    path: "clusters/harvey"
    pullSecret: "flux-system"
```

Use the `flux` CLI to create a secret and add the secret as a Deploy Key:

```bash
flux create secret git flux-system --url ssh://git@github.com/kingdon-ci/cozy-fleet.git -n cozy-fluxcd
```

See the Flux documentation on [`flux create secret git`][] for more details.

This would be better with direct configuration and not merged like this, but
I don't think there should be any issues. (We'll find out! YOLO...)

## Works on my machine

One cluster definitely had some issues, may be it was not Flux or Cozystack's
fault. It seemed to be a node trouble condition. I do not know how node issue
like this can result in deleting a secret or a namespace.

I'm betting Cozystack installer did it somehow.

I will keep testing and update this repo with better configs as I learn.

I was able to clear the issue by scaling the `MachineDeployment` and deleting
the trouble node. Not only was the secret adjacent to my `FluxInstance` wiped
out, but also many pods kept returning to `CrashLoopBackOff` - I suspect this
type of issue is reproducible, but it is also the first time I've seen it.

So far since I deleted the trouble node and let another one replace it, there
have been no further issues. I am very novice at KubeVirt and Kamaji!

My intention is to copy all of the configs from Cozystack down to this repo,
sort of as a reverse-GitOps. The configs are all generated, but I want view
access so I can inspect them.

### Future Plans

One day I may add more `ssa:merge` instructions, but for now, the configs in
`apps` are meant to be referenced by Kustomizations in your clusters, like
the [flux2-kustomize-helm example][] repo.

The `tenants/` definitions are meant to be replayed in the event of total loss,
or used as reference to understand the arrangement of our system's tenants.

And the `system/` directory reflects Cozystack core and system packages.

As of this writing, I am using a home-brew distribution of Cozystack built
from some PRs that have yet to merge for release. You can follow my progress:

* [aenix-io/cozystack][]
* [kingdonb/cozystack][]
* [kingdonb/cozystack-talm-demo][]

In a future release of Cozystack, perhaps users who select the Flux addon can
configure the `sync` right there, in the Kubernetes app. It makes sense to do
that, but an open question is where does the pull secret come from?

For now, the configuration has been kept minimal and only what is required, to
preserve some 3-way merge opportunities like this one.

There are probably more changes coming. Cozystack users, we need your feedback.

Enjoy! (PRs welcome!)

[`flux create secret git`]: https://fluxcd.io/flux/cmd/flux_create_secret_git/
[flux2-kustomize-helm example]: https://github.com/fluxcd/flux2-kustomize-helm-example
[aenix-io/cozystack]: https://github.com/aenix-io/cozystack
[kingdonb/cozystack]: https://github.com/kingdonb/cozystack
[kingdonb/cozystack-talm-demo]: https://github.com/kingdonb/cozystack-talm-demo

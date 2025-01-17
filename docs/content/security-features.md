---
title: Porter Security Features
description: Understanding security features built into Porter
---

Porter has a number of security features to ensure that when you run a bundle, it is doing what you intended and does not expose sensitive data.

* [Just-in-time secret injection](#just-in-time-secret-injection)
* [Digests, not tags](#digests-not-tags)
* [Bring your own deployment artifacts](#bring-your-own-deployment-artifacts)
* [Bundles do not run as root](#bundles-do-not-run-as-root)
* [IronBank distributed artifacts](#ironbank-distributed-artifacts)
* [Force push off by default](#force-push-off-by-default)

## Just-in-time secret injection

When a bundle is run, Porter retrieves any sensitive data that the bundle requires such as cloud provider credentials, service accounts, service tokens, and injects them directly into the bundle's container as either environment variables or files.

Sensitive data is always resolved just-in-time, is never persisted in Porter or on the host, and any sensitive data generated by a bundle is stored securely in your [preferred secret store](/plugins/types/#secrets), such as Hashicorp Vault or Azure Key Vault.
You do not need to copy secrets into Porter, synchronize Porter when you rotate credentials, or worry that Porter could be used leak sensitive data.

Learn how to use [credential sets] and [secrets plugins] to connect Porter to your existing secret stores.

## Digests, not tags

When authoring your bundle, it is easier to think about and remember a tag instead of a digest.
But when a bundle runs, you really want to reference images by their digest.
This ensures that at runtime you are deploying the images that you tested when the bundle was built.
An image tag can be force pushed, or overwritten, and there is no guarantee that a tag won't be changed or tampered with.
Digests are immutable and are the safest way to refer to an image.

Porter helps you reference images by digest, while still using tags that make sense, such as latest or an application version number.

Let's say for example that your bundle deploys nginx:1.23.1.
When you author the bundle, you can use the informal tagged image reference and when the bundle is built, Porter will lookup the digest for you and update your reference in the bundle to use the digest.
Later when running the bundle, you can rely on the images that you pulled when the bundle was built are exactly the ones that you intended to use.

## Bring your own deployment artifacts

A bundle can rely on many types of deployment artifacts such as docker images, helm charts, kubernetes manifests, or terraform providers to name a few.
When you build a bundle, those artifacts can be redistributed with the bundle, locking them in-place.

The goal is that when your bundle runs, it is using well-known, immutable copies of your deployment artifacts that can't be tampered with or removed.
This ensures that you always have access to what your application needs to deploy.

Here are some examples of how Porter ensures that your deployment artifacts are always available to your bundle:

* Porter copies the mixins and tools used by your bundle into the bundle.
* Any files used by your bundle, such as kustomize files, kubernetes manifests and configuration files, are copied into the bundle.
* The helm3 mixin caches your helm repositories into the bundle.
* The terraform mixin automatically mirrors your terraform providers into the bundle.
* Porter keeps a copy of your referenced images, and provides you a way to reference the relocated image. See the [airgap example](/examples/airgap/) for details on how that works.

## Bundles do not run as root

When Porter runs a bundle, it runs inside a Docker container or Kubernetes pod (if you are using the Porter Operator).
Porter builds and runs the bundle with a non-root user, limiting the container's access to resources on the host.

Here is an explanation of [why you do not want to run containers as root](https://medium.com/@mccode/processes-in-containers-should-not-run-as-root-2feae3f0df3b).

The [Custom Dockerfile](/bundle/custom-dockerfile/) documentation has more information about writing a bundle to work when run as a non-root user.

## IronBank distributed artifacts

Porter and the Porter Operator are redistributed on the [PlatformOne IronBank registry](https://p1.dso.mil/products/iron-bank).
These images are built on an isolated network, regularly scanned for CVEs, and new releases are available on average 1-2 days after the official Porter releases on GitHub.

## Force push off by default

When you publish or copy a bundle to a registry, Porter checks if the destination reference already exists and prevents you from accidentally overwriting it.
This prevents you from accidentally overwriting a production bundle.

You can always opt into the default Docker behavior of overwriting existing artifacts on push by specifying the \--force flag.
But by default, that foot gun is disabled.

# See Also

* [Distribute Bundles](/distribute-bundles/)
* [Airgapped Deployments](/administrators/airgap/)
* [Blog: Upgrade your plugins to securely store sensitive data](/blog/persist-sensitive-data-safely/)

[crednetial sets]: /credentials/#credential-sets
[secrets plugins]: /plugins/types/#secrets

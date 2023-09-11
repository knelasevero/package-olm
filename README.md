# Package generation for OLM

*⚠️ this repository was forked from https://github.com/cert-manager/cert-manager-olm and this release process is stil experimental*

This repository contains scripts and files that are used to package a certain project for Red Hat's [Operator Lifecycle Manager (OLM)][].
This allows users of [OpenShift][] and [OperatorHub][] to easily install a project into their clusters.
It is currently an experimental deployment method.

[Operator Lifecycle Manager (OLM)]: https://olm.operatorframework.io/
[OpenShift]: https://www.okd.io/
[OperatorHub]: https://operatorhub.io/

The package is called an [Operator Bundle][] and it is a container image that stores the Kubernetes manifests and metadata associated with an operator.
A bundle is meant to represent a specific version of an operator.

The bundles are indexed in a [Catalog Image][] which is pulled by OLM in the Kubernetes cluster.
Clients such as `kubectl operator` then interact with the [OLM CRDs][] to "subscribe" to a particular release channel.
OLM will then install the newest project bundle in that release channel and perform upgrades as newer versions are added to that release channel.

[Operator Bundle]: https://github.com/operator-framework/operator-registry/blob/master/docs/design/operator-bundle.md
[OLM CRDs]: https://olm.operatorframework.io/docs/concepts/crds/
[Catalog Image]: https://olm.operatorframework.io/docs/glossary/#index

## Release Process

In order to test that OLM can upgrade to the new version you can perform a test release,
and publish a "release candidate" bundle by creating release candidate PRs to
[Kubernetes Community Operators Repository][] and to
[OpenShift Community Operators Repository][].

Once these bundles have been merged, the release candidate version of a project should be available
in the "candidates" channel __only__.

You can test upgrading to the new version by creating a Subscription targeting the "candidate" channel
(which should also contain the latest stable version),
and set the "startingCSV" to the last stable version
and "installPlanApproval" to "Manual". E.g.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: YOURPROJECT
  namespace: openshift-operators
spec:
  channel: candidate
  installPlanApproval: Manual
  name: YOURPROJECT
  source: community-operators
  sourceNamespace: openshift-marketplace
  startingCSV: YOURPROJECT.vVERSION
```

Then when you have published the release candidate, you should verify that you can upgrade the project to the new version.
Check the logs and events for upgrade errors during the upgrade.

### Release Steps

* Add the new version to `VERSION` at the top of the `Makefile`
* If this is a release candidate:
  * Add `-rc1` as a suffix to `BUNDLE_VERSION`
* If this is the final release:
  * Remove the `-rc1` suffix from `BUNDLE_VERSION`
* Run `make bundle-build bundle-push catalog-build catalog-push` to generate a bundle and a catalog.
* Run `make bundle-validate` to check the generated bundle files.
* `git commit` the bundle changes.
* [Preview the generated clusterserviceversion file on OperatorHub ](https://operatorhub.io/preview)
* Test the generated bundle locally (See testing below)
* Create a PR on the [Kubernetes Community Operators Repository][],
  adding the new or updated bundle files to `operators/YOURPROJECT/`
  under a sub-directory named after the bundle version

  `make update-community-operators`

* Create a PR on the [OpenShift Community Operators Repository][],
  adding the new or updated bundle files to `operators/YOURPROJECT/`
  under a sub-directory named after the bundle version

  `make update-community-operators-prod`

[Kubernetes Community Operators Repository]: https://github.com/k8s-operatorhub/community-operators
[OpenShift Community Operators Repository]: https://github.com/redhat-openshift-ecosystem/community-operators-prod
[Where to contribute]: https://operator-framework.github.io/community-operators/contributing-where-to/

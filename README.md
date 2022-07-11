# Kyverno Operator

Operator to deploy the [Kyverno](https://kyverno.io) policy management framework for Kubernetes

## Installation

The operator can be run locally, such as during the development phase, or within the cluster.

First, install the operator CRD's

```shell
make install
```

Then, create a namespace called `kyverno` that will contain all of the resources:

```shell
kubectl apply -f examples/00-namespace.yaml
```

With the CRD installed and namespace created, the operator can be installed by running locally or within a cluster.

### Running Locally

Run the operator from your local machine outside the cluster by executing the following command:

```shell
make run
```

### Running Within a Cluster

Before you can run the operator within a cluster, it must be built locally and pushed to a container registry, such as [quay.io](https://quay.io).

1. Build the image

```shell
make docker-build IMG=<registry>/<repository>:<tag>
```

2. Push the image

```shell
make docker-push IMG=<registry>/<repository>:<tag>
```

3. Deploy the operator

```shell
make deploy IMG=<registry>/<repository>:<tag>
```

## OLM Integration

The operator can be integrated with the [Operator Lifecycle Manager](https://olm.operatorframework.io) for easy deployment and management.

Resources are available in the `examples` folder to deploy the `CatalogSource` to bring in the operator into an environment, but also the `OperatorGroup` and `Subscription` to deploy the operator.

Alternatively, an operator _bundle_ and _index_ can be built as shown below.

First, set environment variables for the operator image published previously and the location fot the bundle and index images:

```shell
export IMG=<registry>/<repository>:<tag>
export BUNDLE_IMG=<registry>/<repository>:<tag>
export CATALOG_IMG=<registry>/<repository>:<tag>
```

1. Create the bundle

```shell
make bundle
```

2. Build the bundle image

```shell
make bundle-build
```

3. Push the bundle image

```shell
make bundle-push
```

4. Build the index image

```shell
make catalog-build
```

4. Push the index image

```shell
make catalog-push
```

With the images built and pushed, create the following resources:

1. Create the `CatalogSource`

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: kyverno-operator
  namespace: openshift-marketplace
spec:
  displayName: Kyverno Operator
  publisher: Kyverno Community
  sourceType: grpc
  image: <registry>/<repository>:<tag>
  updateStrategy:
    registryPoll:
      interval: 45m
```

2. Create the `OperatorGroup`

```shell
kubectl create -f examples/02-operatorgroup.yaml
```

3. Create the `Subscription`

```shell
kubectl create -f examples/03-subscription.yaml
```

The operator should be running in the `kyverno` namespace

## Deploy Kyverno

Now that the operator has been deployed, create an instance of the `Kyverno` Custom Resource to deploy Kyverno. Examples can be found in the [config/samples](config/samples) directory.

# Releasing a new version of apiserver-network-proxy

Please note this guide is only intended for the admins of this repository, and requires write access.

Creating a new release of network proxy involves releasing a new version of the client library (konnectivity-client) and new images for the proxy agent and server. Generally we also want to upgrade kubernetes/kubernetes with the latest version of the images and library, but this is a GCE specific change.

1. If increasing the minor version (the `y` in `x.y.z`), a new release branch must be created. The name of this branch should be `release-x.y` where `x` and `y` correspond to the major and minor release numbers for apiserver-network-proxy. For example, if increasing the apiserver-network-proxy from verision `0.98.4` to `0.99.0` a new branch should be created named `release-0.99`.

    After making the new tag for the release version, use the following command to create the new branch:

    ```
    # assuming a release version of 0.99.0
    export RELEASE=release-0.99
    git checkout -b "${RELEASE}"
    git push upstream "${RELEASE}"
    ```

2. The first step involves creating a new git tag for the release, following semver for go libraries. A tag is required for both the repository and the konnectivity-client library. For example releasing the `0.99.0` version will have two tags `v0.99.0` and `konnectivity-client/v0.99.0` on the appropriate commit. The minor version number (the `y` in `x.y.z`) should match the minor version of the Kubernetes version that is utilized by the apiserver-network-proxy. The patch level version number (the `z` in `x.y.z`) should increase by one unless a new minor version is being created, in which case it should be `0`.

    In the master branch, choose the appropriate commit, and determine a patch version based on the required Kubernetes version and the current patch level.

    Example commands for `HEAD` of `master` branch. (Assumes you have `git remote add upstream git@github.com:kubernetes-sigs/apiserver-network-proxy.git`.)

    ```
    # Assuming v0.2.1 exists
    export TAG=v0.2.2
    export MESSAGE="Meaningful description of change."

    git fetch upstream
    git tag -a "${TAG}" -m "${MESSAGE}" upstream/master
    git tag -a "konnectivity-client/${TAG}" -m "${MESSAGE}" upstream/master
    git push upstream "${TAG}"
    git push upstream "konnectivity-client/${TAG}"
    ```

    In a release branch, the process is similar but corresponds to earlier minor versions.

    Example commands for `HEAD` of `release-0.0` branch:

    ```
    # Assuming v0.1.35 exists, then it is on the release-0.1 branch
    export RELEASE=release-0.1
    export TAG=v0.1.36
    export MESSAGE="Meaningful description of change."

    git fetch upstream
    git tag -a "${TAG}" -m "${MESSAGE}" upstream/${RELEASE}
    git tag -a "konnectivity-client/${TAG}" -m "${MESSAGE}" upstream/${RELEASE}
    git push upstream "${TAG}"
    git push upstream "konnectivity-client/${TAG}"
    ```

    Once the two tags are created, the konnectivity-client can be imported as a library in kubernetes/kubernetes and other go programs.

3. To publish the proxy server and proxy agent images, they must be promoted from the k8s staging repo. An example PR can be seen here: [https://github.com/kubernetes/k8s.io/pull/5686](https://github.com/kubernetes/k8s.io/pull/5686)

    The SHA in the PR corresponds to the SHA of the image within the k8s staging repo. (This is under the **Name** column)

    The images can be found here for the [proxy server](https://console.cloud.google.com/artifacts/docker/k8s-staging-kas-network-proxy/us/gcr.io/proxy-server?gcrImageListsize=30) and [proxy agent](https://console.cloud.google.com/artifacts/docker/k8s-staging-kas-network-proxy/us/gcr.io/proxy-agent?gcrImageListsize=30).

    Please ensure that the commit shown on the tag of the image matches the one shown in the tags page in the network proxy repo.

    <img src="https://user-images.githubusercontent.com/7691399/106816880-09040600-6644-11eb-8907-f50c53dfe475.png" width="400px" height="300px" /> <img src="https://user-images.githubusercontent.com/7691399/106815303-a4e04280-6641-11eb-82d2-4ef4fb34437a.png" width="400px" height="300px" />

4. Finally, update kubernetes/kubernetes with the new client library and images.

    An example PR can be found here: [https://github.com/kubernetes/kubernetes/pull/94983](https://github.com/kubernetes/kubernetes/pull/94983)

    Image paths must be bumped and are located in:

    - cluster/gce/addons/konnectivity-agent/konnectivity-agent-ds.yaml
    - cluster/gce/manifests/konnectivity-server.yaml

    To update the library, no go.mod files need to be manually modified and running these commands in k/k will perform the necessary go.mod changes.

    ```
    ./hack/pin-dependency.sh sigs.k8s.io/apiserver-network-proxy/konnectivity-client v0.0.15
    ./hack/update-codegen.sh
    ./hack/update-vendor.sh
    ```

## Updating Go Dependencies

`go.mod` versions should be kept consistent between the apiserver-network-proxy [main module](go.mod),
and the [konnectivity-client module](/konnectivity-client/go.mod).

Konnectivity-client dependency versions must be compatible
[Kubernetes go.mod versions](https://github.com/kubernetes/kubernetes/blob/master/go.mod),
meaning for a given release branch the konnectivity-client versions must not be newer than the
Kubernetes versions.

In practice, this means that to update `konnectivity-client/go.mod` dependencies, a new Konnectivity
branch must be cut, and the dependencies should be pinned to the latest versions used by Kubernetes.

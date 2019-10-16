Live Git Flexvolume for Kubernetes
==================================

Flexvolume driver for Git repos that should track updates to the mounted repo.

Installing
----------

This Flexvolume driver is a bash script that makes use of binaries installed on
the Kubernetes nodes.

Installing the script itself is done simply as;
```bash
VOLUME_PLUGIN_DIR="/usr/libexec/kubernetes/kubelet-plugins/volume/exec"
mkdir -p "$VOLUME_PLUGIN_DIR/ananace~git-live"
cd "$VOLUME_PLUGIN_DIR/ananace~git-live"
curl -L -O https://raw.githubusercontent.com/ananace/flexvolume-git-live/master/git-live
chmod 755 git-live
```

The `git-live` script requires a couple of tools to be installed on all the
nodes where it's to be used, additionally it currently requires systemd for
launching the background updates;

- `awk`
- `base64`
- `git`
- `jq`
- `realpath`
- `sha256sum`

Running
-------

Once the Flexvolume plugin has been installed on all relevant nodes, running it
is as simple as creating a Kubernetes pod that will use it for a volume;

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: git-example
spec:
  containers:
  - name: busybox
    image: busybox
    command:
      - sh
      - -c
      - |
        ls -l /data
        tail -f /data/README.md
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    flexVolume:
      driver: ananace/git-live
      options:
        repo: https://github.com/ananace/flexvolume-git-live
      readOnly: true
```

This example pod will check out the flexvolume sources and then actively print
any new lines as they are added to the README file and pushed.

Caveats
-------

Due to the fact that checkouts are done as root on the underlying system, the
git index has been moved away from the repository itself. This also means that
tools inside the running pod will not be able to retrieve any data from the
mounted git index.

Currently, there's no way to differentiate read-only and read-write, the
background updates will *always* override any user changes on every interval.

If you only require a static git checkout that doesn't follow any new commits,
the Kubernetes documentation [provides a better route][1].

[1]: https://kubernetes.io/docs/concepts/storage/volumes/#gitrepo

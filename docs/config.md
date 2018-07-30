# Virtlet configuration

Virtlet has a flexible configuration mechanism that makes it possible
to specify per-node configuration using CRDs.

## Using per-node configuration

In order to use per-node configuration, you need to create Virtlet
CRD definitions before you deploy Virtlet:

```bash
virtletctl gen --crd | kubectl apply -f -
```

After that, you can add one or more Virtlet configuration mappings:
```bash
kubectl apply -f - <<EOF
---
apiVersion: "virtlet.k8s/v1"
kind: VirtletConfigMapping
metadata:
  name: test
  namespace: kube-system
spec:
  nodeName: kube-node-2
  priority: 1
  config:
    disableKVM: true
    logLevel: 5
    rawDevices: sda*,loop*
---
apiVersion: "virtlet.k8s/v1"
kind: VirtletConfigMapping
metadata:
  name: test-labels
  namespace: kube-system
spec:
  nodeSelector:
    foo: bar
  priority: 10
  config:
    logLevel: 3
EOF
```

You can delete Virtlet pods to have them restarted and pick up the
changes after you add, remove or modify the configuration mappings.

All the config mappings must reside in `kube-system` namespace.  Each
mapping can specify `nodeSelector` or `nodeName` to target a subset of
the nodes. If neither `nodeSelector` nor `nodeName` is specified, the
mapping will target all the nodes. If the several mappings apply to
the node, their order is determined by `priority` value, with mappings
with higher value of `priority` taking precendence. `config` specifies
the list of configuration fields (see the full table of config fields
in the next section of this document).

In the example above, for the node named `kube-node-2`, KVM is
disabled, log level is set to 5 and the raw device list is set to
`sda*,loop*`.  For nodes with `foo=bar` label, the log level is
overridden and set to 3.  If `kube-node-2` node has `foo=bar` label,
the log level will be set to 3 there as the label-based mapping has
higher priority.

## Virtlet configuration summary

The table below lists all the Virtlet configuration fields, with their
corresponding command line flags and environment variables that are
handled by Virtlet binary. Note that you may only need these
environment variables and configuration flags if you're not using the
standard Virtlet deployment YAML as generated by `virtletctl gen`.

<!--
    The part of this file between 'begin' / 'end' comments is updated
    automatically by build/cmd.sh update-docs
-->
<!-- begin -->
| Description | Config field | Default value | Type | Command line flag / Env |
| --- | --- | --- | --- | --- |
| Path to fd server socket | `fdServerSocketPath` | `/var/lib/virtlet/tapfdserver.sock` | string | `--fd-server-socket-path` / `VIRTLET_FD_SERVER_SOCKET_PATH` |
| Path to the virtlet database | `databasePath` | `/var/lib/virtlet/virtlet.db` | string | `--database-path` / `VIRTLET_DATABASE_PATH` |
| Image download protocol. Can be https or http | `downloadProtocol` | `https` | string | `--image-download-protocol` / `VIRTLET_DOWNLOAD_PROTOCOL` |
| Image directory | `imageDir` | `/var/lib/virtlet/images` | string | `--image-dir` / `VIRTLET_IMAGE_DIR` |
| Image name translation configs directory | `imageTranslationConfigsDir` | `/etc/virtlet/images` | string | `--image-translation-configs-dir` / `VIRTLET_IMAGE_TRANSLATIONS_DIR` |
| Libvirt connection URI | `libvirtURI` | `qemu:///system` | string | `--libvirt-uri` / `VIRTLET_LIBVIRT_URI` |
| Comma separated list of raw device glob patterns which VMs can access (without '/dev/' prefix) | `rawDevices` | `loop*` | string | `--raw-devices` / `VIRTLET_RAW_DEVICES` |
| The path to UNIX domain socket for CRI service to listen on | `criSocketPath` | `/run/virtlet.sock` | string | `--listen` / `VIRTLET_CRI_SOCKET_PATH` |
| Display logging and the streamer | `disableLogging` | `false` | boolean | `--disable-logging` / `VIRTLET_DISABLE_LOGGING` |
| Forcibly disable KVM support | `disableKVM` | `false` | boolean | `--disable-kvm` / `VIRTLET_DISABLE_KVM` |
| Enable SR-IOV support | `enableSriov` | `false` | boolean | `--enable-sriov` / `VIRTLET_SRIOV_SUPPORT` |
| Path to CNI plugin binaries | `cniPluginDir` | `/opt/cni/bin` | string | `--cni-bin-dir` / `VIRTLET_CNI_PLUGIN_DIR` |
| Path to the CNI configuration directory | `cniConfigDir` | `/etc/cni/net.d` | string | `--cni-conf-dir` / `VIRTLET_CNI_CONFIG_DIR` |
| Calico subnet size to use | `calicoSubnetSize` | `24` | integer | `--calico-subnet-size` / `VIRTLET_CALICO_SUBNET` |
| Enable regexp image name translation | `enableRegexpImageTranslation` | `true` | boolean | `--enable-regexp-image-translation` / `IMAGE_REGEXP_TRANSLATION` |
| CPU model to use in libvirt domain definition (libvirt's default value will be used if not set) | `cpuModel` |  | string | `--cpu-model` / `VIRTLET_CPU_MODEL` |
| Log level to use | `logLevel` | `1` | integer | `--v` / `VIRTLET_LOGLEVEL` |
<!-- end -->

Only the following config fields mentioned in this table can be used
with standard Virtlet deployment YAML: `downloadProtocol`,
`rawDevices`, `disableKVM`, `enableSriov`, `calicoSubnetSize`,
`enableRegexpImageTranslation`, `cpuModel` and `logLevel`. Other options may need
adjusting the YAML to change the paths of volume
mounts. `disableLogging` option is intended for debugging purposes
only.
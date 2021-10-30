# Kustomization for Deploying OpenShift API for Data Protection Operator

See also oadp operator [documentation](https://github.com/openshift/oadp-operator/tree/master/docs).

# Velero Client

Configure Velero client to use non-default namespace `oadp-operator`:

```
$ velero client config set namespace=oadp-operator
```

Velero client configuration file can be found at `~/.config/velero/config.json`

# Backup and Restore of CSI Volumes

The respective volumesnapshotclass must have the `velero.io/csi-volumesnapshot-class=true` label:

```
$ oc label volumesnapshotclass ocs-storagecluster-rbdplugin-snapclass velero.io/csi-volumesnapshot-class=true
```

Create a test project:

```
$ oc new-project volume-test
```

Create a PVC to back up:

```
$ oc create --filename - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myvolume
spec:
  storageClassName: ocs-storagecluster-ceph-rbd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
EOF
```

Create a backup:

```
$ velero backup create mybackup --include-namespaces volume-test
```

Check the backup status:

```
$ oc get backup -n oadp-operator mybackup -o yaml
```

Create a test restore project:

```
$ oc new-project volume-test-restore
```

Restore the backup:

```
$ velero restore create myrestore --from-backup mybackup --namespace-mappings volume-test:volume-test-restore
```

Delete the backup if not needed any longer:

```
$ oc delete backup -n oadp-operator mybackup
```

# Backup and Restore of vSphere Volumes

Create a test project:

```
$ oc new-project volume-test
```

Create a PVC to back up:

```
$ oc create --filename - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myvolume
spec:
  storageClassName: thin-csi
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
EOF
```

Create a backup:

```
$ velero backup create mybackup --include-namespaces volume-test --snapshot-volumes --volume-snapshot-locations vsphere
```

Check the backup status:

```
$ oc get backup -n oadp-operator mybackup -o yaml
```

Create a test restore project:

```
$ oc new-project volume-test-restore
```

Restore the backup:

```
$ velero restore create myrestore --from-backup mybackup --namespace-mappings volume-test:volume-test-restore
```

Delete the backup if not needed any longer:

```
$ oc delete backup -n oadp-operator mybackup
```

# Troubleshooting

Scale the oadp operator down to zero replicas:

```
$ oc edit csv oadp-operator.v0.4.0
```

Add `--log-level=debug` to the server command-line:

```
$ oc edit deploy velero
```

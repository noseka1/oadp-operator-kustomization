# Kustomization for Deploying OpenShift API for Data Protection Operator

See also oadp operator [documentation](https://github.com/openshift/oadp-operator/tree/master/docs).

# Velero Client

Configure Velero client to use non-default namespace `oadp-operator`:

```
$ velero client config set namespace=oadp-operator
```

# Backing up CSI Volume

The respective volumesnapshotclass must have the `velero.io/csi-volumesnapshot-class=true` label:

```
$ oc label volumesnapshotclass ocs-storagecluster-rbdplugin-snapclass velero.io/csi-volumesnapshot-class=true
```

Create a test project:

```
$ oc create namespace volume-test
```

Create a PVC to back up:

```
$ oc create --filename - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-rbd-volumeclaim
  namespace: volume-test
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
$ velero backup create mybackup --namespace oadp-operator --include-namespaces volume-test
```

Check the backup status:

```
$ oc get backup -n oadp-operator mybackup -o yaml
```

Delete the backup if not needed any longer:

```
$ oc delete backup -n oadp-operator mybackup
```

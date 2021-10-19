# Kustomization for Deploying OpenShift API for Data Protection Operator


See also oadp operator [documentation](https://github.com/openshift/oadp-operator/tree/master/docs).

The volumesnapshotclass must have the `velero.io/csi-volumesnapshot-class` label:

```
$ oc label volumesnapshotclass ocs-storagecluster-rbdplugin-snapclass velero.io/csi-volumesnapshot-class=true
```

Create a backup:

```
$ velero backup create mybackup --namespace oadp-operator --include-namespaces volume-test
```

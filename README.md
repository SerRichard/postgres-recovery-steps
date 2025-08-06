# postgres-recovery-steps

### Run the recovery pod to run pg_restore.

Ensure the pg dump file is available on a local mount. Find the relevant volume using lsblk and mount the disk ( ex. /dev/vdc ) to a local path.

```
sudo mount /dev/vdc ~/tmp-data
```

We need to create the PV and PVC for the local mount where the dump file is available.

```
kubectl create -n tmp-psql -f volumes.yaml
```

Assuming we have a new minio service in the cluster as minio-sc. We need to register the two alias's, and then we can mirror the data.

```
kubectl create -n tmp-psql -f recovery.yaml
```

### Clean up resources

```
kubectl delete -n tmp-psql -f recovery.yaml
kubectl delete -n tmp-psql -f volumes.yaml
sudo umount ~/tmp-data
```


### Install a tmp postgresql instance ( if required )

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

```
kubectl create ns tmp-psql
microk8s helm3 install tmp-postgres bitnami/postgresql --namespace tmp-psql -f values.yaml
```

Exec into the deployed pod, login to psql and ensure the necessary db and db role are created.
```
CREATE database <DBNAME>;
CREATE ROLE <ROLE> WITH LOGIN PASSWORD <PASSWORD>;
```

Uninstall.
```
microk8s helm3 uninstall tmp-postgres -n tmp-psql
```
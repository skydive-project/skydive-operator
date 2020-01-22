# Skydive Operator

Skydive is an open source real-time network topology and protocols analyzer found here:

* <http://github.com/skydive-project/skydive>

This document shows how to install the skydive-operator, and deploy the Skydive different options

## Pre Requisites

- Kubernetes version 1.10.0 and higher

- Persistent volume is needed only if you want to "look back in time" with skydive (that is, if you are interested in the monitoring history); if you don't , then it is not required (an elastic search container will not be created). You can create a persistent volume via the IBM Cloud Private interface or through a yaml file. For example:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <persistent volume name>
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: <PATH>
```
- S3 compatible Object Storage available either locally on the same cluster, or at some other location.
 It is required only if you want to store the netflows collected by skydive for some future analysis; if you don't , then it is not required (an exporter container will not be created).

## Installing the Operator

To install the operator:

First deploy the custom resource definitions:
```bash
$ kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/crds/charts.helm.k8s.io_skydives_crd.yaml
$ kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/crds/charts.helm.k8s.io_netflowcollectors_crd.yaml
```
The charts.helm.k8s.io_skydives_crd.yaml defines a resource type skydive and charts.helm.k8s.io_netflowcollectors_crd.yaml defines skydive as a netflow collector

then install the operator:
```bash
$ kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/service_account.yaml
$ kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/role.yaml
$ kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/role_binding.yaml
$ kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/operator.yaml
```
The commands deploy skydive-operator on the Kubernetes cluster. The operator now will monitor the defined custom resources (netflowcollectors and skydives).
At this point there are no such actual resources present in the cluster, only definitions of the 2 possible types.

Next an actuall resource can be created - either one of the two defined possible types:
```bash
kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/crds/charts.helm.k8s.io_v1alpha1_skydive_cr.yaml
```
or
```bash
kubectl create -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/crds/charts.helm.k8s.io_v1alpha1_netflowcollector_cr.yaml
```
Note: By running on of the above commands, you instruct the skydive-operator to deploy the selected option of skydive with *default* configuration parameters.
To customize the configuration parameters, copy the rellevant cr yaml file to your local environment, modify the values and run the create command on the modified file.

## Uninstalling

To remove the custom resource:

```bash
$ kubectl delete -f https://github.com/skydive-project/skydive-operator/blob/master/deploy/crds/charts.helm.k8s.io_v1alpha1_skydive_cr.yaml
```
or
```bash
$ kubectl delete -f https://github.com/skydive-project/skydive-operator/blob/master/deploy/crds/charts.helm.k8s.io_v1alpha1_netflowcollector_cr.yaml
```
The command causes the skydive-operator to remove all the Kubernetes components associated with skydive custom resource/ netflow collector custom resource


then (optionally) you can remove the operator:
```bash
$. kubectl delete -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/operator.yaml
```
and the custom resource definitions and rbac definitions:
```bash
$ kubectl delete -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/crds/charts.helm.k8s.io_skydives_crd.yaml
$ kubectl delete -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/crds/charts.helm.k8s.io_netflowcollectors_crd.yaml
$ kubectl delete -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/role_binding.yaml
$ kubectl delete -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/role.yaml
$ kubectl delete -f https://raw.githubusercontent.com/skydive-project/skydive-operator/master/deploy/service_account.yaml
```

## Configuration

The netflowcollector custom resource can be customized using the following configuration parameters:

| Parameter                            | Description                                     | Default                                                    |
| ----------------------------------   | ---------------------------------------------   | ---------------------------------------------------------- |
| `exporter.write.s3.endpoint`         | Endpoint of the Object Storage to be used       | `http://127.0.0.1:9000`                                    |
| `exporter.write.s3.installLocalMinio`| Install default Minio Object Storage locally    | `true`                                                     |
| `exporter.write.s3.region`           | Object Store region                             | `default`                                                  |
| `exporter.write.s3.use_api_key`      | Use an api key for Object Store autentication   | `false`                                                    |
| `exporter.write.s3.api_key`          | api key for Object Store autentication          | Empty                                                         |
| `exporter.write.s3.access_key`       | access key for Object Store autentication       | `admin`                                                    |
| `exporter.write.s3.secret_key`       | secret key for Object Store autentication       | `admin1234`                                                |
| `exporter.store.bucket`              | bucket name to be used in Object Store          | `default`                                                  |
| `exporter.store.objectPrefix`        | prefix of stroed objects                        | `default`                                                  |

The Skydive custom resource can be castomized using the following configuration parameters (and generally using any of the helm configuration parameters defined[https://github.com/skydive-project/skydive-helm](https://github.com/skydive-project/skydive-helm).):

| Parameter                            | Description                                     | Default                                                    |
| ----------------------------------   | ---------------------------------------------   | ---------------------------------------------------------- |
| `image.repository`                   | Skydive image repository                        | `skydive/skydive`                                           |
| `image.tag`                          | Image tag                                       | `0.24.0`                                                   |
| `image.secretName`                   | Image secret for private repository             | Empty                                                      |
| `image.imagePullPolicy`              | Image pull policy                               | `IfNotPresent`                                             |
| `resources`                          | CPU/Memory resource requests/limits             | Memory: `8192Mi`, CPU: `2000m`                             |
| `service.name`                       | service name                                    | `skydive`                                                  |
| `service.type`                       | k8s service type (e.g. NodePort, LoadBalancer)  | `NodePort`                                                 |
| `service.port`                       | TCP port                                        | `8082`                                                     |
| `etcd.port`                          | TCP port                                        | `12379`                                                     |
| `analyzer.topology.fabric`           | Fabric connecting k8s nodes                     | `TOR1->*[Type=host]/eth0`                                  |
| `env`                                | Extended environment variables                  | Empty                                                      |
| `storage.elasticsearch.host`         | ElasticSearch host                              | `127.0.0.1`                                                |
| `storage.elasticsearch.port`         | ElasticSearch port                              | `9200`                                                     |
| `storage.flows.indicesToKeep`        | Number of flow indices to keep in storage       | `10`                                                       |
| `storage.flows.indexEntriesLimit`    | Number of flow records to keep per index        | `10000`                                                    |
| `storage.topology.indicesToKeep`     | Number of topology indices to keep in storage   | `10`                                                       |
| `storage.topology.indexEntriesLimit` | Number of topology records to keep per index    | `10000`                                                    |
| `persistence.enabled`                | Use a PVC to persist data                       | `false`                                                    |
| `persistence.useDynamicProvisioning` | Specify a storageclass or leave empty           | `false`                                                    |
| `dataVolume.name`                    | Name of the PVC to be created                   | `datavolume`                                               |
| `dataVolume.existingClaimName`       | Provide an existing PersistentVolumeClaim       | `nil`                                                      |
| `dataVolume.storageClassName`        | Storage class of backing PVC                    | `nil`                                                      |
| `dataVolume.size`                    | Size of data volume                             | `10Gi`                                                     |

## Documentation

Skydive documentation can be found here:

* [http://skydive.network](http://skydive.network)

## Contact and Support

* Mailing list: [https://www.redhat.com/mailman/listinfo/skydive-dev](https://www.redhat.com/mailman/listinfo/skydive-dev)
* Issues: [https://github.com/skydive-project/skydive/issues](https://github.com/skydive-project/skydive/issues)
          [https://github.com/skydive-project/skydive-operator/issues](https://github.com/skydive-project/skydive-operator/issues)
* Slack

Invite : https://slack.skydive.network
Workspace : https://skydive-project.slack.com

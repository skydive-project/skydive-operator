# Skydive Helm Chart

Skydive is an open source real-time network topology and protocols analyzer found here:

* <http://github.com/skydive-project/skydive>

## Pre Requisites

- Helm version 2.5.0 and higher

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

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
$ helm install --name my-release stable/skydive
```

The command deploys skydive on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The chart can be customized using the following configuration parameters:

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
| `exporter.enabled`                   | Export netflow data into Object Storage         | `false`                                                    |
| `exporter.write.s3.endpoint`         | Endpoint of the Object Storage to be used       | `http://127.0.0.1:9000`                                    |
| `exporter.write.s3.installLocalMinio`| Install default Minio Object Storage locally    | `true`                                                     |
| `exporter.write.s3.region`           | Object Store region                             | `default`                                                  |
| `exporter.write.s3.use_api_key`      | Use an api key for Object Store autentication   | `false`                                                    |
| `exporter.write.s3.api_key`          | api key for Object Store autentication          | Empty                                                      |
| `exporter.write.s3.access_key`       | access key for Object Store autentication       | `admin`                                                    |
| `exporter.write.s3.secret_key`       | secret key for Object Store autentication       | `admin1234`                                                |
| `exporter.store.bucket`              | bucket name to be used in Object Store          | `default`                                                  |
| `exporter.store.objectPrefix`        | prefix of stroed objects                        | `default`                                                  |
| `exporter.write.s3.iam_endpoint`     | endpoint for api key verification               | `https://iam.cloud.ibm.com/identity/token`                 |


Specify parameters using `--set key=value[,key=value]` argument to `helm install`

Alternatively a YAML file that specifies the values for the parameters can be provided like this:

```bash
helm install --name my-release -f values.yaml stable/skydive
```

### Env

The chart allows definition of extended environment variables to be used by Skydive components. The list of configuration parameters is available on [https://github.com/skydive-project/skydive/blob/master/etc/skydive.yml.default](https://github.com/skydive-project/skydive/blob/master/etc/skydive.yml.default). Use upper-case/underline semantics of a configuration parameter prefixed by `SKYDIVE_` to use as an environment variable. For example, to enable debug add to the deployment .yml file:  
```
env:
  # Enable debug
  - name: SKYDIVE_LOGGING_LEVEL
    value: "DEBUG"
```
Environment variables for SkyDive can also be defined using UI. The Env field should be changed, for example:
```
  [{"name":"SKYDIVE_LOGGING_LEVEL","value":"DEBUG"}]
```

### Topology fabric

The chart allows definition of static interfaces and links to be added to skydive topology view by setting the `analyzer.topology.fabric` parameter. This is useful to define external fabric resources like : TOR, Router, etc.
Details on this parameter field are available under the analyzer.topology.Fabric section in the following link: 
[https://github.com/skydive-project/skydive/blob/master/etc/skydive.yml.default](https://github.com/skydive-project/skydive/blob/master/etc/skydive.yml.default)

### Persistence Volume

Skydive analyzer uses elasticsearch to store data at the `/usr/share/elasticsearch/data` path of the Analyzer container.

The chart mounts a [Persistent Volume](http://kubernetes.io/docs/user-guide/persistent-volumes/) at this location. User needs to create a PV before chart deployed, or enable dynamic volume provisioning in chart configuration.

To make Elasticsearch work properly the user needs to set kernel setting vm.max_map_count to at least 262144 on each worker node. Please check
[https://www.elastic.co/guide/en/elasticsearch/reference/5.5/docker.html](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/docker.html)

### Skydive as netflow collector
Skydive exporter component runs as part of the Skydive analyzer, and stores the collected netflow data into an S3 compatible Object Storage.

1. place the following in a values.yaml file, and set the bucket, objectPrefix, endpoint, region, api_key (or use access key and secret key - then set use_api_key=false) to correct values. Any S3 compatible Object Storage can be used.
```
exporter:
  enabled: true
  store:
    bucket: "default"
    objectPrefix: "default"
  write:
    s3:
      #endpoint is a required value, pointing to a working Object Store endpoint Example value - "http://localhost:9000"
      endpoint: "http://localhost:9000"
      #installLocalMinio should be set to false. If set to true a default minio OS will be installed locally (in a container) for testing purposes only.
      installLocalMinio: false
      region: "default"
      use_api_key: true
      api_key: "api key value"
      access_key: ""
      secret_key: ""
```

2. 
```bash
helm install --name my-release --f vlaues.yaml stable/skydive
```

## Documentation

Skydive documentation can be found here:

* [http://skydive-project.github.io/skydive](http://skydive-project.github.io/skydive)

## Contact and Support

* IRC: #skydive-project on [irc.freenode.net](https://webchat.freenode.net/)
* Mailing list: [https://www.redhat.com/mailman/listinfo/skydive-dev](https://www.redhat.com/mailman/listinfo/skydive-dev)
* Issues: [https://github.com/skydive-project/skydive/issues](https://github.com/skydive-project/skydive/issues)
          [https://github.com/skydive-project/skydive-helm/issues](https://github.com/skydive-project/skydive-helm/issues)
* Slack

Invite : https://slack.skydive.network
Workspace : https://skydive-project.slack.com

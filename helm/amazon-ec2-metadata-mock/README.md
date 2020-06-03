# Amazon EC2 Metadata Mock

Amazon EC2 Metadata Mock(AEMM) Helm chart for Kubernetes. For more information on this project see the project repo at https://github.com/aws/amazon-ec2-metadata-mock.

## Prerequisites

* Kubernetes >= 1.14

## Installing the Chart

The helm chart can be installed from several sources. To install the chart with the release name amazon-ec2-metadata-mock and default configuration, pick a source below:

1. Local chart archive: 
Download the chart archive from the latest release and run 
```sh
helm install amazon-ec2-metadata-mock amazon-ec2-metadata-mock-0.1.0.tgz \
  --namespace default
```

2. Unpacked local chart directory: 
Download the source code or unpack the archive from latest release and run
```sh
helm install amazon-ec2-metadata-mock ./helm/amazon-ec2-metadata-mock \
  --namespace default
```
----
To upgrade an already installed chart named amazon-ec2-metadata-mock:
```sh
helm upgrade amazon-ec2-metadata-mock ./helm/amazon-ec2-metadata-mock \
  --namespace default
```

### Installing the Chart with overridden values for AEMM configuration:

AEMM has an [extensive list of parameters](https://github.com/aws/amazon-ec2-metadata-mock#defaults) that can overridden. For simplicity, a selective list of parameters are configurable using Helm custom `values.yaml` or `--set argument`. To override parameters not listed in `values.yaml` use Kubernetes ConfigMap.

The [configuration](#configuration) section details the selective list of parameters. Alternatively, to retrieve the same information via helm, run:
```sh
helm show values ./helm/amazon-ec2-metadata-mock
```

* Passing a custom values.yaml to helm
```sh
helm install amazon-ec2-metadata-mock ./helm/amazon-ec2-metadata-mock \
  --namespace default -f path/to/myvalues.yaml 
```

* Passing custom values to Helm via CLI arguments
```sh
helm install amazon-ec2-metadata-mock ./helm/amazon-ec2-metadata-mock \
  --namespace default --set aemm.spotItn.instanceAction="stop",aemm.mockDelaySec=120
```

* Passing a config file to AEMM

 1. Create a Kubernetes ConfigMap from a custom AEMM configuration file:
See [Readme](https://github.com/aws/amazon-ec2-metadata-mock#configuration) to learn more about AEMM configuration. [Here](https://github.com/aws/amazon-ec2-metadata-mock/blob/master/test/e2e/testdata/output/aemm-config-used.json) is a reference config file to create your own `aemm-config.json`

    Note:
    * AEMM's native config `aemm.server.port` needs to be a fixed value (1338) to be able to run AEMM as a K8s service. So, overriding the `aemm.server.port` in the custom config file will work only when AEMM is accessed via the pod directly. To access the AEMM K8s service on a custom port, override `servicePort` (which is a Helm config).

    * The `configMapFileName` is used to mount the configMap on the containers running AEMM. The default file name is `aemm-config.json`. If a non-default file name was used to create the configMap, override `configMapFileName` in order for AEMM to be able to access it.

    ```sh
    kubectl create configmap aemm-config-map --from-file path/to/aemm-config.json
    ```

 2. Create `myvalues.yaml` with overridden value for configMap:
```yaml
configMap: "aemm-config-map"
servicePort: 1550
```

 3. Install AEMM with override:
```sh
helm install amazon-ec2-metadata-mock ./helm/amazon-ec2-metadata-mock \
  --namespace default -f path/to/myvalues.yaml 
```

## Making a HTTP request to the AEMM server running on a pod

1. Access AEMM pod / service
    i. Set up port-forwarding to access AEMM on your machine:

    ```sh
    kubectl get pods --namespace default
    ```

    ```sh
    kubectl port-forward pod/<AEMM-pod-name> 1338
    ```

    or

    ```
    kubectl port-forward service/amazon-ec2-metadata-mock 1338
    ```

    ii. Access AEMM from your application using the ClusterIP / DNS of the service or the pod directly.

2. Make the HTTP request

    ```sh
    # From outside the cluster:

    curl http://localhost:1338/latest/meta-data/spot/instance-action
    {
        "instance-action": "terminate",
        "time": "2020-05-04T18:11:37Z"
    }
    ```
    or
    ```sh
    # From inside the cluster:
    # ClusterIP and port for the service should be availble in the application pod's environment, if it was created after the AEMM service.

    curl http://$AMAZON_EC2_METADATA_MOCK_SERVICE_HOST:$AMAZON_EC2_METADATA_MOCK_SERVICE_PORT/latest/meta-data/spot/instance-action
    {
        "instance-action": "terminate",
        "time": "2020-05-04T18:11:37Z"
    }
    ```
    or
    ```sh
    # From inside the cluster:

    curl http://amazon-ec2-metadata-mock.default.svc.cluster.local:1338/latest/meta-data/spot/instance-action
    {
        "instance-action": "terminate",
        "time": "2020-05-04T18:11:37Z"
    }
    ```

## Uninstalling the Chart

To uninstall/delete the `amazon-ec2-metadata-mock` release:
```sh
helm uninstall amazon-ec2-metadata-mock
```
The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following tables lists the configurable parameters of the chart and their default values.

Parameter | Description | Default
--- | --- | --- 
`image.repository` | image repository | `amazon/amazon-ec2-metadata-mock` 
`image.tag` | image tag | `<VERSION>` 
`image.pullPolicy` | image pull policy | `IfNotPresent`
`nameOverride` | override for the name of the Helm Chart (default, if not overridden: `amazon-ec2-metadata-mock`) | `""`
`fullnameOverride` | override for the name of the application (default, if not overridden: `amazon-ec2-metadata-mock`) | `""`
`nodeSelector` | tells the DaemonSet where to place the amazon-ec2-metadata-mock pods. | `{}`, meaning every node will receive a pod
`podAnnotations` | annotations to add to each pod | `{}`
`updateStrategy` | the update strategy for a DaemonSet | `RollingUpdate`
`rbac.pspEnabled` | if `true`, create and use a restricted pod security policy | `false`
`serviceAccount.create` | if `true`, create a new service account | `true`
`serviceAccount.name` | service account to be used | `amazon-ec2-metadata-mock-service-account`
`serviceAccount.annotations` | specifies the annotations for service account | `{}`
`securityContext.runAsUserID` | user ID to run the container | `1000`
`securityContext.runAsGroupID` | group ID to run the container | `1000` 
`namespace` | Kubernetes namespace to use for AEMM pods | `default`
`configMap` | name of the Kubernetes ConfigMap to use to pass a config file for AEMM overrides | `""`
`configMapFileName` | name of the file used to create the Kubernetes ConfigMap | `aemm-config.json`
`servicePort` | port to run AEMM K8s Service on | `1338`

NOTE: A selective list of AEMM parameters are configurable via Helm CLI and values.yaml file. 
Use the [Kubernetes ConfigMap option](#installing-the-chart-with-overridden-values-for-aemm-configuration) to configure [other AEMM parameters](https://github.com/aws/amazon-ec2-metadata-mock/blob/master/test/e2e/testdata/output/aemm-config-used.json). 

Parameter | Description | Default in Helm | Default AEMM configuration
--- | --- | --- | ---
`aemm.server.hostname` | hostname to run AEMM on | `""`, in order to listen on all available interfaces e.g. ClusterIP | `localhost`
`aemm.mockDelaySec` | mock delay in seconds, relative to the start time of AEMM | `0` | `0`
`aemm.imdsv2` | if true, IMDSv2 only works | `false` | `false`, meaning both IMDSv1/v2 work 
`aemm.spotItn.instanceAction` | instance action in the spot interruption notice | `""` | `terminate`
`aemm.spotItn.terminationTime` | termination time in the spot interruption notice | `""` | HTTP request time + 2 minutes
`aemm.scheduledEvents.code` | event code in the scheduled event | `""` | `system-reboot`
`aemm.scheduledEvents.notAfter` | the latest end time for the scheduled event | `""` | Start time of AEMM  + 7 days
`aemm.scheduledEvents.notBefore` | the earliest start time for the scheduled event | `""` | Start time of AEMM
`aemm.scheduledEvents.notBeforeDeadline` | the deadline for starting the event | `""` | Start time of AEMM  + 9 days
`aemm.scheduledEvents.state` | state of the scheduled event | `""` | `active`
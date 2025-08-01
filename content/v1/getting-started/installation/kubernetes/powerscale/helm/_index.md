---
title: "Installation Guide"
linkTitle: "Helm"
no_list: true
description: Helm Installation
weight: 3
---
1. Set up a Kubernetes cluster following the official documentation.
2. Complete the base installation.
3. Proceed with module installation.
### Install Helm 3.0

Install Helm 3.0 on the master node before you install the CSI Driver for PowerScale.

**Steps**

  Run the command to install Helm 3.0.

  ```bash
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```

{{< accordion id="One" title="CSM Installation Wizard" >}}
            {{<include  file="content/v1/getting-started/installation/installationwizard/helm.md" Var="powerscale" hideIds="2">}}
{{< /accordion >}}
<br>
{{< accordion id="Two" title="Base Install" markdown="true" >}}  

## Prerequisites

The following are requirements to be met before installing the CSI Driver for PowerScale:

- Install Kubernetes or OpenShift (see [supported versions](../../../../../concepts/csidriver/#features-and-capabilities))
- Install Helm 3
- Mount propagation is enabled on container runtime that is being used
- `nfs-utils` package must be installed on nodes that will mount volumes
- If using Snapshot feature, satisfy all Volume Snapshot requirements
- If enabling CSM for Authorization, please refer to the [Authorization deployment steps](../helm/csm-modules/authorizationv2-0/) first
- If enabling CSM for Replication, please refer to the [Replication deployment steps](../helm/csm-modules/replication/) first
- If enabling CSM for Resiliency, please refer to the [Resiliency deployment steps](../helm/csm-modules/resiliency/) first

### (Optional) Volume Snapshot Requirements

  For detailed snapshot setup procedure, [click here.](../../../../../concepts/snapshots/#helm-optional-volume-snapshot-requirements)

### (Optional) Volume Health Monitoring

Volume Health Monitoring feature is optional and by default this feature is disabled for drivers when installed via helm.

If enabled capacity metrics (used & free capacity, used & free inodes) for PowerScale PV will be expose in Kubernetes metrics API.

To enable this feature, add the below block to the driver manifest before installing the driver. This ensures to install external
health monitor sidecar. To get the volume health state value under controller should be set to true as seen below. To get the
volume stats value under node should be set to true.

```yaml
controller:
  healthMonitor:
    # enabled: Enable/Disable health monitor of CSI volumes
    # Allowed values:
    #   true: enable checking of health condition of CSI volumes
    #   false: disable checking of health condition of CSI volumes
    # Default value: None
    enabled: false
    # interval: Interval of monitoring volume health condition
    # Allowed values: Number followed by unit (s,m,h)
    # Examples: 60s, 5m, 1h
    # Default value: 60s
    interval: 60s
node:
  healthMonitor:
    # enabled: Enable/Disable health monitor of CSI volumes- volume usage, volume condition
    # Allowed values:
    #   true: enable checking of health condition of CSI volumes
    #   false: disable checking of health condition of CSI volumes
    # Default value: None
    enabled: false
```

*NOTE*: To enable this feature to existing driver OR enable this feature while upgrading the driver versions,
follow either of the way.

1. Reinstall of Driver
2. Upgrade the driver smoothly with "--upgrade" option

### (Optional) Replication feature Requirements

Applicable only if you decided to enable the Replication feature in `values.yaml`

```yaml
replication:
  enabled: true
```

#### Replication CRD's

The CRDs for replication can be obtained and installed from the csm-replication project on Github. Use `csm-replication/deploy/replicationcrds.all.yaml` located in the csm-replication git repo for the installation.

CRDs should be configured during replication prepare stage with repctl as described in [install-repctl](../helm/csm-modules/replication/install-repctl)

## Install Driver

**Steps**

1. Run `git clone -b {{< version-v1 key="PScale_latestVersion" >}} https://github.com/dell/csi-powerscale.git` to clone the git repository.
2. Ensure that you have created the namespace where you want to install the driver. You can run `kubectl create namespace isilon` to create a new one. The use of "isilon"  as the namespace is just an example. You can choose any name for the namespace.
3. Collect information from the PowerScale Systems like IP address, IsiPath, username, and password. Make a note of the value for these parameters as they must be entered in the *secret.yaml*.

   **Note**: The 'clusterName' serves as a logical, unique identifier for the array that should remain unchanged once it is included in the volume handle. Altering this identifier is not advisable, as it would result in the failure of all operations associated with the volume that was created earlier.

4. Download `wget -O my-isilon-settings.yaml https://raw.githubusercontent.com/dell/helm-charts/csi-isilon-2.14.0/charts/csi-isilon/values.yaml` into `cd ../dell-csi-helm-installer` to customize settings for installation.
5. Edit *my-isilon-settings.yaml* to set the following parameters for your installation:
   The following table lists the primary configurable parameters of the PowerScale driver Helm chart and their default values. More detailed information can be
   found in the  [`values.yaml`](https://github.com/dell/helm-charts/blob/csi-isilon-2.14.0/charts/csi-isilon/values.yaml) file in this repository.

  <ul>
   {{< collapse id="2" title="Parameters">}}
   | Parameter | Description | Required | Default |
   | --------- | ----------- | -------- |-------- |
   |<div style="text-align: left"> images |<div style="text-align: left"> List all the images used by the CSI driver and CSM. If you use a private repository, change the registries accordingly. | Yes | "" |
   |<div style="text-align: left"> logLevel |<div style="text-align: left"> CSI driver log level | No | "debug" |
   |<div style="text-align: left"> certSecretCount |<div style="text-align: left"> Defines the number of certificate secrets, which the user is going to create for SSL authentication. (isilon-cert-0..isilon-cert-(n-1)); Minimum value should be 1.| Yes | 1 |
   |<div style="text-align: left"> [allowedNetworks](../../../../../concepts/csidriver/features/powerscale/#support-custom-networks-for-nfs-io-traffic) |<div style="text-align: left"> Defines the list of networks that can be used for NFS I/O traffic, CIDR format must be used. | No | [ ] |
   |<div style="text-align: left"> maxIsilonVolumesPerNode |<div style="text-align: left"> Defines the default value for a maximum number of volumes that the controller can publish to the node. If the value is zero CO SHALL decide how many volumes of this type can be published by the controller to the node. This limit is applicable to all the nodes in the cluster for which node label 'max-isilon-volumes-per-node' is not set. | Yes | 0 |
   |<div style="text-align: left"> imagePullPolicy |<div style="text-align: left"> Defines the policy to determine if the image should be pulled prior to starting the container | Yes | IfNotPresent |
   |<div style="text-align: left"> verbose |<div style="text-align: left"> Indicates what content of the OneFS REST API message should be logged in debug level logs | Yes | 1 |
   |<div style="text-align: left"> kubeletConfigDir |<div style="text-align: left"> Specify kubelet config dir path | Yes | "/var/lib/kubelet" |
   |<div style="text-align: left"> enableCustomTopology |<div style="text-align: left"> Indicates PowerScale FQDN/IP which will be fetched from node label and the same will be used by controller and node pod to establish a connection to Array. This requires enableCustomTopology to be enabled. | No | false |
   |<div style="text-align: left"> fsGroupPolicy |<div style="text-align: left"> Defines which FS Group policy mode to be used, Supported modes `None, File and ReadWriteOnceWithFSType` | No | "ReadWriteOnceWithFSType" |
   |<div style="text-align: left"> storageCapacity.enabled |<div style="text-align: left"> Enable/Disable storage capacity tracking | No | true |
   |<div style="text-align: left"> storageCapacity.pollInterval |<div style="text-align: left"> Configure how often the driver checks for changed capacity | No | 5m |
   |<div style="text-align: left"> podmonAPIPort |<div style="text-align: left"> Defines the port which csi-driver will use within the cluster to support podmon | No | 8083 |
   |<div style="text-align: left"> maxPathLen |<div style="text-align: left"> Defines the maximum length of path for a volume | No | 192 |
   |<div style="text-align: left"> ***controller*** |<div style="text-align: left"> Configure controller pod specific parameters | | |
   |<div style="text-align: left"> controllerCount |<div style="text-align: left"> Defines the number of csi-powerscale controller pods to deploy to the Kubernetes release| Yes | 2 |
   |<div style="text-align: left"> volumeNamePrefix |<div style="text-align: left"> Defines a string prefix for the names of PersistentVolumes created | Yes | "k8s" |
   |<div style="text-align: left"> snapshot.enabled |<div style="text-align: left"> Enable/Disable volume snapshot feature | Yes | true |
   |<div style="text-align: left"> snapshot.snapNamePrefix |<div style="text-align: left"> Defines a string prefix for the names of the Snapshots created | Yes | "snapshot" |
   |<div style="text-align: left"> resizer.enabled |<div style="text-align: left"> Enable/Disable volume expansion feature | Yes | true |
   |<div style="text-align: left"> healthMonitor.enabled |<div style="text-align: left"> Enable/Disable health monitor of CSI volumes- volume status, volume condition | Yes | false |
   |<div style="text-align: left"> healthMonitor.interval |<div style="text-align: left"> Interval of monitoring volume health condition | Yes | 60s |
   |<div style="text-align: left"> nodeSelector |<div style="text-align: left"> Define node selection constraints for pods of controller deployment | No | |
   |<div style="text-align: left"> tolerations |<div style="text-align: left"> Define tolerations for the controller deployment, if required | No | |
   |<div style="text-align: left"> leader-election-lease-duration |<div style="text-align: left"> Duration, that non-leader candidates will wait to force acquire leadership | No | 20s |
   |<div style="text-align: left"> leader-election-renew-deadline   |<div style="text-align: left"> Duration, that the acting leader will retry refreshing leadership before giving up  | No | 15s |
   |<div style="text-align: left"> leader-election-retry-period   |<div style="text-align: left"> Duration, the LeaderElector clients should wait between tries of actions  | No | 5s |
   |<div style="text-align: left"> ***node*** |<div style="text-align: left"> Configure node pod specific parameters | | |
   |<div style="text-align: left"> nodeSelector |<div style="text-align: left"> Define node selection constraints for pods of node daemonset | No | |
   |<div style="text-align: left"> tolerations |<div style="text-align: left"> Define tolerations for the node daemonset, if required | No | |
   |<div style="text-align: left"> dnsPolicy |<div style="text-align: left"> Define the DNS Policy of the Node service | Yes | ClusterFirstWithHostNet |
   |<div style="text-align: left"> healthMonitor.enabled |<div style="text-align: left"> Enable/Disable health monitor of CSI volumes- volume usage, volume condition | Yes | false |
   |<div style="text-align: left"> ***PLATFORM ATTRIBUTES*** | | | |
   |<div style="text-align: left"> endpointPort |<div style="text-align: left"> Define the HTTPs port number of the PowerScale OneFS API server. If authorization is enabled, endpointPort should be the HTTPS localhost port that the authorization sidecar will listen on. This value acts as a default value for endpointPort, if not specified for a cluster config in secret. | No | 8080 |
   |<div style="text-align: left"> skipCertificateValidation |<div style="text-align: left"> Specify whether the PowerScale OneFS API server's certificate chain and hostname must be verified. This value acts as a default value for skipCertificateValidation, if not specified for a cluster config in secret. | No | true |
   |<div style="text-align: left"> isiAuthType |<div style="text-align: left"> Indicates the authentication method to be used. If set to 1 then it follows as session-based authentication else basic authentication. If authorization.enabled=true, this value must be set to 1. | No | 0 |
   |<div style="text-align: left"> isiAccessZone |<div style="text-align: left"> Define the name of the access zone a volume can be created in. If storageclass is missing with AccessZone parameter, then value of isiAccessZone is used for the same. | No | System |
   |<div style="text-align: left"> enableQuota |<div style="text-align: left"> Indicates whether the provisioner should attempt to set (later unset) quota on a newly provisioned volume. This requires SmartQuotas to be enabled.| No | true |
   |<div style="text-align: left"> isiPath |<div style="text-align: left"> Define the base path for the volumes to be created on PowerScale cluster. This value acts as a default value for isiPath, if not specified for a cluster config in secret| No | /ifs/data/csi |
   |<div style="text-align: left"> ignoreUnresolvableHosts |<div style="text-align: left"> Allows new host to add to existing export list though any of the existing hosts from the same exports are unresolvable/doesn't exist anymore. | No | false |
   |<div style="text-align: left"> noProbeOnStart |<div style="text-align: left"> Define whether the controller/node plugin should probe all the PowerScale clusters during driver initialization | No | false |
   |<div style="text-align: left"> autoProbe |<div style="text-align: left"> Specify if automatically probe the PowerScale cluster if not done already during CSI calls | No | true |
   |<div style="text-align: left"> **authorization** |<div style="text-align: left"> [Authorization](../helm/csm-modules/authorizationv2-0/) is an optional feature to apply credential shielding of the backend PowerScale. | - | - |
   |<div style="text-align: left"> enabled                  |<div style="text-align: left"> A boolean that enables/disables authorization feature. If enabled, isiAuthType must be set to 1. |  No      |   false   |
   |<div style="text-align: left"> proxyHost |<div style="text-align: left"> Hostname of the csm-authorization server. | No | Empty |
   |<div style="text-align: left"> skipCertificateValidation |<div style="text-align: left"> A boolean that enables/disables certificate validation of the csm-authorization proxy server. | No | true |
   |<div style="text-align: left"> **podmon**               |<div style="text-align: left"> [Podmon](../helm/csm-modules/resiliency/) is an optional feature to enable application pods to be resilient to node failure.  |  -        |  -       |
   |<div style="text-align: left"> enabled                  |<div style="text-align: left"> A boolean that enables/disables podmon feature. |  No      |   false   |
    
   *NOTE:*

   - ControllerCount parameter value must not exceed the number of nodes in the Kubernetes cluster. Otherwise, some of the controller pods remain in a "Pending" state till new nodes are available for scheduling. The installer exits with a WARNING on the same.
   - Whenever the *certSecretCount* parameter changes in *my-isilon-setting.yaml* user needs to reinstall the driver.
   - In order to enable authorization, there should be an authorization proxy server already installed.
   - If you are using custom images, update each attributes under the *images* field in *my-isilon-setting.yaml* to make sure that they are pointing to the correct image repository and version.
   {{< /collapse >}}
   </ul>

6. Edit following parameters in samples/secret/secret.yaml file and update/add connection/authentication information for one or more PowerScale clusters. If replication feature is enabled, ensure the secret includes all the PowerScale clusters involved in replication.

<ul>
{{< collapse id="3" title="Parameters">}}
   | Parameter | Description | Required | Default |
   | --------- | ----------- | -------- |-------- |
   |<div style="text-align: left"> clusterName |<div style="text-align: left"> Logical name of PoweScale cluster against which volume CRUD operations are performed through this secret. | Yes | - |
   |<div style="text-align: left"> username |<div style="text-align: left"> username for connecting to PowerScale OneFS API server | Yes | - |
   |<div style="text-align: left"> password |<div style="text-align: left"> password for connecting to PowerScale OneFS API server | Yes | - |
   |<div style="text-align: left"> endpoint |<div style="text-align: left"> HTTPS endpoint of the PowerScale OneFS API server. If authorization is enabled, endpoint should be the HTTPS localhost endpoint that the authorization sidecar will listen on | Yes | - |
   |<div style="text-align: left"> isDefault |<div style="text-align: left"> Indicates if this is a default cluster (would be used by storage classes without ClusterName parameter). Only one of the cluster config should be marked as default. | No | false |
   |<div style="text-align: left"> ***Optional parameters*** |<div style="text-align: left"> Following parameters are Optional. If specified will override default values from values.yaml. |
   |<div style="text-align: left"> skipCertificateValidation |<div style="text-align: left"> Specify whether the PowerScale OneFS API server's certificate chain and hostname must be verified. | No | default value from values.yaml |
   |<div style="text-align: left"> ignoreUnresolvableHosts |<div style="text-align: left"> Allows new host to add to existing export list though any of the existing hosts from the same exports are unresolvable/doesn't exist anymore. | No | default value from values.yaml |
   |<div style="text-align: left"> endpointPort |<div style="text-align: left"> Specify the HTTPs port number of the PowerScale OneFS API server | No | default value from values.yaml |
   |<div style="text-align: left"> isiPath |<div style="text-align: left"> The base path for the volumes to be created on PowerScale cluster. Note: IsiPath parameter in storageclass, if present will override this attribute. | No | default value from values.yaml |
   |<div style="text-align: left"> mountEndpoint |<div style="text-align: left"> Endpoint of the PowerScale OneFS API server, for example, 10.0.0.1. This must be specified if [CSM-Authorization](https://github.com/dell/karavi-authorization) is enabled. | No | - |
{{< /collapse >}} 

### User privileges

   The username specified in *secret.yaml* must be from the authentication providers of PowerScale. The user must have enough privileges to perform the actions. The suggested privileges are as follows:

   | Privilege              | Type       |
   | ---------------------- | ---------- |
   | ISI_PRIV_LOGIN_PAPI    | Read Only  |
   | ISI_PRIV_NFS           | Read Write |
   | ISI_PRIV_QUOTA         | Read Write |
   | ISI_PRIV_SNAPSHOT      | Read Write |
   | ISI_PRIV_IFS_RESTORE   | Read Only  |
   | ISI_PRIV_NS_IFS_ACCESS | Read Only  |
   | ISI_PRIV_IFS_BACKUP    | Read Only  |
   | ISI_PRIV_AUTH_ZONES    | Read Only  |
   | ISI_PRIV_SYNCIQ        | Read Write |
   | ISI_PRIV_STATISTICS    | Read Only  |

Create isilon-creds secret using the following command:
  <br/> `kubectl create secret generic isilon-creds -n isilon --from-file=config=secret.yaml`

   *NOTE:*
   - If any key/value is present in all *my-isilon-settings.yaml*, *secret*, and storageClass, then the values provided in storageClass parameters take precedence.
   - The user has to validate the yaml syntax and array-related key/values while replacing or appending the isilon-creds secret. The driver will continue to use previous values in case of an error found in the yaml file.
   - For the key isiIP/endpoint, the user can give either IP address or FQDN. Also, the user can prefix 'https' (For example, https://192.168.1.1) with the value.
   - The *isilon-creds* secret has a *mountEndpoint* parameter which should only be updated and used when [Authorization](../../../../../concepts/authorization) is enabled.
</ul>

7. Install OneFS CA certificates by following the instructions from the next section, if you want to validate OneFS API server's certificates. If not, create an empty secret using the following command and an empty secret must be created for the successful installation of CSI Driver for PowerScale.
    ```bash
    kubectl create -f empty-secret.yaml
    ```
   This command will create a new secret called `isilon-certs-0` in isilon namespace.

8. Install the driver using `csi-install.sh` bash script and default yaml by running
    ```bash
    cd dell-csi-helm-installer && wget -O my-isilon-settings.yaml https://raw.githubusercontent.com/dell/helm-charts/csi-isilon-2.13.0/charts/csi-isilon/values.yaml &&
    ./csi-install.sh --namespace isilon --values  my-isilon-settings.yaml --helm-charts-version <version>
    ```

*NOTE:*
- The parameter `--helm-charts-version` is optional and if you do not specify the flag, by default the `csi-install.sh` script will clone the version of the helm chart that is specified in the driver's [csi-install.sh](https://github.com/dell/csi-powerscale/blob/main/dell-csi-helm-installer/csi-install.sh#L16) file. If you wish to install the driver using a different version of the helm chart, you need to include this flag. Also, remember to delete the `helm-charts` repository present in the `csi-powerscale` directory if it was cloned before.

## Certificate validation for OneFS REST API calls

The CSI driver exposes an install parameter 'skipCertificateValidation' which determines if the driver
performs client-side verification of the OneFS certificates. The 'skipCertificateValidation' parameter is set to true by default and the driver does not verify the OneFS certificates.

If the 'skipCertificateValidation' is set to false, then the secret isilon-certs must contain the CA certificate for OneFS.
If this secret is an empty secret, then the validation of the certificate fails, and the driver fails to start.

If the 'skipCertificateValidation' parameter is set to false and a previous installation attempt to create the empty secret, then this secret must be deleted and re-created using the CA certs. If the OneFS certificate is self-signed, then perform the following steps:

### Procedure

1. To fetch the certificate, run
```bash
openssl s_client -showcerts -connect [OneFS IP] </dev/null 2>/dev/null | openssl x509 -outform PEM > ca_cert_0.pem
```
2. To create the certs secret, run
```bash
kubectl create secret generic isilon-certs-0 --from-file=cert-0=ca_cert_0.pem -n isilon
```
3. Use the following command to replace the secret <br/>
```bash
kubectl create secret generic isilon-certs-0 -n isilon --from-file=cert-0=ca_cert_0.pem -o yaml --dry-run | kubectl replace -f -
```

**NOTES:**

1. The OneFS IP can be with or without a port, depends upon the configuration of OneFS API server.
2. The commands are based on the namespace 'isilon'
3. It is highly recommended that ca_cert.pem file(s) having the naming convention as ca_cert_number.pem (example: ca_cert_0, ca_cert_1), where this number starts from 0 and grows as the number of OneFS arrays grows.
4. The cert secret created out of these pem files must have the naming convention as isilon-certs-number (example: isilon-certs-0, isilon-certs-1, and so on.); The number must start from zero and must grow in incremental order. The number of the secrets created out of pem files should match certSecretCount value in myvalues.yaml or my-isilon-settings.yaml.

### Dynamic update of array details via secret.yaml

CSI Driver for PowerScale now provides supports for Multi cluster. Now users can link the single CSI Driver to multiple OneFS Clusters by updating *secret.yaml*. Users can now update the isilon-creds secret by editing the *secret.yaml* and executing the following command

```bash
kubectl create secret generic isilon-creds -n isilon --from-file=config=secret.yaml -o yaml --dry-run=client | kubectl replace -f -
```

**Note**: Updating isilon-certs-x secrets is a manual process, unlike isilon-creds. Users have to re-install the driver in case of updating/adding the SSL certificates or changing the certSecretCount parameter.

## Storage Classes

The CSI driver for PowerScale version 1.5 and later, `dell-csi-helm-installer` does not create any storage classes as part of the driver installation. A sample storage class manifest is available at `samples/storageclass/isilon.yaml`. Use this sample manifest to create a storageclass to provision storage; uncomment/ update the manifest as per the requirements.

### What happens to my existing storage classes?

*Upgrading from CSI PowerScale v2.3 driver*:
The storage classes created as part of the installation have an annotation - "helm.sh/resource-policy": keep set. This ensures that even after an uninstall or upgrade, the storage classes are not deleted. You can continue using these storage classes if you wish so.

*NOTE*:

- At least one storage class is required for one array.
- If you uninstall the driver and reinstall it, you can still face errors if any update in the `values.yaml` file leads to an update of the storage class(es):

```bash
    Error: cannot patch "<sc-name>" with kind StorageClass: StorageClass.storage.k8s.io "<sc-name>" is invalid: parameters: Forbidden: updates to parameters are forbidden
```

In case you want to make such updates, ensure to delete the existing storage classes using the `kubectl delete storageclass` command.
Deleting a storage class has no impact on a running Pod with mounted PVCs. You cannot provision new PVCs until at least one storage class is newly created.

>Note: If you continue to use the old storage classes, you may not be able to take advantage of any new storage class parameter supported by the driver.

**Steps to create secondary storage class:**

There are samples storage class yaml files available under `samples/storageclass`.  These can be copied and modified as needed.

1. Copy the `storageclass.yaml` to `second_storageclass.yaml` (This is just an example, you can rename to file you require.)
2. Edit the  `second_storageclass.yaml` yaml file and update following parameters:
- Update the `name` parameter to you require
   ```yaml
    metadata:
      name: isilon-new
   ```
- Cluster name of 2nd array looks like this in the secret file.( Under `/samples/secret/secret.yaml`)
  ```yaml
  - clusterName: "cluster2"
    username: "user name"
    password: "Password"
    endpoint: "10.X.X.X"
    endpointPort: "8080
  ```
- Use same clusterName &#8593; in the  `second_storageclass.yaml`
   ```yaml
    # Optional: true
    ClusterName: "cluster2"
   ```
- *Note*: These are two essential parameters that you need to change in the "second_storageclass.yaml" file and other parameters that you change as required.
3. Save the  `second_storageclass.yaml` file
4. Create your 2nd storage class by using `kubectl`:
    ```bash
    kubectl create -f <path_to_second_storageclass_file>
    ```
5. Use newly created storage class `isilon-new` for volumes to spin up on `cluster2`

    PVC example
   ````yaml
     apiVersion: v1
     kind: PersistentVolumeClaim
     metadata:
       name: test-pvc
     spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
      storageClassName: isilon-new
     ````

## Volume Snapshot Class

Starting CSI PowerScale v1.6, `dell-csi-helm-installer` will not create any Volume Snapshot Class during the driver installation. Sample volume snapshot class manifests are available at `samples/volumesnapshotclass/`. Use these sample manifests to create a volumesnapshotclass for creating volume snapshots; uncomment/ update the manifests as per the requirements.

## Silent Mount Re-tries (v2.6.0)

There are race conditions, when completing the ControllerPublish call to populate the client to volumes export list takes longer time than usual due to background NFS refresh process on OneFS wouldn't have completed at same time, resulted in error:"mount failed" with initial attempts and might log success after few re-tries. This unnecessarily logs false positive "mount failed" error logs and to overcome this scenario Driver does silent mount re-tries attempts after every two sec. (five attempts max) for every NodePublish Call and allows successful mount within five re-tries without logging any mount error messages.
"mount failed" will be logged once these five mount retrial attempts are exhausted and still client is not populated to export list.

Mount Re-tries handles below scenarios:

- Access denied by server while mounting (NFSv3)
- No such file or directory (NFSv4)

*Sample*:

```bash
level=error clusterName=powerscale runid=10 msg="mount failed: exit status 32
mounting arguments: -t nfs -o rw XX.XX.XX.XX:/ifs/data/csi/k8s-ac7b91962d /var/lib/kubelet/pods/9f72096a-a7dc-4517-906c-20697f9d7375/volumes/kubernetes.io~csi/k8s-ac7b91962d/mount
output: mount.nfs: access denied by server while mounting XX.XX.XX.XX:/ifs/data/csi/k8s-ac7b91962d
```
{{< /accordion >}}

{{< accordion id="Three" title="Modules" >}}
{{< cardcontainer >}}
  {{< customcard link1="./csm-modules/authorizationv1-x" image="1" title="Authorization v1.x" >}}
  {{< customcard link1="./csm-modules/authorizationv2-0" image="1" title="Authorization v2.0" >}}
  {{< customcard  link1="./csm-modules/observability" image="1" title="Observability"  >}}
  {{< customcard  link1="./csm-modules/replication" image="1" title="Replication"  >}}
  {{< customcard link1="./csm-modules/resiliency" image="1" title="Resiliency"  >}}
{{< /cardcontainer >}}
{{< /accordion >}}

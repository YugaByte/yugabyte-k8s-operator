
# Yugabyte Kubernetes Operator Documentation

## Overview

The Yugabyte Kubernetes Operator automates the deployment and management of YugabyteDB clusters on Kubernetes. This documentation is intended for Kubernetes clusters version 1.27 and above and includes details on setting up necessary roles and permissions for the service account.

## Prerequisites

- **Kubernetes Cluster**: Version 1.27 or later.
- **Helm**: Version 3 or later.
- **Administrative Access**: Required for the Kubernetes cluster, abilty to create cluster roles, roles, namespaces

## Installation

### Adding the Helm Chart Repository

To add the Yugabyte Helm chart repository:

```shell
helm repo add yugabyte https://charts.yugabyte.com
helm repo update
```

### Installing the Operator with RBAC Permissions

To install the Yugabyte Operator with necessary RBAC permissions:

```shell
kubectl create ns <operator_namespace>
kubectl apply -f crd/concatinated_crds.yaml
helm install -n <operator_namespace> yugabyte-k8s-operator yugabyte/yugabyte-operator --set rbac.create=true
```

This command sets up the necessary Role-Based Access Control (RBAC) permissions, 
including the creation of cluster roles and roles within the service account.

### Verifying the Installation

To verify the installation of the operator:

```shell
kubectl get pods -n <operator_namespace>
[centos@dev-server-anijhawan-4 yugabyte_k8s_operator_chart]$ kubectl get pods -n <operator_namespace> 
NAME                          READY   STATUS    RESTARTS   AGE
chart-1706728534-yugaware-0   3/3     Running   0          26h

```
Check for available releases for yugabyte operator to use. 

```
[centos@dev-server-anijhawan-4 yugabyte-k8s-operator]$ kubectl get releases -n testoperator
NAME                VERSION         STATUS      DOWNLOADED
2.18.6.0-b73        2.18.6.0-b73    Available   true
2.19.3.0-b140       2.19.3.0-b140   Available   true
2.20.1.3-b3         2.20.1.3-b3     Available   true
```






### Service Account

The Yugabyte Operator requires a service account with sufficient permissions to manage resources within the Kubernetes cluster. When installing the operator, ensure that the service account has the necessary roles and cluster roles bound to it.

### Cluster Roles and Roles

- **ClusterRole**: Grants permissions at the cluster-level, necessary for operations that span multiple namespaces or have cluster-wide implications.
- **Role**: Grants permissions within a specific namespace, used for namespace-specific operations.

The operator chart, when installed with `rbac.create=true`, will automatically create appropriate ClusterRoles and Roles.

## Custom Resource Definitions (CRDs)

The operator supports various CRDs for managing YugabyteDB clusters, software releases, backups, and restores. Detailed configurations for each CRD are available in the operator's documentation or the CRD YAML files.

## Getting Started


## Example CRs
To create a universe:
```
apiVersion: operator.yugabyte.io/v1alpha1                                                               
kind: YBUniverse                                                                                        
metadata:                                                                                               
  name: operator-universe-demo                                                                         
spec:                                                                                                                                                                               
  numNodes:    3                                                                                        
  replicationFactor:  1                                                                                 
  enableYSQL: true                                                                                      
  enableNodeToNodeEncrypt: true                                                                         
  enableClientToNodeEncrypt: true                                                                       
  enableLoadBalancer: false 
  ybSoftwareVersion: "2.20.1.3-b3" 
  enableYSQLAuth: false                                                                                 
  enableYCQL: true                                                                                      
  enableYCQLAuth: false                                                                                 
  gFlags:                                                                                               
    tserverGFlags: {}                                                                                   
    masterGFlags: {}                                                                                    
  deviceInfo:                                                                                           
    volumeSize: 400                                                                                     
    numVolumes: 1                                                                                       
    storageClass: "yb-standard"                                                                         
  kubernetesOverrides:                                                                                  
    resource:                                                                                           
      master:                                                                                           
        requests:                                                                                       
          cpu: 2                                                                                        
          memory: 8Gi                                                                                   
        limits:                                                                                         
          cpu: 3                                                                                        
          memory: 8Gi 
```

To add a new software release to use of yugabyteDB
```
apiVersion: operator.yugabyte.io/v1alpha1
kind: Release
metadata:
  name: example-release7
spec:
  config:
    version: "2.19.3.0-b140"
    downloadConfig:
      http:
        paths:
          helmChart: "https://charts.yugabyte.com/yugabyte-2.19.3.tgz" 
```

To create a backup

We first need to create a storage config that describes where the backup will be stored. 
```
apiVersion: operator.yugabyte.io/v1alpha1
kind: StorageConfig
metadata:
  name: sco
spec:
  config_type: STORAGE_S3
  data:
    AWS_ACCESS_KEY_ID: __AWS__ACCESS_KEY 
    AWS_SECRET_ACCESS_KEY: __AWS_SECRET_KEY
    BACKUP_LOCATION:  s3://example-backup-location
```

To create a backup that refers to the above storage config we use the backup CR
```
apiVersion: operator.yugabyte.io/v1alpha1
kind: Backup
metadata:
  name: my-backup
spec:
  backupType: PGSQL_TABLE_TYPE 
  sse: true
  storageConfig: "sco"
  universe: "operator-universe919"
  tableByTableBackup: false
  keyspaceTableList: []
  timeBeforeDelete: 0
```

To Restore the backup from the above backup to an existing universe we use

```
apiVersion: operator.yugabyte.io/v1alpha1
kind: RestoreJob
metadata:
  name: example-restore-job
spec:
  actionType: RESTORE
  universe: example-universe
  storageConfig: example-storage-config
  backupType: YQL_TABLE_TYPE
  keyspaceTableList:
    - keyspace1.table1
    - keyspace2.table2
  backupStorageLocation: example-backup-location
```
## Testing
We have tested this operator on GKE, EKS, AKS, it needs the following labels defined on the nodes to
work correctly. 
          ```
          failure-domain.beta.kubernetes.io/region: us-west1
          failure-domain.beta.kubernetes.io/zone: us-west1-b
          ```

## Additional Information

- Ensure Kubernetes cluster version is 1.27 or higher.
- CRDs are located in a file named `./crd/concatenated_crd.yaml`
- The operator is provided as a Helm chart.

## Support

For assistance or issues related to the Yugabyte Kubernetes Operator, please reach out to us on slack 
https://communityinviter.com/apps/yugabyte-db/register
```

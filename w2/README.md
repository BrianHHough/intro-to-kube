# Week 2 Extra Notes

## Step 3: Persistent Volumes

You can create the objects by running the snippet below.

`kubectl apply -f postgres-persist.yml`

Hint: You may want to run the below to find where exactly to add these specifications in your YAML file:
`kubectl explain pv.spec --recursive` 
and 
`kubectl explain pvc.spec --recursive` 

```bash
$ kubectl explain pv.spec --recursive

KIND:       PersistentVolume
VERSION:    v1

FIELD: spec <PersistentVolumeSpec>

DESCRIPTION:
    spec defines a specification of a persistent volume owned by the cluster.
    Provisioned by an administrator. More info:
    https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistent-volumes
    PersistentVolumeSpec is the specification of a persistent volume.
    
FIELDS:
  accessModes   <[]string>
  awsElasticBlockStore  <AWSElasticBlockStoreVolumeSource>
    fsType      <string>
    partition   <integer>
    readOnly    <boolean>
    volumeID    <string> -required-
  azureDisk     <AzureDiskVolumeSource>
    cachingMode <string>
    diskName    <string> -required-
    diskURI     <string> -required-
    fsType      <string>
    kind        <string>
    readOnly    <boolean>
  azureFile     <AzureFilePersistentVolumeSource>
    readOnly    <boolean>
    secretName  <string> -required-
    secretNamespace     <string>
    shareName   <string> -required-
  capacity      <map[string]Quantity>
  cephfs        <CephFSPersistentVolumeSource>
    monitors    <[]string> -required-
    path        <string>
    readOnly    <boolean>
    secretFile  <string>
    secretRef   <SecretReference>
      name      <string>
      namespace <string>
    user        <string>
  cinder        <CinderPersistentVolumeSource>
    fsType      <string>
    readOnly    <boolean>
    secretRef   <SecretReference>
      name      <string>
      namespace <string>
    volumeID    <string> -required-
  claimRef      <ObjectReference>
    apiVersion  <string>
    fieldPath   <string>
    kind        <string>
    name        <string>
    namespace   <string>
    resourceVersion     <string>
    uid <string>
  csi   <CSIPersistentVolumeSource>
    controllerExpandSecretRef   <SecretReference>
      name      <string>
      namespace <string>
    controllerPublishSecretRef  <SecretReference>
      name      <string>
      namespace <string>
    driver      <string> -required-
    fsType      <string>
    nodeExpandSecretRef <SecretReference>
      name      <string>
      namespace <string>
    nodePublishSecretRef        <SecretReference>
      name      <string>
      namespace <string>
    nodeStageSecretRef  <SecretReference>
      name      <string>
      namespace <string>
    readOnly    <boolean>
    volumeAttributes    <map[string]string>
    volumeHandle        <string> -required-
  fc    <FCVolumeSource>
    fsType      <string>
    lun <integer>
    readOnly    <boolean>
    targetWWNs  <[]string>
    wwids       <[]string>
  flexVolume    <FlexPersistentVolumeSource>
    driver      <string> -required-
    fsType      <string>
    options     <map[string]string>
    readOnly    <boolean>
    secretRef   <SecretReference>
      name      <string>
      namespace <string>
  flocker       <FlockerVolumeSource>
    datasetName <string>
    datasetUUID <string>
  gcePersistentDisk     <GCEPersistentDiskVolumeSource>
    fsType      <string>
    partition   <integer>
    pdName      <string> -required-
    readOnly    <boolean>
  glusterfs     <GlusterfsPersistentVolumeSource>
    endpoints   <string> -required-
    endpointsNamespace  <string>
    path        <string> -required-
    readOnly    <boolean>
  hostPath      <HostPathVolumeSource>
    path        <string> -required-
    type        <string>
  iscsi <ISCSIPersistentVolumeSource>
    chapAuthDiscovery   <boolean>
    chapAuthSession     <boolean>
    fsType      <string>
    initiatorName       <string>
    iqn <string> -required-
    iscsiInterface      <string>
    lun <integer> -required-
    portals     <[]string>
    readOnly    <boolean>
    secretRef   <SecretReference>
      name      <string>
      namespace <string>
    targetPortal        <string> -required-
  local <LocalVolumeSource>
    fsType      <string>
    path        <string> -required-
  mountOptions  <[]string>
  nfs   <NFSVolumeSource>
    path        <string> -required-
    readOnly    <boolean>
    server      <string> -required-
  nodeAffinity  <VolumeNodeAffinity>
    required    <NodeSelector>
      nodeSelectorTerms <[]NodeSelectorTerm> -required-
        matchExpressions        <[]NodeSelectorRequirement>
          key   <string> -required-
          operator      <string> -required-
          values        <[]string>
        matchFields     <[]NodeSelectorRequirement>
          key   <string> -required-
          operator      <string> -required-
          values        <[]string>
  persistentVolumeReclaimPolicy <string>
  photonPersistentDisk  <PhotonPersistentDiskVolumeSource>
    fsType      <string>
    pdID        <string> -required-
  portworxVolume        <PortworxVolumeSource>
    fsType      <string>
    readOnly    <boolean>
    volumeID    <string> -required-
  quobyte       <QuobyteVolumeSource>
    group       <string>
    readOnly    <boolean>
    registry    <string> -required-
    tenant      <string>
    user        <string>
    volume      <string> -required-
  rbd   <RBDPersistentVolumeSource>
    fsType      <string>
    image       <string> -required-
    keyring     <string>
    monitors    <[]string> -required-
    pool        <string>
    readOnly    <boolean>
    secretRef   <SecretReference>
      name      <string>
      namespace <string>
    user        <string>
  scaleIO       <ScaleIOPersistentVolumeSource>
    fsType      <string>
    gateway     <string> -required-
    protectionDomain    <string>
    readOnly    <boolean>
    secretRef   <SecretReference> -required-
      name      <string>
      namespace <string>
    sslEnabled  <boolean>
    storageMode <string>
    storagePool <string>
    system      <string> -required-
    volumeName  <string>
  storageClassName      <string>
  storageos     <StorageOSPersistentVolumeSource>
    fsType      <string>
    readOnly    <boolean>
    secretRef   <ObjectReference>
      apiVersion        <string>
      fieldPath <string>
      kind      <string>
      name      <string>
      namespace <string>
      resourceVersion   <string>
      uid       <string>
    volumeName  <string>
    volumeNamespace     <string>
  volumeMode    <string>
  vsphereVolume <VsphereVirtualDiskVolumeSource>
    fsType      <string>
    storagePolicyID     <string>
    storagePolicyName   <string>
    volumePath  <string> -required-
```


```bash
$ kubectl explain pvc.spec --recursive

KIND:       PersistentVolumeClaim
VERSION:    v1

FIELD: spec <PersistentVolumeClaimSpec>

DESCRIPTION:
    spec defines the desired characteristics of a volume requested by a pod
    author. More info:
    https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims
    PersistentVolumeClaimSpec describes the common attributes of storage devices
    and allows a Source for provider-specific attributes
    
FIELDS:
  accessModes   <[]string>
  dataSource    <TypedLocalObjectReference>
    apiGroup    <string>
    kind        <string> -required-
    name        <string> -required-
  dataSourceRef <TypedObjectReference>
    apiGroup    <string>
    kind        <string> -required-
    name        <string> -required-
    namespace   <string>
  resources     <ResourceRequirements>
    claims      <[]ResourceClaim>
      name      <string> -required-
    limits      <map[string]Quantity>
    requests    <map[string]Quantity>
  selector      <LabelSelector>
    matchExpressions    <[]LabelSelectorRequirement>
      key       <string> -required-
      operator  <string> -required-
      values    <[]string>
    matchLabels <map[string]string>
  storageClassName      <string>
  volumeMode    <string>
  volumeName    <string>

```
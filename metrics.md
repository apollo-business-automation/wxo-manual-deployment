# watsonx Orchestrate agentic only, without GPUs real metrics from cluster

Version of Software Hub 5.3.1

## Metrics gathered with artificial resource quota

Label projects we want to count

```bash
oc label namespace wxo quota-count=true
oc label namespace wxo-operators quota-count=true
oc label namespace ibm-knative-events quota-count=true
oc label namespace knative-eventing quota-count=true
oc label namespace cert-manager quota-count=true
oc label namespace cert-manager-operator quota-count=true
oc label namespace ibm-licensing quota-count=true
```

```yaml
apiVersion: quota.openshift.io/v1
kind: ClusterResourceQuota
metadata:
  name: counter
spec:
  quota:
    hard:
      count/deployments.apps: '99999999'
      count/statefulsets.apps: '99999999'
      count/jobs.batch: '99999999'
      count/cronjobs.batch: '99999999'
      count/services: '99999999'
      count/secrets: '99999999'
      count/configmaps: '99999999'
      pods: '99999999'
      requests.cpu: '99999999'
      requests.memory: 99999999Gi
      limits.cpu: '99999999'
      limits.memory: 99999999Gi
      requests.storage: 99999999Gi
      persistentvolumeclaims: '99999999'
      ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: 99999Gi
      ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '99999999'
      ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 99999Gi
      ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '99999999'
      requests.ephemeral-storage: 99999999Gi
      limits.ephemeral-storage: 99999999Gi
  selector:
    annotations: null
    labels:
      matchLabels:
        quota-count: 'true'
```

Outputs for wxo relevant projects below, total is last item

```yaml
status:
  namespaces:
    - namespace: ibm-knative-events
      status:
        used:
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.ephemeral-storage: '0'
          count/jobs.batch: '0'
          count/secrets: '2'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 384Mi
          pods: '1'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          limits.cpu: '1'
          count/deployments.apps: '1'
          limits.ephemeral-storage: '0'
          limits.memory: 384Mi
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/configmaps: '5'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 200m
          count/services: '0'
    - namespace: wxo-operators
      status:
        used:
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.ephemeral-storage: 13121Mi
          count/jobs.batch: '1'
          count/secrets: '7'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 14499Mi
          pods: '23'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          limits.cpu: 18765m
          count/deployments.apps: '23'
          limits.ephemeral-storage: 19744Mi
          limits.memory: '36710304256'
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/configmaps: '17'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 2481m
          count/services: '2'
    - namespace: cert-manager-operator
      status:
        used:
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.ephemeral-storage: '0'
          count/jobs.batch: '0'
          count/secrets: '0'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 32Mi
          pods: '1'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          limits.cpu: '0'
          count/deployments.apps: '1'
          limits.ephemeral-storage: '0'
          limits.memory: '0'
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/configmaps: '5'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 10m
          count/services: '1'
    - namespace: cert-manager
      status:
        used:
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.ephemeral-storage: '0'
          count/jobs.batch: '0'
          count/secrets: '1'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: '0'
          pods: '3'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          limits.cpu: '0'
          count/deployments.apps: '3'
          limits.ephemeral-storage: '0'
          limits.memory: '0'
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/configmaps: '4'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: '0'
          count/services: '3'
    - namespace: ibm-licensing
      status:
        used:
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.ephemeral-storage: 250Mi
          count/jobs.batch: '1'
          count/secrets: '6'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 100Mi
          pods: '2'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          limits.cpu: 20m
          count/deployments.apps: '1'
          limits.ephemeral-storage: 500Mi
          limits.memory: 150Mi
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/configmaps: '5'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 20m
          count/services: '3'
    - namespace: knative-eventing
      status:
        used:
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 2100Mi
          requests.ephemeral-storage: 6Gi
          count/jobs.batch: '3'
          count/secrets: '31'
          count/statefulsets.apps: '1'
          persistentvolumeclaims: '6'
          requests.memory: 30082Mi
          pods: '22'
          count/cronjobs.batch: '0'
          requests.storage: 2100Mi
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          limits.cpu: 12300m
          count/deployments.apps: '14'
          limits.ephemeral-storage: 6Gi
          limits.memory: 34848Mi
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '6'
          count/configmaps: '34'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 2615m
          count/services: '24'
    - namespace: wxo
      status:
        used:
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 300Gi
          requests.ephemeral-storage: 22980Mi
          count/jobs.batch: '50'
          count/secrets: '139'
          count/statefulsets.apps: '3'
          persistentvolumeclaims: '22'
          requests.memory: '62091964160'
          pods: '81'
          count/cronjobs.batch: '20'
          requests.storage: 805Gi
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '2'
          limits.cpu: 98302m
          count/deployments.apps: '36'
          limits.ephemeral-storage: '61769173504'
          limits.memory: '174265332224'
          ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '20'
          count/configmaps: '160'
          ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: 505Gi
          requests.cpu: 34623m
          count/services: '53'
  total:
    used:
      ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 309300Mi
      requests.ephemeral-storage: 42495Mi
      count/jobs.batch: '55'
      count/secrets: '186'
      count/statefulsets.apps: '4'
      persistentvolumeclaims: '28'
      requests.memory: '109379596032'
      pods: '133'
      count/cronjobs.batch: '20'
      requests.storage: 826420Mi
      ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '2'
      limits.cpu: 130387m
      count/deployments.apps: '79'
      limits.ephemeral-storage: '89438996992'
      limits.memory: 242262063Ki
      ocs-external-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '26'
      count/configmaps: '230'
      ocs-external-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: 505Gi
      requests.cpu: 39949m
      count/services: '86'
```

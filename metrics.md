# watsonx Orchestrate agentic only, without GPUs real metrics from cluster

Version of Software Hub 5.2.2

## Nodes utilisation

node/worker-1
  Resource           Requests           Limits
  cpu                8492m (54%)        26965m (173%)
  memory             27652046400 (41%)  60083034Ki (92%)

node/worker-2
  Resource           Requests       Limits
  cpu                8027m (51%)    26810m (172%)
  memory             36187Mi (57%)  57088Mi (90%)

node/worker-3
  Resource           Requests       Limits
  cpu                9510m (61%)    31280m (201%)
  memory             27723Mi (43%)  53696Mi (84%)

node/worker-4
  Resource           Requests       Limits
  cpu                8955m (57%)    36210m (233%)
  memory             25194Mi (39%)  48384Mi (76%)

node/worker-5
  Resource           Requests           Limits
  cpu                10732m (69%)       30550m (197%)
  memory             25333254656 (38%)  58092866048 (87%)

node/worker-6
  Resource           Requests       Limits
  cpu                9402m (60%)    35110m (226%)
  memory             23892Mi (37%)  55687432704 (84%)

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
      ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: 99999Gi
      ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '99999999'
      ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 99999Gi
      ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '99999999'
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
    - namespace: knative-eventing
      status:
        used:
          requests.ephemeral-storage: 6376Mi
          count/jobs.batch: '3'
          count/secrets: '51'
          count/statefulsets.apps: '1'
          persistentvolumeclaims: '6'
          requests.memory: 30210Mi
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          pods: '22'
          count/cronjobs.batch: '0'
          requests.storage: 2100Mi
          limits.cpu: 12800m
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '6'
          count/deployments.apps: '14'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          limits.ephemeral-storage: 6376Mi
          limits.memory: 34976Mi
          count/configmaps: '33'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 2100Mi
          requests.cpu: 2690m
          count/services: '26'
    - namespace: cert-manager
      status:
        used:
          requests.ephemeral-storage: '0'
          count/jobs.batch: '0'
          count/secrets: '8'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: '0'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          pods: '3'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          limits.cpu: '0'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/deployments.apps: '3'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          limits.ephemeral-storage: '0'
          limits.memory: '0'
          count/configmaps: '4'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: '0'
          count/services: '3'
    - namespace: wxo-operators
      status:
        used:
          requests.ephemeral-storage: 11151Mi
          count/jobs.batch: '23'
          count/secrets: '66'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 18509Mi
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          pods: '46'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          limits.cpu: 22015m
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/deployments.apps: '27'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          limits.ephemeral-storage: 28398Mi
          limits.memory: '46373980672'
          count/configmaps: '38'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 3491m
          count/services: '30'
    - namespace: wxo
      status:
        used:
          requests.ephemeral-storage: 24216Mi
          count/jobs.batch: '48'
          count/secrets: '129'
          count/statefulsets.apps: '3'
          persistentvolumeclaims: '28'
          requests.memory: '93261105216'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '2'
          pods: '90'
          count/cronjobs.batch: '17'
          requests.storage: 1200Gi
          limits.cpu: 130480m
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '26'
          count/deployments.apps: '35'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: 600Gi
          limits.ephemeral-storage: 56550757Ki
          limits.memory: '237855322624'
          count/configmaps: '158'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 600Gi
          requests.cpu: 40920m
          count/services: '55'
    - namespace: cert-manager-operator
      status:
        used:
          requests.ephemeral-storage: '0'
          count/jobs.batch: '0'
          count/secrets: '5'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 32Mi
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          pods: '1'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          limits.cpu: '0'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/deployments.apps: '1'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          limits.ephemeral-storage: '0'
          limits.memory: '0'
          count/configmaps: '4'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 10m
          count/services: '1'
    - namespace: ibm-licensing
      status:
        used:
          requests.ephemeral-storage: 506Mi
          count/jobs.batch: '1'
          count/secrets: '16'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 356Mi
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          pods: '3'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          limits.cpu: 520m
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/deployments.apps: '2'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          limits.ephemeral-storage: 500Mi
          limits.memory: 1174Mi
          count/configmaps: '12'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 220m
          count/services: '3'
    - namespace: ibm-knative-events
      status:
        used:
          requests.ephemeral-storage: 1Gi
          count/jobs.batch: '1'
          count/secrets: '6'
          count/statefulsets.apps: '0'
          persistentvolumeclaims: '0'
          requests.memory: 1074Mi
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          pods: '2'
          count/cronjobs.batch: '0'
          requests.storage: '0'
          limits.cpu: '2'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '0'
          count/deployments.apps: '1'
          ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: '0'
          limits.ephemeral-storage: 10Gi
          limits.memory: 2Gi
          count/configmaps: '6'
          ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: '0'
          requests.cpu: 210m
          count/services: '1'
  total:
    used:
      requests.ephemeral-storage: 43273Mi
      count/jobs.batch: '76'
      count/secrets: '281'
      count/statefulsets.apps: '4'
      persistentvolumeclaims: '34'
      requests.memory: '145879697472'
      ocs-storagecluster-cephfs.storageclass.storage.k8s.io/persistentvolumeclaims: '2'
      pods: '167'
      count/cronjobs.batch: '17'
      requests.storage: 1230900Mi
      limits.cpu: 167815m
      ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/persistentvolumeclaims: '32'
      count/deployments.apps: '83'
      ocs-storagecluster-cephfs.storageclass.storage.k8s.io/requests.storage: 600Gi
      limits.ephemeral-storage: 103157093Ki
      limits.memory: 316682431Ki
      count/configmaps: '255'
      ocs-storagecluster-ceph-rbd.storageclass.storage.k8s.io/requests.storage: 616500Mi
      requests.cpu: 47541m
      count/services: '119'
```

# watsonx Orchestrate without GPUs manual installation ✍️<!-- omit in toc -->

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing

Version of Software Hub 5.2.2

- [Disclaimer ✋](#disclaimer-)
- [Used environement](#used-environement)
- [Installing Software Hub](#installing-software-hub)
- [Setting up a client workstation](#setting-up-a-client-workstation)
  - [Creating an install client directly in OCP](#creating-an-install-client-directly-in-ocp)
  - [Command line preparation in install Pod](#command-line-preparation-in-install-pod)
- [Setting up a cluster for IBM Software Hub](#setting-up-a-cluster-for-ibm-software-hub)
  - [Installing the Red Hat OpenShift Container Platform cert-manager Operator](#installing-the-red-hat-openshift-container-platform-cert-manager-operator)
- [Collecting information required to install IBM Software Hub](#collecting-information-required-to-install-ibm-software-hub)
  - [Setting up installation environment variables](#setting-up-installation-environment-variables)
- [Preparing your cluster for IBM Software Hub](#preparing-your-cluster-for-ibm-software-hub)
  - [Updating the global image pull secret for IBM Software Hub](#updating-the-global-image-pull-secret-for-ibm-software-hub)
  - [Installing shared cluster components for IBM Software Hub](#installing-shared-cluster-components-for-ibm-software-hub)
  - [Installing Red Hat OpenShift Serverless Knative Eventing](#installing-red-hat-openshift-serverless-knative-eventing)
- [Preparing to install an instance of IBM Software Hub](#preparing-to-install-an-instance-of-ibm-software-hub)
- [Applying the required permissions by running the authorize-instance-topology command](#applying-the-required-permissions-by-running-the-authorize-instance-topology-command)
  - [Creating secrets for services that use Multicloud Object Gateway](#creating-secrets-for-services-that-use-multicloud-object-gateway)
- [Installing an instance of IBM Software Hub](#installing-an-instance-of-ibm-software-hub)
- [Setting up IBM Software Hub](#setting-up-ibm-software-hub)
  - [Applying your entitlements without node pinning](#applying-your-entitlements-without-node-pinning)
- [Installing solutions and services](#installing-solutions-and-services)
  - [Specifying installation options for services](#specifying-installation-options-for-services)
  - [Running a batch installation of solutions and services](#running-a-batch-installation-of-solutions-and-services)
- [Post-installation setup (Day 1 operations)](#post-installation-setup-day-1-operations)
- [Post-installation setup for watsonx Orchestrate](#post-installation-setup-for-watsonx-orchestrate)
  - [Creating a service instance for watsonx Orchestrate from the web client](#creating-a-service-instance-for-watsonx-orchestrate-from-the-web-client)
  - [Registering external models through AI gateway](#registering-external-models-through-ai-gateway)


## Disclaimer ✋

This is **not** an official IBM documentation.  
Absolutely no warranties, no support, no responsibility for anything.  
Use it on your own risk and always follow the official IBM documentations.  
It is always your responsibility to make sure you are license compliant when using this repository to install watsonx Orchestrate.  

Please do not hesitate to create an issue here if needed. Your feedback is appreciated.

Not for production use. Suitable for Demo and PoC environments.

## Used environement

- Empty OpenShift cluster of a supported version
- With direct internet connection
- File RWX StorageClass - in this case ocs-external-storagecluster-cephfs is used, feel free to find and replace
- Block RWO StorageClass - in this case ocs-external-storagecluster-ceph-rbd is used, feel free to find and replace
- Cluster admin user
- IBM entitlement key from https://myibm.ibm.com/products-services/containerlibrary, replace to <enter your IBM entitlement API key>

## Installing Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing#install__client

## Setting up a client workstation

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-setting-up-client-workstation

Requested tooling provided in the install Pod.

- cpd-cli
- oc

### Creating an install client directly in OCP

In your OCP cluster create Project *cp4d-install* using OpenShift console.
```yaml
kind: Project
apiVersion: project.openshift.io/v1
metadata:
  name: cp4d-install
  labels:
    app: cp4d-install
```

Create PVC where all files used for installation would be kept if you would need to re-instantiate the install Pod.
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: install
  namespace: cp4d-install
  labels:
    app: cp4d-install
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: ocs-external-storagecluster-cephfs
  volumeMode: Filesystem
```

Assign cluster admin permissions to *cp4d-install* default ServiceAccount under which the installation is performed by applying the following yaml.

This requires the logged in OpenShift user to be cluster admin.

The ServiceAccount needs to have cluster admin to be able to create all resources needed to deploy the platform.
```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cluster-admin-cp4d-install
  labels:
    app: cp4d-install
subjects:
  - kind: User
    apiGroup: rbac.authorization.k8s.io
    name: "system:serviceaccount:cp4d-install:default"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```

Create a pod from which the install will be performed with the following definition.  
It is ready when the message *Install pod - Ready* is in its Log.
```yaml
kind: Pod
apiVersion: v1
metadata:
  name: install
  namespace: cp4d-install
  labels:
    app: cp4d-install
spec:
  containers:
    - name: install
      securityContext:
        privileged: true
      image: ubi9/ubi:9.6
      command: ["/bin/bash"]
      args:
        ["-c","cd /usr;
          yum install podman -y;
          curl -k https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-client-linux.tar.gz --output oc.tar;
          tar -xvf oc.tar oc;
          chmod u+x oc;
          ln -fs /usr/oc /usr/bin/oc;
          curl -kL https://github.com/IBM/cpd-cli/releases/download/v14.2.2/cpd-cli-linux-EE-14.2.2.tgz --output cpd-cli-linux.tgz;
          tar -xzf cpd-cli-linux.tgz;
          mv $(tar -tzf cpd-cli-linux.tgz | head -1 | cut -f1 -d'/') cpd-cli;
          echo 'export PATH=/usr/cpd-cli:$PATH' >> ~/.bashrc;
          export PATH=/usr/cpd-cli:$PATH;
          cd install;
          cpd-cli manage restart-container;
          podman -v;
          oc version;
          cpd-cli version;
          while true;
          do echo 'Install pod - Ready - Enter it via Terminal and \"bash\"';
          sleep 300; done"]
      imagePullPolicy: IfNotPresent
      volumeMounts:
        - name: install
          mountPath: /usr/install
  volumes:
    - name: install
      persistentVolumeClaim:
        claimName: install
```

### Command line preparation in install Pod

This needs to be repeated if you don't perform this in one go and you come back to resume.

Open Terminal window of the *install* pod.

Enter bash.
```sh
bash
```

Enter change dir and source variables.
```bash
cd /usr/install
source /usr/install/cpd_vars.sh # Only if you are returning later to continue, doesn't work the first time as it doesn't exist yet.
```

## Setting up a cluster for IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-setting-up-cluster

### Installing the Red Hat OpenShift Container Platform cert-manager Operator

Based on https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/security_and_compliance/cert-manager-operator-for-red-hat-openshift#cert-manager-operator-install

Based on https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/security_and_compliance/cert-manager-operator-for-red-hat-openshift#cert-manager-install-cli_cert-manager-operator-install

Create cert-manager-operator Project
```bash
echo "
kind: Project
apiVersion: project.openshift.io/v1
metadata:
  name: cert-manager-operator
" | oc apply -f -
```

Create cert-manager-operator OperatorGroup
```bash
echo "
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-cert-manager-operator
  namespace: cert-manager-operator
spec:
  targetNamespaces: []
  spec: {}
" | oc apply -f -
```

Create cert-manager-operator OperatorGroup
```bash
echo "
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-cert-manager-operator
  namespace: cert-manager-operator
spec:
  channel: stable-v1
  name: openshift-cert-manager-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  installPlanApproval: Automatic
" | oc apply -f -
```

Wait for pods to reach Running status

```bash
oc get pods -n cert-manager -w
```

## Collecting information required to install IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-collecting-required-information

### Setting up installation environment variables

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=information-setting-up-installation-environment-variables

```bash
cat << EOF > /usr/install/cpd_vars.sh
#===============================================================================
# IBM Software Hub installation variables
#===============================================================================

# ------------------------------------------------------------------------------
# Cluster
# ------------------------------------------------------------------------------

export OCP_URL=$(oc whoami --show-server=true)
export OPENSHIFT_TYPE=self-managed
export IMAGE_ARCH=amd64
# export OCP_USERNAME=
# export OCP_PASSWORD=<enter your password>
export OCP_TOKEN=$(oc whoami -t)
export SERVER_ARGUMENTS="--server=\${OCP_URL}"
# export LOGIN_ARGUMENTS="--username=\${OCP_USERNAME} --password=\${OCP_PASSWORD}"
export LOGIN_ARGUMENTS="--token=\${OCP_TOKEN}"
export CPDM_OC_LOGIN="cpd-cli manage login-to-ocp \${SERVER_ARGUMENTS} \${LOGIN_ARGUMENTS}"
export OC_LOGIN="oc login \${SERVER_ARGUMENTS} \${LOGIN_ARGUMENTS}"

# ------------------------------------------------------------------------------
# Projects
# ------------------------------------------------------------------------------

export PROJECT_CERT_MANAGER=ibm-cert-manager
export PROJECT_LICENSE_SERVICE=ibm-licensing
export PROJECT_SCHEDULING_SERVICE=ibm-cpd-scheduler
export PROJECT_IBM_EVENTS=ibm-knative-events
export PROJECT_CPD_INST_OPERATORS=wxo-operators
export PROJECT_CPD_INST_OPERANDS=wxo

# ------------------------------------------------------------------------------
# Storage
# ------------------------------------------------------------------------------

export STG_CLASS_BLOCK=ocs-external-storagecluster-ceph-rbd
export STG_CLASS_FILE=ocs-external-storagecluster-cephfs

# ------------------------------------------------------------------------------
# IBM Entitled Registry
# ------------------------------------------------------------------------------

export IBM_ENTITLEMENT_KEY=<enter your IBM entitlement API key>

# ------------------------------------------------------------------------------
# IBM Software Hub version
# ------------------------------------------------------------------------------

export VERSION=5.2.2

# ------------------------------------------------------------------------------
# Components
# ------------------------------------------------------------------------------

export COMPONENTS=ibm-licensing,scheduler,cpfs,cpd_platform,watsonx_orchestrate
EOF
```

Make excutable and source in current environment
```bash
chmod 700 /usr/install/cpd_vars.sh
source /usr/install/cpd_vars.sh
```

## Preparing your cluster for IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-preparing-your-cluster

### Updating the global image pull secret for IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=cluster-updating-global-image-pull-secret

```bash
${CPDM_OC_LOGIN}
cpd-cli manage add-icr-cred-to-global-pull-secret \
--entitled_registry_key=${IBM_ENTITLEMENT_KEY}
```

### Installing shared cluster components for IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=cluster-installing-shared-components

```bash
${CPDM_OC_LOGIN}
cpd-cli manage apply-cluster-components \
--release=${VERSION} \
--license_acceptance=true \
--licensing_ns=${PROJECT_LICENSE_SERVICE}
```

### Installing Red Hat OpenShift Serverless Knative Eventing

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=software-installing-red-hat-openshift-serverless-knative-eventing

```bash
oc new-project ${PROJECT_IBM_EVENTS}
```

```bash
${CPDM_OC_LOGIN}
cpd-cli manage deploy-knative-eventing \
--release=${VERSION} \
--block_storage_class=${STG_CLASS_BLOCK}
```

## Preparing to install an instance of IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-preparing-install-instance-software-hub

## Applying the required permissions by running the authorize-instance-topology command

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=arppn-applying-required-permissions-by-running-authorize-instance-topology-command

```bash
${CPDM_OC_LOGIN}
cpd-cli manage authorize-instance-topology \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

### Creating secrets for services that use Multicloud Object Gateway

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=piish-creating-secrets-services-that-use-multicloud-object-gateway

```bash
export NOOBAA_ACCOUNT_CREDENTIALS_SECRET=noobaa-admin
export NOOBAA_ACCOUNT_CERTIFICATE_SECRET=noobaa-s3-serving-cert
```

```bash
cpd-cli manage setup-mcg \
--components=watsonx_orchestrate \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--noobaa_account_secret=${NOOBAA_ACCOUNT_CREDENTIALS_SECRET} \
--noobaa_cert_secret=${NOOBAA_ACCOUNT_CERTIFICATE_SECRET}
```

## Installing an instance of IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-instance-software-hub
Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=hub-installing-software

The following command is a long running thing, so it is important to keep the terminal alive or go back and restart the command.
```bash
cpd-cli manage setup-instance \
--release=${VERSION} \
--license_acceptance=true \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--run_storage_tests=true
```

Confirm that the status of the operands is Completed (Progress of both 100% - takes some time)
```bash
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```

Check the health of the resources in the operators project
```bash
cpd-cli health operators \
--operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--control_plane_ns=${PROJECT_CPD_INST_OPERANDS}
```

Check the health of the resources in the operands project:
```bash
cpd-cli health operands \
--control_plane_ns=${PROJECT_CPD_INST_OPERANDS}
```

Get the URL and default credentials of the web client and make note about URL, Username and Password
```bash
cpd-cli manage get-cpd-instance-details \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--get_admin_initial_credentials=true
```

## Setting up IBM Software Hub

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-setting-up-software-hub

### Applying your entitlements without node pinning

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=entitlements-applying-your-without-node-pinning

```bash
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=watsonx-orchestrate
```

## Installing solutions and services

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=installing-solutions-services

### Specifying installation options for services

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=services-specifying-installation-options#install-platform-param-file__orchestrate-parms

If you plan to install watsonx Orchestrate configuration, specify the appropriate installation options in a file named install-options.yml in the cpd-cli work directory (For example: cpd-cli-workspace/olm-utils-workspace/work).

```bash
cat << EOF > /usr/install/cpd-cli-workspace/olm-utils-workspace/work/install-options.yml
################################################################################
# watsonx Orchestrate parameters
################################################################################
watson_orchestrate_watsonx_ai_type: false
EOF
```

### Running a batch installation of solutions and services

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=services-running-batch-installation-solutions

Install Operators
```bash
${CPDM_OC_LOGIN}
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=${COMPONENTS}
```

Install Operands
```bash
cpd-cli manage apply-cr \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=${COMPONENTS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--license_acceptance=true \
--param-file=/tmp/work/install-options.yml
```

## Post-installation setup (Day 1 operations)

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=administering-post-installation-setup-day-1

## Post-installation setup for watsonx Orchestrate

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=orchestrate-post-installation-setup

### Creating a service instance for watsonx Orchestrate from the web client

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=csi-creating-service-instance-from-web-client-3

Crate new instance

Access on
https://cpd-wxo.apps.xxx/orchestrate/chat

### Registering external models through AI gateway

Based on https://www.ibm.com/docs/en/software-hub/5.2.x?topic=setup-registering-external-models-through-ai-gateway

You have to add external models

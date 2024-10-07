# RedHat EX DO280 (Red Hat Certified Specialist in OpenShift Administration ) Notes 

Songs: 
- [| C I T Y N I G H T S | - A NewRetroWave Mix | 1 Hour | Synthwave/ Retrowave/ Darksynth |
](https://www.youtube.com/watch?v=WhjuZUlcmk0)
- [| O U T L A W | - A NewRetroWave Mix | 1 Hour | Retrowave/ Synthwave/ Outrun |
](https://www.youtube.com/watch?v=0ZQoqvRtiaM)

## About the exam

- Exam format: task based. The exam itself contains a set of tasks (16) that you have to perform, which represents the tasks for an OpenShift administrator.

- Exam duration: 3 hours

- Exam passing grade: 70% - you must got 12 questions right out of 16. (note that partial completion of a question is not counted as a correct answer)

## Exam objectives

These objectives was retrieved from [EX280](https://www.redhat.com/en/services/training/red-hat-certified-openshift-administrator-exam?section=objectives).

To become a Red Hat Certified Specialist in OpenShift Administration, you should be able to perform these tasks:

- **Manage OpenShift Container Platform**
  - Use the web console to manage and configure an OpenShift cluster
  - Use the command-line interface to manage and configure an OpenShift cluster
  - Query, format, and filter attributes of Kubernetes resources
  - Import, export, and configure Kubernetes resources
  - Locate and examine container images
  - Create and delete projects
  - Examine resources and cluster status
  - View logs
  - Monitor cluster events and alerts
  - Assess the health of an OpenShift cluster
  - Troubleshoot common container, pod, and cluster events and alerts
  - Use product documentation

- **Deploy Applications**
  - Deploy applications from resource manifests
  - Use Kustomize overlays to modify application configurations
  - Deploy applications from images, OpenShift templates, and Helm charts
  - Deploy jobs to perform one-time tasks
  - Manage application deployments
  - Work with replica sets
  - Work with labels and selectors
  - Configure services
  - Expose both HTTP and non-HTTP applications to external access
  - Work with operators such as MetalLB and Multus

- **Manage Storage for Application Configuration and Data**
  - Create and use secrets
  - Create and use configuration maps
  - Provision Persistent Storage volumes for block and file-based data
  - Use storage classes
  - Manage non-shared storage with StatefulSets

- **Configure Applications for Reliability**
  - Configure and use health probes
  - Reserve and limit application compute capacity
  - Scale applications to meet increased demand

- **Manage Application Updates**
  - Identify images using tags and digests
  - Roll back failed deployments
  - Manage image streams
  - Use triggers to manage images

- **Manage Authentication and Authorization**
  - Configure the HTPasswd identity provider for authentication
  - Create and delete users
  - Modify user passwords
  - Create and manage groups
  - Modify user and group permissions

- **Configure Network Security**
  - Configure networking components
  - Troubleshoot software defined networking
  - Create and edit external routes
  - Control cluster network ingress
  - Secure external and internal traffic using TLS certificates
  - Configure application network policies

- **Enable Developer Self-Service**
  - Configure cluster resource quotas
  - Configure project quotas
  - Configure project resource requirements
  - Configure project limit ranges
  - Configure project templates

- **Manage OpenShift Operators**
  - Install an operator
  - Delete an operator

- **Configure Application Security**
  - Configure and manage service accounts
  - Run privileged applications
  - Create service accounts
  - Manage and apply permissions using security context constraints
  - Create and apply secrets to manage sensitive information
  - Configure application access to Kubernetes APIs
  - Configure Kubernetes CronJobs


## Get started with OpenShift Local
- [Guide](https://www.redhat.com/sysadmin/install-openshift-local)

## Manage OpenShift Container Platform

Executing troubleshooting commands:

- Getting node informations:
```
oc get node
oc describe node <nodename>
```

- Getting busiest nodes
```
oc adm top nodes
```

- Getting `journalctl` logs from a node
```
oc adm node-logs -u kubelet my-node-name
```

- Running a remote shell for a node
```
oc debug node/<nodename>
```

- Work with cluster installers
```
oc get clusteroperators
oc get clusterversion -o yaml
```

- Getting a pod logs
```
oc logs <pod> [-c <container>] [-f]
```

- Debugging a deployment or a pod
```
oc debug deployment/<deplname> [--as-root]
oc rsh <podname>
oc port-forward <pod> <localport>:<remoteport>
```

- Getting a file from a pod
```
oc cp file <pod>:/file
oc cp <pod>:/file file
```

## Manage Users and Policies

Removing the default kubeadmin:
```
oc delete secret kubeadmin -n kube-system
```

Working with htpasswd

- Create: `htpasswd -c -B -b /tmp/htpasswd luiz redhat123`
- Update: `htpasswd -b /tmp/htpasswd luiz redhat1234`
- Delete: `htpasswd -D /tmp/htpasswd luiz`

Create a secret:
```
oc create secret generic htpasswd-secret \
> --from-file htpasswd=/tmp/htpasswd -n openshift-config
```
Extract the secret content: 
```
oc extract secrets/htpasswd-secret --to=-
```

Adding to OAuth:
```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd-secret
```
or 

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
...output omitted...
spec:
  identityProviders:
  - ldap:
...output omitted...
    type: LDAP
  - htpasswd:
      fileData:
        name: localusers
    mappingMethod: claim
    name: myusers
    type: HTPasswd
```

the authentication pods should be restarted 
```
oc get pods -n openshift-authentication
```

### Getting users and identities
```
oc get users
oc get identity
```

## Manage Resource Access

Defining and Applying Permissions Using RBAC  
```
oc adm policy add-cluster-role-to-user cluster-admin username
oc adm policy remove-cluster-role-from-user cluster-admin username
oc adm policy who-can delete user
oc adm policy add-role-to-user basic-user dev -n wordpress
```

Default roles
-	**basic-user** Users with this role have read access to the project.
-	**cluster-admin** Users with this role have superuser access to the cluster resources. These users can perform any action on the cluster, and have full control of all projects.
-	**cluster-status** Users with this role can get cluster status information.
-	**self-provisioner** Users with this role can create new projects.
Default roles that can be added or removed from a project level:
-	**admin** Users with this role can manage all project resources, including granting access to other users to the project.
-	**edit** Users with this role can create, change, and delete common application resources from the project, such as services and deployment configurations. These users cannot act on management resources such as limit ranges and quotas, and cannot manage access permissions to the project.
-	**view** Users with this role can view project resources, but cannot modify project resources.

System users: system:admin, system:openshift-registry, and system:node:node1.example.com.

Managing Sensitive Information with Secrets  
```
oc create secret generic secret_name \
> --from-literal key1=secret1 \
> --from-literal key2=secret2
oc secrets add --for mount serviceaccount/serviceaccount-name \
> secret/secret_name
oc set env dc/demo --from=secret/demo-secret
oc set volume dc/demo \
> --add \
> --type=secret \
> --secret-name=demo-secret \
> --mount-path=/app-secrets
```
### Difference between add-cluster-role-to-user and add-role-to-user

1. oc adm policy add-cluster-role-to-user
Cluster-wide scope: This command is used to assign cluster roles, which apply across the entire OpenShift cluster.

- **Cluster role:** A cluster role is a role that is defined at the cluster level and can provide permissions that apply to all projects (namespaces) in the cluster.

- **Use case:** You would use this command to assign roles like cluster-admin or other roles that grant access across all namespaces, nodes, or the cluster as a whole.

```shell
oc adm policy add-cluster-role-to-user cluster-admin user1
```

2. oc adm policy add-role-to-user
Namespace scope: This command is used to assign a role that is scoped to a specific namespace (project).

- **Role:** A role is limited to a particular namespace, providing permissions only within that namespace.

- **Use case:** You would use this command when you want to assign roles like admin, edit, or view to users for specific projects.

**Example:**
```
oc adm policy add-role-to-user admin user1 -n myproject
```
This command gives user1 the admin role in the myproject namespace.


Controlling Application Permissions with Security Context Constraints (SCCs) (anyuid, privileged etc)  
```
oc adm policy add-scc-to-user anyuid -z default
```

### Remove the Capability to Create Projects for All Regular Users

Removing the `self-provisioner` cluster role from authenticated users and groups denies permissions for self-provisioning any new projects:
```
$ oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
```
Try creating a new project with a regular user, it should fail:
```
$ oc login -u alice -p password1
$ oc new-project project1
Error from server (Forbidden): You may not request a new project via this API.
```
### Create Cluster Admin
```
$ oc adm policy add-cluster-role-to-user cluster-admin admin
```

### Create a New Project
Create two new projects:
```
$ oc new-project project1 \
  --description="Alice OpenShift Project" \
  --display-name="Playground for Alice"
```
```
$ oc new-project project2 \
  --description="Vince OpenShift Project" \
  --display-name="Playground for Vince"
```

View project details:
```yaml
$ oc describe project/project1
Name:			project1
Created:		2 minutes ago
Labels:			<none>
Annotations:		openshift.io/description=Alice OpenShift Project
			openshift.io/display-name=Playground for Alice
			openshift.io/requester=system:admin
			openshift.io/sa.scc.mcs=s0:c13,c12
			openshift.io/sa.scc.supplemental-groups=1000180000/10000
			openshift.io/sa.scc.uid-range=1000180000/10000
Display Name:		Playground for Alice
Description:		Alice OpenShift Project
Status:			Active
Node Selector:		<none>
Quota:			<none>
Resource
```

Associate a User with a Project
Add alice as an administrator for the `project1` project:
```
$ oc policy add-role-to-user admin alice -n project1
```

Add vince a developer for the `project2` project:
```
$ oc policy add-role-to-user edit vince -n project2
```

Add alice to have read access to the `project2` project:
```
$ oc policy add-role-to-user view alice -n project2
```

### View Local Role Bindings for a Project
```
$ oc describe rolebinding.rbac -n project1
```

### Security Context Constraints (SCCs)
List security context constraints
```
$ oc get scc | awk '{ print $1 }'
NAME
anyuid
hostaccess
hostmount-anyuid
hostnetwork
nonroot
privileged
restricted
```

### Create a Service Account
Create a service account apache-account:
```
$ oc create serviceaccount apache-account
```
Associate the new service account with the `anyuid` security context:
```
$ oc adm policy add-scc-to-user anyuid -z apache-account
```
Edit the deployment config for apache:
```
$ oc edit dc/apache
```
Add the service account definition:
```yaml
spec:
  template:
    spec:
      serviceAccountName: apache-account
```

## Quota and Limits
A project can contain multiple `ResourceQuota` objects.

A `LimitRange` resource, also called a limit, defines the default, minimum, and maximum values for compute resource requests and limits for a single pod or for a single container defined inside the project.

### Get Quota and Limits
```
$ oc get quota
$ oc get limits
```
### Create Quota
```
$ oc create quota project1-quota --hard=memory=2Gi,cpu=200m,pods=10 -n project1
$ oc create quota my-quota --hard=cpu=2,memory=1G,pods=3,services=6,secrets=6,replicationcontrollers=6 
```
```
$ oc describe quota -n project1
Name:       project1-quota
Namespace:  project1
Resource    Used  Hard
--------    ----  ----
cpu         0     200m
memory      0     2Gi
pods        0     10
```

```
$ oc get quota
NAME       AGE   REQUEST                                                                                       LIMIT
my-quota   4s    cpu: 0/2, memory: 0/1G, pods: 0/3, replicationcontrollers: 0/6, secrets: 6/6, services: 0/6 
```

### Delete All Quota
```
$ oc delete quota --all -n project1
```
### Create Limits
File `project1-resource-limits.yaml`:
```yaml
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "project1-resource-limits"
spec:
  limits:
    - type: "Pod"
      max:
        cpu: "2"
        memory: "1Gi"
      min:
        cpu: "200m"
        memory: "16Mi"
    - type: "Container"
      max:
        cpu: "2"
        memory: "1Gi"
      min:
        cpu: "100m"
        memory: "8Mi"
      default:
        cpu: "300m"
        memory: "200Mi"
      defaultRequest:
        cpu: "200m"
        memory: "100Mi"
      maxLimitRequestRatio:
        cpu: "10"
```

```
$ oc create -f project1-resource-limits.yaml -n project1
```

```
$ oc describe limits -n project1
Name:       project1-resource-limits
Namespace:  project1
Type        Resource  Min   Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---  ---------------  -------------  -----------------------
Pod         cpu       200m  2    -                -              -
Pod         memory    16Mi  1Gi  -                -              -
Container   cpu       100m  2    200m             300m           10
Container   memory    8Mi   1Gi  100Mi            200Mi          -
```

### List Nodes Including Labels
```
$ oc get nodes --show-labels
```
### Label a Node
```
$ oc label node node1.lab.example.com region=infra --overwrite=true
$ oc label node node2.lab.example.com region=apps --overwrite=true
```

## Configure Networking Components

Service types: ClusterIP, NodePort, LoadBalancer, ExternalService
```
oc describe dns.operator/default
```
CoreDNS entries:
•	A `svcname.namespace.svc.cluster.local`
•	SRV `_port-name._port-protocol.svc.namespace.svc.cluster.local`

```
oc get Network.config.openshift.io cluster -o yaml
```

Ingress rule:
```
oc get ingress
```

Create a private key, CSR and certificate:
```
$ openssl genrsa -out file.key 2048
```
```
$ openssl req -new -key file.key -out file.req  \
  -subj "/C=GB/ST=Ireland/L=Ireland/O=IT/OU=IT/CN=www.example.com"
```
```
$ openssl x509 -req -days 366 -in file.req  \
      -signkey file.key -out file.crt
```



Secure route (edge/passthru):
```
$ oc get svc
$ oc create route edge --service=my-php-service \
    --hostname=<host>.apps.acme.com \
    --key=file.key --cert=file.crt \
    --insecure-policy=Redirect
```

### Create a Secret
```
$ oc create secret generic secret_name \
  --from-literal=key1=secret1 \
  --from-literal=key2=secret2
```


## Configure Pod Scheduling

Controlling pod scheduling behavior (factors that can affect on which nodes a pod can or cannot be run)  
```
oc label node node1.us-west-1.compute.internal env[-|=dev] [--overwrite]
oc get node node2.us-west-1.compute.internal --show-labels
oc get node -L failure-domain.beta.kubernetes.io/region
oc patch deployment/myapp --patch \
> '{"spec":{"template":{"spec":{"nodeSelector":{"env":"dev"}}}}}'
oc adm new-project demo --node-selector "tier=1"
oc annotate namespace demo \
> openshift.io/node-selector="tier=2" --overwrite
```

Limiting resource usage (factors that can affect the resources that a pod is allowed use or run)  
```
oc adm top nodes -l node-role.kubernetes.io/worker
oc set resources deployment hello-world-nginx \
> --requests cpu=10m,memory=20Mi --limits cpu=80m,memory=100Mi
oc create quota dev-quota --hard services=10,cpu=1300,memory=1.5Gi -n <ns>
oc get resourcequota -n <ns>
oc describe limitrange dev-limits
```

A violation of LimitRange constraints prevents pod creation, and resulting error messages are displayed. A violation of ResourceQuota constraints prevents a pod from being scheduled to any node. The pod might be created but remain in the pending state

```
oc create clusterquota user-qa \
> --project-annotation-selector openshift.io/requester=qa \
> --hard pods=12,secrets=20
oc create clusterquota env-qa \
> --project-label-selector environment=qa \
> --hard pods=10,services=5
```

Scaling an Application  
```
oc scale --replicas 3 deployment/myapp
oc autoscale dc/hello --min 1 --max 10 --cpu-percent 80
oc get hpa
```

Autoscale pods:
```
oc autoscale dc/my-httpd --min 1 --max 5 --cpu-percent=80
oc get hpa/my-httpd
oc get hpa --watch
```

## Configure Cluster Scaling

Manually Scaling an OpenShift Cluster  
```
oc scale --replicas=2 \
> machineset MACHINE-SET -n openshift-machine-api
```

Automatically Scaling an OpenShift Cluster
```
oc get clusterautoscaler
oc get machineautoscaler -n openshift-machine-api
```

## Import Template into OpenShift
```
$ oc apply -f httpd.yml -n openshift
```

## Declarative Resource Management 

- Kustomize 
- Helm

### Kustomize File Structure
Kustomize works on directories that contain a kustomization.yaml file at the root. Kustomize supports compositions and customization of different resources such as deployment, service, and secret. You can use patches to apply customization to different resources. Kustomize has a concept of base and overlays.

```shell
base
├── configmap.yaml
├── deployment.yaml
├── secret.yaml
├── service.yaml
├── route.yaml
└── kustomization.yaml
```
The base directory has YAML files to create configuration map, deployment, service, secret, and route resources. The base directory also has a kustomization.yaml file, such as the following example:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- configmap.yaml
- deployment.yaml
- secret.yaml
- service.yaml
- route.yaml
```

### Overlays 
Kustomize overlays declarative YAML artifacts, or patches, that override the general settings without modifying the original files. The overlay directory contains a kustomization.yaml file. The kustomization.yaml file can refer to one or more directories as bases. Multiple overlays can use a common base kustomization directory.

```shell
$ tree
base
├── configmap.yaml
├── deployment.yaml
├── secret.yaml
├── service.yaml
├── route.yaml
└── kustomization.yaml
overlay
└── development
    └── kustomization.yaml
└── testing
    └── kustomization.yaml
└── production
    ├── kustomization.yaml
    └── patch.yaml
```
The following example shows a kustomization.yaml file in the overlays/development directory:

```shell
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev-env
resources:
- ../../base
```

Kustomize provides fields to set values for all resources in the kustomization file:

| Field               | Description                                         |
|---------------------|-----------------------------------------------------|
| namespace           | Set a specific namespace for all resources.         |
| namePrefix          | Add a prefix to the name of all resources.          |
| nameSuffix          | Add a suffix to the name of all resources.          |
| commonLabels        | Add labels to all resources and selectors.          |
| commonAnnotations   | Add annotations to all resources and selectors.     |

The following is an example of a `kustomization.yaml` file in the overlays/testing directory:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: test-env
patches: 
- patch: |-
    - op: replace 
      path: /metadata/name
      value: frontend-test
  target: 
    kind: Deployment
    name: frontend
- patch: |- 
    - op: replace
      path: /spec/replicas
      value: 15
  target:
    kind: Deployment
    name: frontend
resources: 
- ../../base
commonLabels: 
  env: test
```
1. The patches field contains a list of patches.

2. The patch field defines operation, path, and value keys. In this example, the name changes to `frontend-test`.

3. The target field specifies the kind and name of the resource to apply the patch. In this example, you are changing the `frontend` deployment name to `frontend-test`.

4. This patch updates the number of replicas of the `frontend` deployment.

5. The `frontend-app/overlay/testing/kustomization.yaml` file uses the base kustomization file at `../../base` to create an application.

6. The `commonLabels` field adds the `env: test` label to all resources.

The following example shows a `kustomization.yaml` file that uses a `patch.yaml` file:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: prod-env
patches: 
- path: patch.yaml 
  target: 
    kind: Deployment
    name: frontend
  options:
    allowNameChange: true 
resources: 
- ../../base
commonLabels: 
  env: prod
```

The `patch.yaml` file has the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-prod
spec:
  replicas: 5
```

### View and Deploy Resources by Using Kustomize

view: 
```
oc kustomize base/
oc kustomize overlay/production
```

deploy
```
oc apply -k base/ 
oc apply -k overlay/production
oc kustomize base/ | oc apply -f -
```

### Delete Resources by Using Kustomize

```
oc delete -k base/
oc delete -k overlay/production
```
### Kustomize Generators
Configuration maps hold non-confidential data by using a key-value pair. Secrets are similar to configuration maps, but secrets hold confidential information such as usernames and passwords. Kustomize has configMapGenerator and secretGenerator fields that generate configuration map and secret resources.

```shell
# Create a application.properties file
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF
```

```
kubectl kustomize ./
oc kustomize ./
```

The generated ConfigMap is:
```yaml
apiVersion: v1
data:
  application.properties: |
    FOO=Bar    
kind: ConfigMap
metadata:
  name: example-configmap-1-8mbdf7882g
```

**Kustomize References:** 
- [Customization of Kubernetes YAML Configurations](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/kustomization.go)
- [Declarative Management of Kubernetes Objects Using Kustomize
](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
- [ConfigMap Generator](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#configmapgenerator)


## Packaged Applications - Helm
Helm is an open source application that helps to manage the lifecycle of Kubernetes applications.

Helm introduces the concept of charts. A chart is a package that describes a set of Kubernetes resources that you can deploy. Helm charts define values that you can customize when deploying an application. Helm includes functions to distribute charts and updates.

A Helm chart defines Kubernetes resources that you can deploy. A chart is a collection of files with a defined structure. These files include chart metadata (such as the chart name or version), resource definitions, and supporting material.

The following diagram shows the structure of a minimal Helm chart:

```yaml
sample/
├── Chart.yaml
├── templates
|   |── example.yaml
└── values.yaml 
```
#### Using Helm Charts
Helm is a command-line application. The helm command interacts with the following entities:

**Charts**
Charts are the packaged applications that the helm command deploys.

**Releases**
A release is the result of deploying a chart. You can deploy a chart many times to the same cluster. Each deployment is a different release.

**Versions**
A Helm chart can have many versions. Chart authors can release updates to charts, to adapt to later application versions, introduce new features, or fix issues.

#### Inspecting Helm Charts

```yaml
$ helm show chart chart-reference
apiVersion: v1
description: A Helm chart for Kubernetes
name: examplechart
version: 0.1.0
maintainers:
- email: dev@example.com
  name: Developerie
sources:
- https://git.example.com/examplechart
```

The show values subcommand displays the default values for the chart. The output is in YAML format and comes from the `values.yaml` file in the chart.

```shell
$ helm show values chart-reference
image:
  repository: "sample"
  tag: "1.8.10"
  pullPolicy: IfNotPresent
...output omitted...
```

List of the most important **Helm** commands:

1. `helm create [NAME]`
   - Creates a new Helm chart with the specified name.

2. `helm install [RELEASE_NAME] [CHART_PATH]`
   - Installs a Helm chart to Kubernetes.

3. `helm upgrade [RELEASE_NAME] [CHART_PATH]`
   - Upgrades an existing Helm release.

4. `helm uninstall [RELEASE_NAME]`
   - Uninstalls a Helm release.

5. `helm repo add [REPO_NAME] [REPO_URL]`
   - Adds a new Helm chart repository.

6. `helm repo update`
   - Updates the Helm chart repository cache.

7. `helm search repo [KEYWORD] --versions`
   - Searches for Helm charts in a repository.

8. `helm lint [CHART_PATH]`
   - Lints a Helm chart to check for issues.

9. `helm package [CHART_PATH]`
   - Packages a Helm chart into a tarball.

10. `helm template [CHART_PATH]`
    - Renders templates locally without sending them to Kubernetes.

#### Releases
When the helm install command runs successfully, besides creating the resources, Helm creates a release. Helm stores information about the release as a secret of the helm.sh/release.v1 type.

Inspecting Releases
```shell
$ helm list
NAME         NAMESPACE   REVISION  ...  STATUS     CHART            APP VERSION
my-release   example     1         ...  deployed   example-4.12.1   1.8.10
```

#### Upgrading Releases
The helm upgrade command uses similar arguments and options to the `helm install` command. However, the `helm upgrade` command interacts with existing resources in the cluster instead of creating resources from a blank state. Therefore, the `helm upgrade` command can have more complex effects, such as conflicting changes. Always review the chart documentation when using a later version of a chart, and when changing values.

```shell
helm upgrade --install <release-name>  repo/chart --version  0.0.2 -f values.yaml 
helm upgrade <release-name>  repo/chart --version  0.0.2 -f values.yaml 
```

#### Rolling Back Helm Upgrades


```shell
helm list 
helm history <release-name>
helm rollback <release-name> <revision>
Rollback was a success! Happy Helming!
```

#### Helm Repositories

Charts can be distributed as files, archives, or container images, or by using chart repositories.

| Subcommand                                 | Description                              |
|--------------------------------------------|------------------------------------------|
| `add NAME REPOSITORY_URL`                  | Add a Helm chart repository.             |
| `list`                                     | List Helm chart repositories.            |
| `update`                                   | Update Helm chart repositories.          |
| `remove REPOSITORY1_NAME REPOSITORY2_NAME …​` | Remove Helm chart repositories.          |

The following command adds a repository:
```shell
$ helm repo add \
  openshift-helm-charts https://charts.openshift.io/
"openshift-helm-charts" has been added to your repositories
```

The helm search repo command lists all available charts in the configured repositories:
```shell
helm search repo
```

**Helm References:**
- [Using Helm](https://helm.sh/docs/intro/using_helm/)
- [Charts](https://helm.sh/docs/topics/charts/)


## Authenticating with the X.509 Certificate
The installation logs provide the location of the kubeconfig file:

```
INFO Run 'export KUBECONFIG=/root/auth/kubeconfig' to manage the cluster with 'oc'.
```

```shell
export KUBECONFIG=/home/user/auth/kubeconfig
oc whoami
oc get nodes
```

As an alternative, we can use the --kubeconfig option of the oc command.
```shell
oc --kubeconfig /home/user/auth/kubeconfig get nodes
```

#### Deleting the Virtual User

```
oc delete secret kubeadmin -n kube-system
```

### Network Security 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: network-svc
  namespace: network-review
  annotations:
    service.beta.openshift.io/serving-cert-secret-name: service-cert
spec:
  selector:
    app: network-svc
  ports:
    - port: 80
      targetPort: 8085
      name: http
```


## OpenShift SDN 
The DNS operator implements the CoreDNS DNS server
* The internal CoreDNS server is used by Pods for DNS resolution
* Use oc describe dns.operator/default to see its config
* The DNS Operator has different roles:
  * Create a default cluster DNS name cluster.local
  * Assign DNS names to namespaces
  * Assign DNS names to services

```
oc describe dns.operator/default
```

* DNS names are composed as servicename.projectname.cluster-dns-name
  * Example: db.myproject.cluster.local
* Apart from the A resource records, CoreDNS also implements an SRV record, in which port name and protocol are prepended to the service A record name
  * Example: _443._tcp.webserver.myproject.cluster.local
* If a service has no IP address, DNS records are created for the IP addresses of the Pods, and round-robin is applied

```
oc get network/cluster -o yaml
```

* Network policy allows defining Ingress and Egress filtering
* If no network policy exists, all traffic is allowed
* If a network policy exists, it will block all traffic with the exception of allowed Ingress and Egress traffic













### Various OpenShift Samples
- https://github.com/OpenShiftDemos
- https://github.com/openshift/origin/

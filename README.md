# Litmus POC

This project setups Litmus and runs on a tiny [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) cluster to learn how [Litmus](https://litmuschaos.io/) works.

----

## Pre-Requisites

- Install Docker Desktop
- Install [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- Install [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

> Note: This guide uses `k` as alias for `kubectl`.
>
> You can set it on your favourite terminal by adding following to your profile
>
> `alias k=kubectl`

----

## Setup Cluster

After installing [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/), start a new k8s cluster:

```sh
kind create cluster --name litmus-poc
```

This will output something similar:

```sh
Creating cluster "litmus-poc" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-litmus-poc"
You can now use your cluster with:

kubectl cluster-info --context kind-litmus-poc
```

You can check your cluster up & running:

```sh
k get pods -A
```

This will output something similar:

```sh
NAMESPACE            NAME                                               READY   STATUS    RESTARTS   AGE
kube-system          coredns-565d847f94-q2f2t                           1/1     Running   0          6m20s
kube-system          coredns-565d847f94-vrsgh                           1/1     Running   0          6m20s
kube-system          etcd-litmus-poc-control-plane                      1/1     Running   0          6m34s
kube-system          kindnet-6j2gx                                      1/1     Running   0          6m21s
kube-system          kube-apiserver-litmus-poc-control-plane            1/1     Running   0          6m34s
kube-system          kube-controller-manager-litmus-poc-control-plane   1/1     Running   0          6m34s
kube-system          kube-proxy-smc2k                                   1/1     Running   0          6m21s
kube-system          kube-scheduler-litmus-poc-control-plane            1/1     Running   0          6m34s
local-path-storage   local-path-provisioner-684f458cdd-v7khv            1/1     Running   0          6m20s
```

## Install Litmus using Kubectl

Create a Litmus namespace in Kubernetes

```sh
k create ns litmus
```

Install Litmus from the manifest

```sh
k apply -f https://litmuschaos.github.io/litmus/2.12.0/litmus-2.12.0.yaml
```

This will output something similar:

```sh
clusterrole.rbac.authorization.k8s.io/argo-cr-for-litmusportal-server created
clusterrolebinding.rbac.authorization.k8s.io/argo-crb-for-litmusportal-server created
clusterrole.rbac.authorization.k8s.io/litmus-cluster-scope-for-litmusportal-server created
clusterrolebinding.rbac.authorization.k8s.io/litmus-cluster-scope-crb-for-litmusportal-server created
clusterrole.rbac.authorization.k8s.io/litmus-admin-cr-for-litmusportal-server created
clusterrolebinding.rbac.authorization.k8s.io/litmus-admin-crb-for-litmusportal-server created
clusterrole.rbac.authorization.k8s.io/chaos-cr-for-litmusportal-server created
clusterrolebinding.rbac.authorization.k8s.io/chaos-crb-for-litmusportal-server created
clusterrole.rbac.authorization.k8s.io/subscriber-cr-for-litmusportal-server created
clusterrolebinding.rbac.authorization.k8s.io/subscriber-crb-for-litmusportal-server created
clusterrole.rbac.authorization.k8s.io/event-tracker-cr-for-litmusportal-server created
clusterrolebinding.rbac.authorization.k8s.io/event-tracker-crb-for-litmusportal-server created
clusterrole.rbac.authorization.k8s.io/litmus-server-cr created
clusterrolebinding.rbac.authorization.k8s.io/litmus-server-crb created
Warning: resource namespaces/litmus is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
namespace/litmus configured
serviceaccount/litmus-server-account created
secret/litmus-portal-admin-secret created
configmap/litmus-portal-admin-config created
configmap/litmusportal-frontend-nginx-configuration created
deployment.apps/litmusportal-frontend created
service/litmusportal-frontend-service created
deployment.apps/litmusportal-server created
service/litmusportal-server-service created
deployment.apps/litmusportal-auth-server created
service/litmusportal-auth-server-service created
statefulset.apps/mongo created
service/mongo-service created
service/mongo-headless-service created
```

After waiting 3-4 minutes, by running `k get pods -n litmus`, you should see all litmus related things started running:

```sh
NAME                                        READY   STATUS    RESTARTS   AGE
litmusportal-auth-server-76499f6bc8-4lprc   1/1     Running   0          2m32s
litmusportal-frontend-6974547497-gd562      1/1     Running   0          2m32s
litmusportal-server-8686d5c747-gtz8q        1/1     Running   0          2m32s
mongo-0                                     1/1     Running   0          2m32s
```

From this point onwards, we will use the ChaosCenter dashboard to set up your first chaos project, verify the installation of chaos delegate services on the cluster and prepare for chaos scenario execution.

## Access and Login to ChaosCenter Dashboard

Run following to obtain the service endpoint of the litmus-frontend service, either via nodePort or LoadBalancer URL and open on your preferred browser.

```sh
k get svc -n litmus
```

This will output something similar:

```sh
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
litmusportal-auth-server-service   NodePort    10.96.11.27     <none>        9003:30690/TCP,3030:31209/TCP   3m53s
litmusportal-frontend-service      NodePort    10.96.196.108   <none>        9091:32449/TCP                  3m53s
litmusportal-server-service        NodePort    10.96.80.81     <none>        9002:32037/TCP,8000:31063/TCP   3m53s
mongo-headless-service             ClusterIP   None            <none>        27017/TCP                       3m53s
mongo-service                      ClusterIP   10.96.173.122   <none>        27017/TCP                       3m53s
```

By default Kind cluster doesn't forward ports to host. In order to access `litmusportal-frontend-service`, you can forward ports by running in another tab:

```sh
k port-forward service/litmusportal-frontend-service 9091:9091 -n litmus
```

This will output the stream of traffic from port 9091 on your machine to `litmusportal-frontend-service`.
Now, open your browser and navigate to `http://localhost:9091`. Now you should see the Litmus dashboard:

![Litmus Dashboard](images/litmus-dashboard.png)

Default credentials for the dashboard is:

```sh
Username: admin
Password: litmus
```

🎉 You should be able to see ChaosCenter dashboard

![Litmus Chaos Center](images/litmus-chaos-center.png)

----

## Verify Successful Registration of the Self Chaos Delegate

Once the project is created, the cluster is automagically registered as a chaos target via installation of Chaos Delegate. This is represented as "Self-Agent " in the Delegate console of the ChaosCenter Dashboard.

```sh
k get pods -n litmus
```

This will output something similar:

```sh
NAME                                        READY   STATUS    RESTARTS   AGE
chaos-exporter-76c5d578bf-gcjfp             1/1     Running   0          3m18s
chaos-operator-ce-5db7fdfd89-m76zc          1/1     Running   0          3m18s
event-tracker-79cfddf9fc-mppxw              1/1     Running   0          3m18s
litmusportal-auth-server-76499f6bc8-4lprc   1/1     Running   0          17m
litmusportal-frontend-6974547497-gd562      1/1     Running   0          17m
litmusportal-server-8686d5c747-gtz8q        1/1     Running   0          17m
mongo-0                                     1/1     Running   0          17m
subscriber-6b7467c76c-mhpw5                 1/1     Running   0          3m19s
workflow-controller-768f7d94dc-s5ns8        1/1     Running   0          3m19s
```

🎉 At this point, we are ready to run chaos experiments!

----

## Installing Litmusctl

The Litmuschaos command-line tool, [litmusctl](https://github.com/litmuschaos/litmusctl), allows you to manage litmuschaos's agent plane. You can use litmusctl to connect Chaos Delegates, create project, schedule Chaos Scenarios, disconnect Chaos Delegates and manage multiple litmuschaos accounts.

Download latest version compatible to your OS from <https://github.com/litmuschaos/litmusctl> and follow installation guide.
Verify your installation:

```sh
litmusctl version
```

This will output something similar:

```sh
Litmusctl version:  0.15.0
Compatible ChaosCenter versions: 
[ '2.9.0' '2.10.0' '2.11.0' '2.12.0' '2.13.0' '2.14.0' '3.0-beta1' ]
```

>**Installation modes**
>
> Litmusctl can install a Chaos Delegate in two different modes.
>
> 1. cluster mode: With this mode, the Chaos Delegate can run the chaos in any namespace. It installs appropriate cluster roles and cluster role bindings to achieve this mode. It can be enabled by passing a flag --installation-mode=cluster
>
> 2. namespace mode: With this mode, the Chaos Delegate can run the chaos in its namespace. It installs appropriate roles and role bindings to achieve this mode. It can be enabled by passing a flag --installation-mode=namespace
>
> ⚠️⚠️⚠️ ⚠️⚠️⚠️ ⚠️⚠️⚠️
>
> **Previously we installed Litmuschaos in `litmus` namespace. So, in this guide, we will install Chaos Delegate in `namespace mode`.**
>
> ⚠️⚠️⚠️ ⚠️⚠️⚠️ ⚠️⚠️⚠️
>

## Configure Listmusctl Config

Litmusctl is using the `.litmusconfig` config file to manage multiple accounts

1. If the --config flag is set, then only the given file is loaded. The flag may only be set once and no merging takes place.
2. Otherwise, the `${HOME}/.litmusconfig` file is used, and no merging takes place.

To setup an account with litmusctl:

```sh
litmusctl config set-account --endpoint="http://localhost:9091" --username="admin" --password="litmus"
```

To verify your setup:

```sh
litmusctl get projects
```

This will output something similar:

```sh
PROJECT ID                            PROJECT NAME     CREATED AT
7f8a0a09-68db-48a7-b2df-92b2f5a5f521  admin's project  2022-12-02 10:58:57 +0000 GMT
```

Take a note of this Project ID or set it in your environment variables for further steps.

```sh
PROJECTID=7f8a0a09-68db-48a7-b2df-92b2f5a5f521
```

----

## Connect to Chaos Delegate

To get an overview of the Chaos Delegates available within a project, run the following command:

```sh
litmusctl get chaos-delegates
```

This will output something similar:

```sh
Enter the Project ID: 7f8a0a09-68db-48a7-b2df-92b2f5a5f521

CHAOS DELEGATE ID                    CHAOS DELEGATE NAME   STATUS   REGISTRATION 
f57d748b-f09a-4a9e-88f3-5c860103f7ca Self-Agent            ACTIVE   REGISTERED
```

Connect to chaos delegate:

```sh
litmusctl connect chaos-delegate --name="Litmus-Poc-Agent" --project-id="" --non-interactive
```

This will output something similar below. You can also see the new delegate on ChaosCenter at <http://localhost:9091/> or running `litmusctl get chaos-delegates --project-id="7f8a0a09-68db-48a7-b2df-92b2f5a5f521"`

```sh
🏃 Running prerequisites check....
🔑 clusterrole ✅
🔑 clusterrolebinding ✅
🌟 Sufficient permissions. Installing the Chaos Delegate...

📌 Summary 
Chaos Delegate Name: Litmus-Poc-Agent
Chaos Delegate Description: ---
Chaos Delegate SSL/TLS Skip: false
Platform Name: Others
Namespace:  litmus
Service Account:  litmus

Installation Mode: cluster
Applying YAML:
http://localhost:9091/api/file/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjbHVzdGVyX2lkIjoiZjhjZDFlYzAtMzk0OC00NTAwLWE0YzYtYWY2ZWRhN2I5MTk1In0.6l6t6vmxdM1SyHCXCKtjT5IImheThOOlKRJylz_mhE8.yaml
namespace/litmus unchanged
serviceaccount/litmus configured
customresourcedefinition.apiextensions.k8s.io/clusterworkflowtemplates.argoproj.io configured
customresourcedefinition.apiextensions.k8s.io/cronworkflows.argoproj.io configured
customresourcedefinition.apiextensions.k8s.io/workflows.argoproj.io configured
customresourcedefinition.apiextensions.k8s.io/workflowtemplates.argoproj.io configured
customresourcedefinition.apiextensions.k8s.io/workflowtasksets.argoproj.io configured
customresourcedefinition.apiextensions.k8s.io/workflowtaskresults.argoproj.io configured
deployment.apps/chaos-operator-ce configured
deployment.apps/chaos-exporter configured
service/chaos-exporter configured
service/workflow-controller-metrics configured
deployment.apps/workflow-controller configured
serviceaccount/argo configured
clusterrole.rbac.authorization.k8s.io/argo-cluster-role configured
clusterrolebinding.rbac.authorization.k8s.io/argo-binding configured
configmap/agent-config configured
secret/agent-secret configured
deployment.apps/subscriber configured
deployment.apps/event-tracker configured
customresourcedefinition.apiextensions.k8s.io/chaosengines.litmuschaos.io configured
customresourcedefinition.apiextensions.k8s.io/chaosexperiments.litmuschaos.io configured
customresourcedefinition.apiextensions.k8s.io/chaosresults.litmuschaos.io configured
customresourcedefinition.apiextensions.k8s.io/eventtrackerpolicies.eventtracker.litmuschaos.io configured
configmap/workflow-controller-configmap configured
serviceaccount/litmus-admin configured
clusterrole.rbac.authorization.k8s.io/litmus-admin configured
clusterrolebinding.rbac.authorization.k8s.io/litmus-admin configured
serviceaccount/argo-chaos configured
clusterrole.rbac.authorization.k8s.io/chaos-cluster-role configured
clusterrolebinding.rbac.authorization.k8s.io/chaos-cluster-role-binding configured
clusterrole.rbac.authorization.k8s.io/subscriber-cluster-role configured
clusterrolebinding.rbac.authorization.k8s.io/subscriber-cluster-role-binding configured
serviceaccount/event-tracker-sa configured
clusterrole.rbac.authorization.k8s.io/event-tracker-cluster-role configured
clusterrolebinding.rbac.authorization.k8s.io/event-tracker-clusterole-binding configured
serviceaccount/litmus-cluster-scope configured
clusterrole.rbac.authorization.k8s.io/litmus-cluster-scope configured
clusterrolebinding.rbac.authorization.k8s.io/litmus-cluster-scope configured
💡 Connecting Chaos Delegate to ChaosCenter.
🏃 Chaos Delegate is running!!

🚀 Chaos Delegate connection successful!! 🎉
👉 Litmus Chaos Delegates can be accessed here: http://localhost:9091/targets

```

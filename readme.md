# Datadog Cluster Agent : Horizontal Pod Autoscaling Exercice
# 1. Setting up the Node Agent

The aim of the first section is to set up the Datadog Agent using the daemonset provided in the `/node-agent`. In order to get this working you'll first need to create a Kubernetes Secret, as well as applying the correct RBAC for the Datadog agent. 

## 1. Setting up the Datadog API key in a Kubernetes Secret

### a. Why use a secret?

In the `/node-agent/datadog-agent.yaml` file, you can see the following configuration under the `env` section :

```
          - name: DD_API_KEY
            valueFrom:
             secretKeyRef:
               name: datadog-secret
               key: api-key 
```

This means that the `DD_API_KEY` environment variable is configured from the `api-key` key in a Kubernetes Secret called `datadog-secret`. Configuring an environment variable through a Secret is very practical: instead of modifying the value of `DD_API_KEY` in every Datadog Agent manifest, you can just modify it in the Secret, which will save you a lot of time.

### b. Create your own secret

To check whether this Secret, `datadog-secret` exists on your cluster, you can execute the following command :

 `kubectl get secrets | awk '{print $1}' | grep datadog-secret`

 This command can be decomposed in 3 steps:
 1. `kubectl get secrets` : list all secrets on your default namespace,
 2. `awk '{print $1}'` : print the values of the first column of the table, i.e. the secret names
 3. `grep datadog-secret` : grep for `datadog-secret`


 If you haven't been through this exercice before, you'll likely not see any results. That's normal, because you have to create it yourself!


The `/node-agent/datadog-secret.yaml` is a definition file for a Secret. To create a secret from it, you'll have to execute the following steps:

1. Replace `<YOUR_DD_API_KEY_IN_BASE_64>` by the result of executing `echo -n <YOUR_DD_API_KEY> | base64`. Any data within the data map of the Secret should be encoded in base 64. If you do not wish to encode your data, sou can use a map called `stringData` instead, which should look like this:

```
apiVersion: v1
kind: Secret
metadata:
  name: datadog-secret
type: Opaque
stringData:
  api-key: <MY_PLAIN_TEXT_DD_API_KEY>

```
2. Execute `kubectl apply -f /node-agent/datadog-secret.yaml`

This should have created a Kubernetes secret with your Datadog API key. To verify this, you can run :

`kubectl get secrets` which will give you : 

```
NAME                        TYPE                                  DATA   AGE
datadog-secret              Opaque                                1      11m
```

You can also run `kubectl describe secret datadog-secret` which will return :

```
Name:         datadog-secret
Namespace:    default
Labels:       app=datadog
Annotations:
Type:         Opaque

Data
====
api-key:  32 bytes
```

If you want to add an extra key in the `data` map of this secret, like your Datadog application key for example, you can simply edit the Secret using `kubectl edit secret datadog-secret`. This will open text editor that will allow you to add a new key under the current `api-key`.


Now that you have your `DD_API_KEY` being populated by the `datadog-secret`, let's move onto RBAC!


## 2. Setting up the correct RBAC for the Datadog Agent

In many cases, your Kubernetes wil have role-based access control enabled. To get the Datadog Agent working in such an environment, you'll need to create a the following :
1. [Datadog Service Account](https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/serviceaccount.yaml) : In other words, a Datadog User which is managed by the Kubernetes API
2. [ClusterRole](https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/clusterrole.yaml) : A Kubernetes object listing a set of permissions to various resources that can be used over a Cluster (In contrast of a `Role` which is defined for a namespace rather than a cluster)
3. [ClusterRoleBinding](https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/clusterrolebinding.yaml) : An object binding the permissions from a ClusterRole to a specific user or set of users. In our case, you'll use the ClusterRoleBinding to bind the ClusterRole we've defined to the Datadog Service Account.


To enable the correct RBAC for the Datadog Agent, simply create the Datadog Service Account, ClusterRole, and ClusterRoleBinding :

```
kubectl create -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/clusterrole.yaml"

kubectl create -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/serviceaccount.yaml"

kubectl create -f "https://raw.githubusercontent.com/DataDog/datadog-agent/master/Dockerfiles/manifests/rbac/clusterrolebinding.yaml"
```

Once that is done, you can go ahead and create the Datadog Agent Daemonset, with `kubectl create -f /node-agent/datadog-agent.yaml`. You're all set now! You should be able to see your Datadog Agent reporting in your account!


## 3. Troubleshooting

If you're not able to see your agent running, there might be several reasons like having a misconfigured secret, or a misconfigured RBAC. In any case, these following commands will be helpful to troubleshoot :

1. **Describe your Kubernetes objects**, may it be your secret, your daemonset, your service account, your clusterrole, etc : `kubectl describe <object_category> <object_name>`
2. **List your Kubernetes objects**. Sometimes you might think that an object -like your daemonset- was created successfully, but listing your Daemonset via `kubectl get daemonsets` will show your otherwise. This command, which can be used with various Kubernetes objects is helfpul to verify the objects running on your cluster.
3. **Exec into your pods**. If you want to dig around the logs, execute agent commands, use `kubectl exec -it <pod_name> <command>`


# Tigera Secure with Azure AKS Workshop

# Overview

Welcome to the Tigera / AKS Tech Workshop

## Prerequisites

You will need the following items to complete the labs.

  * General knowledge of Kubernetes and AKS
  * az CLI installed on your laptop
  * kubectl installed on your laptop
  * OpenSSH public and private RSA keys to add to your AKS cluster
  * Linux subsystem for Windows or a Linux VM could also be handy
  * [Integrate Azure Active Directory with Azure Kubernetes Service](IntegrateAzureActiveDirectory.md)
  * [Deploy Azure AKS cluster with Azure CNI Plugin](AKS-PREP.md)

## Install Tigera Secure Enterprise Edition

### A. Deploy TSEE version of calico-node with AKS cluster

Download the `calico.yaml` file and configure it to start the installation:

```
kubectl apply -f calico.yaml
```

Verify that the calico-node and calico-typha pods are running and ready.

```
watch kubectl get pods --all-namespaces -o wide
```

Deploy calicoctl and apply the license

```
kubectl apply -f calicoctl.yaml
alias calicoctl="kubectl exec -i -n kube-system calicoctl /calicoctl -- "
calicoctl apply -f - < tigera-license-azurecsa.yaml
calicoctl get license
```

### B. Deploy TSEE UI and Aggregate APIServer

Create a cert for cnx-manager (UI). Sample certs are provided for convenience

```
kubectl create secret generic cnx-manager-tls --from-file=cert=cnxmanager.crt --from-file=key=cnxmanager.key -n kube-system
```

Apply the TSEE manifest with the following command: (you should see loads of resources and services created by this command)

```
kubectl apply -f cnx.yaml
```

Verify that the cnx-manager and cnx-apiserver pods are running and ready without restarts

Determine the token to access the Tigera Secure Dashboard

```
export TIGERA_UI_USER=tigera-user
kubectl create serviceaccount -n kube-system $TIGERA_UI_USER
kubectl create clusterrolebinding tigera-user-tigera --clusterrole=tigera-manager-user   --user=tigera-user
kubectl create clusterrolebinding  tigera-user-network-admin   --clusterrole=network-admin   --user=tigera-user
kubectl get secret -n kube-system -o jsonpath='{.data.token}' $(kubectl -n kube-system get secret | grep $TIGERA_UI_USER | awk '{print $1}') | base64 --decode
```

Save the token listed in the last command above, and use the token to log into the UI as described in the next step.

Access the Dashboard as described below.

```
kubectl get svc cnx-manager -n kube-system -o wide
```

Connect to the external Load Balancer IP on port 8080 in your browser, and log in with the token retrieved above.

Create failsafe policies to avoid cutting yourself off from the UI due to incorrectly defined policy

```
kubectl apply -f https://docs.tigera.io/v2.3/getting-started/kubernetes/installation/hosted/cnx/1.7/cnx-policy.yaml
```

### C. Enable Statistics and Flow/Audit Logs with Tigera Secure.

Apply the Prometheus and Elastic operators.

```
kubectl apply -f operator.yaml
kubectl get crd |grep coreos
```

Wait for the CRDs to appear before moving to the next step.

Apply the calico-monitoring manifest with the following command: (you should see loads of resources and services created by this command within the calico-monitoring namespace)

```
kubectl apply -f elastic-storage-local.yaml
kubectl apply -f monitor-calico.yaml
```

Allow Kibana access from Dashboard

```
kubectl get svc tigera-kibana -n calico-monitoring -o wide
```

Patch the tigera-cnx-manager-config configmap and add the load balancer external IP and port 5601 for Kibana in tigera.cnx-manager.kibana-url. Restart cnx-manager to force it to read the new configmap

```
kubectl get svc -n calico-monitoring -o wide
kubectl patch configmap -n kube-system tigera-cnx-manager-config -p $'data:\n  tigera.cnx-manager.kibana-url: http://<Insert-kibana-LB-ExternalIP-here>:5601'
kubectl delete pod -n kube-system cnx-manager-xxxxx
```

Open the Management -> Index Patterns pane in Kibana, select one of the imported index patterns and click the star to set it as the default pattern.

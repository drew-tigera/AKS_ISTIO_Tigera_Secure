# Demos

These two sample demos are provided as examples

-- Peira demo - illustrates malware example

-- Yaobank demo - illustrates Calico Application Layer Policy (requires Istio)


**Demo 1 - Malware Demo**

0. Deploy Peira
```
  kubectl apply -f peira.yaml
  kubectl get pods -n dev
```

Verify that logserver is running

2. Deploy baseline microservices

```
  ./deploy_baseline.sh
  kubectl get pods -n dev
```

Verify that frontend, microservice-1, microservice-2 and database are running

3. Deploy the rogue workload to create malware
```
  ./deploy_rogue.sh
```
4. Verify that policies are denying traffic. You should also see an alert in the Azure-test slack, within the csa-burlington-mar19 channel

5. Go to the Flow Visualizations and verify that the Allowed/Denied status shows traffic from the rogue pod shows as red. You might need to wait a few minutes for the flows to be captured in elastic.

6. Label the pod with the "quarantine=true" label

```
  kubectl label pod -n dev rogue-xxxxxx-xxxx "quarantine=true"
  kubectl get pods -n dev --show-labels
```

7. Verify that traffic is now being filtered by the quarantine policy in the SecOps tier, and at the egress from the rogue pod

8. Go to the Kibana link, and verify that the denied flows show up in flow logs.



**Demo 2 - Yaobank (Showcasing Calico Application Layer Policy with Istio)**



# FPGA Applications on Nautilus

**Mohammad Sada and Justas Balcas**

**Sixth National Research Platform (6NRP) Workshop**

**San Diego Supercomputer Center**

**January 28th, 2025**

# Multus Network Setup on `node-2-6` and `node-2-7`

## Introduction
In this tutorial, we will set up two nodes (`node-2-6` and `node-2-7`) with a Multus CNI configuration for VLAN 1784
and ensure that they can ping each other. This tutorial will walk you through:
- Configuring the network on both nodes
- Creating the necessary NetworkAttachmentDefinition (NAD) for VLAN 1784
- Deploying pods on both nodes
- Testing connectivity between the pods

## Step 1: Setting up the NetworkAttachmentDefinition (NAD)
First, create a NetworkAttachmentDefinition (NAD) for VLAN 1784. This will allow pods to access this VLAN interface.

```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: vlan-1784
  namespace: 6nrp
spec:
  config: |
    {
      "cniVersion": "0.4.0",
      "type": "macvlan",
      "master": "vlan.1784",
      "mode": "bridge",
      "ipam": {
        "type": "host-local",
        "subnet": "10.251.87.160/30",
        "rangeStart": "10.251.87.161",
        "rangeEnd": "10.251.87.162",
        "gateway": "10.251.87.159"
      }
    }
```

Save the YAML as `vlan-1784-nad.yaml`.
Apply the NAD to the cluster:
```bash
kubectl apply -f vlan-1784-nad.yaml
```

## Step 2: Set Up Pods on `node-2-6` and `node-2-7`
Now we need to create pods on `node-2-6` and `node-2-7`. These pods will be attached to the `vlan-1784` network.

### 2.1. Pod on `node-2-6`

Create the pod configuration YAML for `node-2-6` using the `ubuntu` image.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-2-6
  namespace: 6nrp
  annotations:
    k8s.v1.cni.cncf.io/networks: vlan-1784
spec:
  nodeName: node-2-6.sdsc.optiputer.net
  containers:
    - name: my-container
      image: ubuntu:20.04
      command: ["sleep", "3600"]  # Keeps the pod running for 1 hour
      resources:
        requests:
          memory: "64Mi"
          cpu: "100m"
        limits:
          memory: "128Mi"
          cpu: "100m"
```

Save the YAML as `pod-node-2-6-ubuntu.yaml`.
Apply the pod configuration:
```bash
kubectl apply -f pod-node-2-6-ubuntu.yaml
```

### 2.2. Pod on `node-2-7`

Similarly, create the pod configuration YAML for `node-2-7`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-2-7
  namespace: 6nrp
  annotations:
    k8s.v1.cni.cncf.io/networks: vlan-1784
spec:
  nodeName: node-2-7.sdsc.optiputer.net
  containers:
    - name: my-container
      image: ubuntu:20.04
      command: ["sleep", "3600"]  # Keeps the pod running for 1 hour
      resources:
        requests:
          memory: "64Mi"
          cpu: "100m"
        limits:
          memory: "128Mi"
          cpu: "100m"
```

Save the YAML as `pod-node-2-7-ubuntu.yaml`.
Apply the pod configuration:
```bash
kubectl apply -f pod-node-2-7-ubuntu.yaml
```

## Step 3: Installing Necessary Tools in the Pods
Since we're using the `ubuntu:20.04` image, the `ping` tool might not be installed by default. We need to install it inside the pods.

### 3.1. Install `ping` in `pod-node-2-6`

Exec into the `pod-node-2-6` container:
```bash
kubectl exec -it pod-node-2-6 -n 6nrp -- /bin/bash
```

Inside the pod, update the package list and install `iputils-ping`:
```bash
apt-get update
apt-get install iputils-ping -y
```

### 3.2. Install `ping` in `pod-node-2-7`

Exec into the `pod-node-2-7` container:
```bash
kubectl exec -it pod-node-2-7 -n 6nrp -- /bin/bash
```

Inside the pod, update the package list and install `iputils-ping`:
```bash
apt-get update
apt-get install iputils-ping -y
```

## Step 4: Verify Network Connectivity Between Pods
Now that both pods have the necessary tools installed, we can test connectivity between them.

### 4.1. Ping from `pod-node-2-6` to `pod-node-2-7`

Inside `pod-node-2-6`, ping `pod-node-2-7`:
```bash
ping 10.251.87.162  # IP of pod-node-2-7
```

### 4.2. Ping from `pod-node-2-7` to `pod-node-2-6`

Inside `pod-node-2-7`, ping `pod-node-2-6`:
```bash
ping 10.251.87.161  # IP of pod-node-2-6
```

## Step 5: Troubleshooting

If the ping tests fail, follow these steps to troubleshoot:
- Ensure that both pods are attached to the correct VLAN network and have IP addresses within the same subnet.
- Verify that the network interfaces on the nodes (`node-2-6` and `node-2-7`) are correctly configured.
- Check for any firewall rules or network policies that could be blocking traffic between the pods.

## Conclusion
You have successfully set up two pods on `node-2-6` and `node-2-7` attached to VLAN 1784, and you verified connectivity between them using `ping`.
This setup can now be extended for further experiments involving communication between pods across nodes in the same VLAN.

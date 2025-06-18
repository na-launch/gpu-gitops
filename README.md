
# GPU GitOps for OpenShift / Kubernetes

This repository uses GitOps to deploy NVIDIA GPU support into your cluster. It automates:

- 👉 **Node Feature Discovery (NFD)** to detect GPU nodes  
- 👉 **NVIDIA GPU Operator** to install drivers, device plugin, runtime, monitoring, etc.
- 👉 **NVIDIA DCGM Dashboard** to visualize GPU telemetry exposed in Prometheus by using the NVIDIA DCGM Exporter.

## ✅ Prerequisites

1. A Kubernetes or OpenShift cluster with at least one GPU-capable node.  
2. `oc` (or `kubectl`) logged into the cluster as a cluster-admin.  
3. `kustomize` CLI (optional if you use `oc apply -k`).

## 💡 How it works

1. Deploys NFD → adds GPU labels to nodes (`feature.node.kubernetes.io/pci-10de.present`).  
2. GPU Operator watches GPU-labeled nodes → installs NVIDIA drivers, NVIDIA Container Toolkit, device plugin, DCGM, etc.

You can validate GPU detection with:

```bash
oc describe node | grep pci-10de
```

And test GPU operator with:

```bash
oc -n nvidia-gpu-operator get pods
oc exec -n nvidia-gpu-operator \
  $(oc get pod -l app=nvidia-driver -o jsonpath='{.items[0].metadata.name}') \
  -- nvidia-smi
```

---

## 🛠️ Deploying via GitOps

You can bootstrap these components declaratively without cloning this repo with:

```bash
# 1. Log in
oc login <your-cluster-url> --token=<token> --server=<api-url>

# 2. Bootstrap GPU GitOps
oc apply -k https://raw.githubusercontent.com/na-launch/gpu-gitops/refs/heads/main/gitops/manifests/cluster/bootstrap/base/bigbang-app.yaml

# 3. Monitor progress
oc get pods -n nvidia-gpu-operator -w
oc get nodes -l feature.node.kubernetes.io/pci-10de.present=true -o name

# 4. Verify GPU functionality
oc -n nvidia-gpu-operator get pods -l openshift.driver-toolkit=true -o name | while read pod; do
  echo "$pod"
  oc exec -n nvidia-gpu-operator -it "$pod" -- nvidia-smi
  echo
done
```

---

## 🛡️ Next Steps

Once the GPU infrastructure is installed:

- Create GPU-capable workloads using `resources.limits.nvidia.com/gpu: 1`.  
- Integrate with ML platform (e.g. OpenDataHub or OpenShift AI).  
- Deploy monitoring tools like Prometheus to collect DCGM metrics from GPUs.  
- Scale your GPU node pools and GPU workloads using standard GitOps.

---

## 🔗 References

- NVIDIA GPU Operator + NFD setup with GitOps workflows  
- GPU detection using NFD labels (`pci-10de`)  
- Typical GPU Operator installation patterns and validation via `nvidia-smi`

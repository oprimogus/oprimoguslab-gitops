# Setup a Talos Cluster

## Control Plane
export CONTROL_PLANE_IP=<your-control-plane-ip>
export YOUR_ENDPOINT=<your-control-plane-ip>
export CLUSTER_NAME=<your_cluster_name>
export TALOSCONFIG=./talosconfig

Imagem modificada: factory.talos.dev/metal-installer/88d1f7a5c4f1d3aba7df787c448c1d3d008ed29cfb34af53fa0df4336a56040b:v1.13.9

```bash
talosctl gen secrets -o secrets.yaml
```

```bash
talosctl get disks --insecure --nodes $CONTROL_PLANE_IP
```

```bash
talosctl gen config --with-secrets secrets.yaml $CLUSTER_NAME https://$CONTROL_PLANE_IP:6443
```

```bash
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file controlplane.yaml
```

```bash
talosctl --talosconfig=./talosconfig config endpoints $CONTROL_PLANE_IP
```

```bash
talosctl bootstrap --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig
```

```bash
talosctl kubeconfig --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig
```

```bash
talosctl --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig health
```

```bash
kubectl get nodes
```

## Worker nodes

```bash
export WORKER_IP=<your-worker-ip>
```

```bash
talosctl apply-config --insecure --nodes $WORKER_IP --file worker.yaml
```

## Upgrade Talos

```bash
talosctl --talosconfig=./projects/cluster/talosconfig upgrade -n 192.168.18.44
```

## Install argocd

```bash
kubectl create namespace argocd
```

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

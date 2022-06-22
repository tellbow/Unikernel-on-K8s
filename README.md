# Unikernel-on-K8s

Strongly inspired by https://hackernoon.com/running-linux-applications-as-unikernels-with-k8s-gl2p3yfr

## Requirements

* xz (sudo apt-get install xz-utils)
* docker
* [kind](https://kubernetes.io/docs/tasks/tools/#kind)
* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
* [Ops](https://ops.city/)
* [Virtctl](https://kubevirt.io/quickstart_kind/#virtctl)

## Build

1. Run and test the image

```
GOOS=linux go build main.go
ops run -n -p 8083 main
```

2. Move and put image in format for K8s to understand

```
cp $HOME/.ops/images/main.img .
xz main.img
```

3. Upload image to a bucket, CDN or something similar

## Installation

### K8s cluster

`kind create cluster --name unikernel-k8s`

### [KubeVirt](https://kubevirt.io/quickstart_kind/)

1. Install

```
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
echo $VERSION
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```

2. Verify

```
kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"
kubectl get all -n kubevirt
```

### [Containerized Data Importer (CDI)](https://kubevirt.io/labs/kubernetes/lab2.html)

1. Install

```
export VERSION=$(curl -sL https://github.com/kubevirt/containerized-data-importer/releases/latest | grep -o -m1 "v[0-9]\.[0-9]*\.[0-9]*")
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml
```

2. Verify

```
kubectl get cdi cdi -n cdi
kubectl get pods -n cdi
```

#### Disk Image

1. Update `cdi.kubevirt.io/storage.import.endpoint` to the URL where the image is stored
2. Optionally change the name of the PVC by modifying `name`
3. `kubectl create -f manifests/pvc_main_go.yaml`
4. Check if the storage Pod for the PVC could be created: `kubectl get pvc main-go -o jsonpath="{.metadata.annotations.cdi\.kubevirt\.io/storage\.pod\.phase}"`

#### Virtual Machine

1. Change `claimName` to the name of the PVC (if modified)
2. `kubectl create -f manifests/vm1_pvc.yaml`
3. Check `Phase` of Virtual Machine by `kubectl get vmi` and copy the IP address

### Access

1. SSH into the control-plane of the kind cluster
2. `curl <ip-address>:8083`
- [Local Kubernetes environment](#org5ee8a74)
  - [Create Kubernetes environment](#org50a76cb)
  - [Destroy Kubernetes environment](#org68b430a)
- [Deploying Ingress controller](#org38bfd3e)
- [Deploying cert-manager](#org84925c3)
  - [Deploying root CA certificate](#orgc0e06b8)
  - [Creating CA issuer](#org0218e53)
- [Deploying ArgoCD](#orge322ae4)
  - [With kustomize](#org824838a)
- [Deploying Prometheus Operator](#org350f103)


<a id="org5ee8a74"></a>

# Local Kubernetes environment


<a id="org50a76cb"></a>

## Create Kubernetes environment

Options:

-   Microk8s
-   K3s
-   Minikube
-   Minishift
-   **Kind**

Checking the kubeconfig:

```bash
echo $KUBECONFIG
```

Kind config:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```

To create a new cluster:

```eshell
kind create cluster --name local-test --config kind-config.yaml
```

```bash
kubectl cluster-info --context kind-local-test
```


<a id="org68b430a"></a>

## Destroy Kubernetes environment

```eshell
kind delete cluster --name kind-local-test
```


<a id="org38bfd3e"></a>

# Deploying Ingress controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```


<a id="org84925c3"></a>

# Deploying cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.3/cert-manager.yaml
```


<a id="orgc0e06b8"></a>

## Deploying root CA certificate

```yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJaVENDQVF1Z0F3SUJBZ0lSQU1NdHZXZHNSRVdQUDEyYUxDU3VSRG93Q2dZSUtvWkl6ajBFQXdJd0VqRVEKTUE0R0ExVUVBeE1IY205dmRDMWpZVEFlRncweU5EQTVNalV4TkRFNU1qaGFGdzB5TkRFeU1qUXhOREU1TWpoYQpNQkl4RURBT0JnTlZCQU1UQjNKdmIzUXRZMkV3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVM1CkNkR2hWRmQrTUM5aWo0dkRHZ1EzS0hlVHN2ek1wN1doaHFYNS9rQllaQnlNVDJuVjF6c1BEZ3B1eDdGekFmdGYKRkNRVzhzbEtZc2RwZzg3eFFwQUdvMEl3UURBT0JnTlZIUThCQWY4RUJBTUNBcVF3RHdZRFZSMFRBUUgvQkFVdwpBd0VCL3pBZEJnTlZIUTRFRmdRVXlZMTd6NkY5c01LZEJ1TDN1a2tiMkQ1ZlZSUXdDZ1lJS29aSXpqMEVBd0lEClNBQXdSUUloQUtOTTZMMmlZLzNuUFFGL0Z6ZnRWcVMrS1J1TlBYbTVoY1ZDRmcxYlZxTGtBaUFiWCs3dVNML0sKOXFtS1dtdWQvRnd1QThmQVhXaVM1R0tVckZPTHZ4QklpUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSUp0REU4UFFIRk9VVkpGSnk4Zis2RU1ieUQ4YlY2b3BRRGZFUi8rQjhGd0JvQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFdVFuUm9WUlhmakF2WW8rTHd4b0VOeWgzazdMOHpLZTFvWWFsK2Y1QVdHUWNqRTlwMWRjNwpEdzRLYnNleGN3SDdYeFFrRnZMSlNtTEhhWVBPOFVLUUJnPT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=
kind: Secret
metadata:
  name: root-ca
type: kubernetes.io/tls
```

```bash
kubectl apply -f root-ca-cert-secret.yaml -n argocd
```


<a id="org0218e53"></a>

## Creating CA issuer

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-cluster-issuer
spec:
  ca:
    secretName: root-ca
```

```bash
kubectl apply -f ca-issuer.yaml -n argocd
```


<a id="orge322ae4"></a>

# Deploying ArgoCD

Options:

-   `kubectl apply -f`
-   kustomize
-   Helm
-   Helmfile


<a id="org824838a"></a>

## With kustomize

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: argocd
resources:
- https://raw.githubusercontent.com/argoproj/argo-cd/v2.7.2/manifests/install.yaml
- namespace.yaml
- ingress.yaml
```

To diff the output:

```bash
kustomize build argocd/base | kubectl diff -f -
```

To perform the deployment:

```bash
kustomize build argocd/base | kubectl apply -f -
```


<a id="org350f103"></a>

# Deploying Prometheus Operator

Options:

-   `kubectl apply -f` from yaml
-   `helm install`
-   helmfile
-   ArgoCD + one of above

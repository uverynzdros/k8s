# Argo

## Done list

```shell
multipass launch --name microk8s --mem 4G --disk 50GB
multipass shell microk8s
sudo snap install microk8s --classic
sudo snap install docker
microk8s.enable dns
sudo snap alias microk8s.kubectl kubectl
exit
multipass exec microk8s -- /snap/bin/microk8s.config > kubeconfig
```

I have used `--kubeconfig` flag for running all command. If you don't want to, set your kubeconfig to `KUBECONFIG` environmet variable.

- Setup kubernetes components by `kubectl`.

```shell
kubectl --kubeconfig=kubeconfig create ns argo
kubectl --kubeconfig=kubeconfig apply -f https://raw.githubusercontent.com/argoproj/argo/v2.2.1/manifests/install.yaml
kubectl --kubeconfig=kubeconfig create rolebinding default-admin --clusterrole=admin --serviceaccount=default:default
```

- Argo workflows

```shell
# This workflow need installing `docker`.
# argo submit --kubeconfig=kubeconfig --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
argo submit --kubeconfig=kubeconfig --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/coinflip.yaml
argo submit --kubeconfig=kubeconfig --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/loops-maps.yaml
argo list --kubeconfig=kubeconfig
argo get --kubeconfig=kubeconfig hello-world-xxx
argo logs --kubeconfig=kubeconfig hello-world-xxx
argo delete --kubeconfig=kubeconfig hello-world-xxx
```

- Argo UI

```shell
kubectl --kubeconfig=kubeconfig -n argo port-forward deployment/argo-ui 8001:8001
```

- Create Google Cloud Storage bucket
- Enable interoperability
- Create gcs key
- Create secret for argo artifacts of gcs by `kustomize`.

```shell
brew install kustomize
```

- From Kubernetes 1.14, kubectl can apply `kustomizaiton.yaml`.

```shell
kubectl --kubeconfig=kubeconfig apply -k ./
```

- Edit argo workflow artifacts config map

```shell
KUBE_EDITOR="code" kubectl --kubeconfig=kubeconfig edit cm -n argo workflow-controller-configmap
# Add below.
data:
  config: |
    artifactRepository:
      s3:
        bucket: gcs-bucket
        endpoint: storage.googleapis.com
        accessKeySecret:
          name: argo-gcs-credentials-v1
          key: argo-gcs-access-key
        secretKeySecret:
          name: argo-gcs-credentials-v1
          key: argo-gcs-secret-key
```

- Run workflow uses artifacts

```shell
argo --kubeconfig=kubeconfig submit gcs-artifact.yaml
```

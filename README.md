# cluster-api-provider-dind

POC of cluster api using Docker in Docker

Quick notes taken during the creation of the cluster-api-provider-dind scaffolding creation

## Prerequesist

Install dep

```bash
curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
sudo mv $HOME/bin/dep /usr/bin
```

Install kustomize

```bash
wget https://github.com/kubernetes-sigs/kustomize/releases/download/v1.0.11/kustomize_1.0.11_linux_amd64
mv kustomize_1.0.11_linux_amd64 kustomize
chmod +x kustomize 
sudo mv kustomize /usr/bin/
```

Install kubebuilder (contains a copy of kubectl, etcd and kubeapi-server)

```bash
wget https://github.com/kubernetes-sigs/kubebuilder/releases/download/v1.0.8/kubebuilder_1.0.8_linux_amd64.tar.gz
tar xvf kubebuilder_1.0.8_linux_amd64.tar.gz 
mv kubebuilder_1.0.8_linux_amd64 kubebuilder
sudo mv kubebuilder /usr/local
```

Ensure the local kubelete and kubernetes cluster is stopped

```bash
sudo systemctl stop kubelet
docker stop $(docker ps -qa)
export PATH=$PATH:/usr/local/kubebuilder/bin
```

## Creation of scaffolding

### Generation steps

```bash
cd cluster-api-provider-dind/
```

```bash
kubebuilder init --domain kubekit.cloud --license apache2 --owner "The Kubernetes Authors"
git add .
git commit -m "Generate scaffolding."
git push
```

```bash
kubebuilder create api --group dind --version v1alpha1 --kind AirshipClusterProviderSpec
kubebuilder create api --group dind --version v1alpha1 --kind AirshipClusterProviderStatus
kubebuilder create api --group dind --version v1alpha1 --kind AirshipMachineProviderStatus
kubebuilder create api --group dind --version v1alpha1 --kind AirshipMachineProviderSpec
vi cmd/manager/main.go 
mkdir -p pkg/cloud/dind/actuators/cluster/
mkdir -p pkg/cloud/dind/actuators/machine/
vi pkg/cloud/dind/actuators/cluster/actuator.go
vi pkg/cloud/dind/actuators/machine/actuator.go
vi pkg/controller/add_cluster_controller.go
vi pkg/controller/add_machine_controller.go
vi Makefile 
dep ensure
export IMG=kubekit99/cluster-api-provider-dind
dep ensure
make
make docker-build IMG=${IMG}
git add .
git commit -m "Add CRDs and build image"
git push
```

### Cleanup steps

If you used brute force, git add . the repository becomes bulky. In order to keep the
scrict minimum

```bash
git rm -fr vendor/
git rm -fr bin/
git commit
git push origin --force --all
```

```bash
git filter-branch --index-filter "git rm -rf --cached --ignore-unmatch vendor" HEAD
git push origin --force --all
git clone https://github.com/kubekit99/cluster-api-provider-dind.git
```

```bash
git filter-branch --index-filter "git rm -rf --cached --ignore-unmatch bin" HEAD
git push origin --force --all
git clone https://github.com/kubekit99/cluster-api-provider-dind.git
```

## Rebuild images

List of commands used to rebuild and publish the images.

```bash
export IMG=kubekit99/cluster-api-provider-dind
dep ensure
make
make docker-build IMG=${IMG}
make docker-push IMG=${IMG}
```

## Deployment and tests

List of commands to deploy and test.

Ensure that /usr/local/kubebuilder is no longer in your path if you are using the same machine


```bash
kubectl get nodes
kubectl apply -f provider-components.yaml
kubectl get all --all-namespaces
kubectl logs pod/cluster-api-provider-dind-controller-manager-0 -n cluster-api-provider-dind-system manager
```

TBD




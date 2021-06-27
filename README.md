# Kubernetes cluster on vagrant using kubeadm

## How to start cluster

1. install vagrant `brew intall vagrant`
2. run `vagrant up` and wait until provisioning is completed
3. install k9s `brew innstall k9s`
4. configure kubeconfig `export KUBECONFIG=./config`
5. run k9s to explore cluster `k9s`

## current issues

| issue | status | details | solution |
|-|-|-|-|
| master provisioning fails | done | master preflight checks fail | increase memory and cpu for vms |
| cni plugin is missing | done | flannel is not installing correctly | add cni plugin installation to master provisioning `kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version \| base64 \| tr -d '\n')` |
| workers fail to join | done | discovery.bootstrapToken: Invalid value: "": using token-based discovery without caCertHashes can be unsafe. Set unsafeSkipCAVerification as true in your kubeadm config file or pass --discovery-token-unsafe-skip-ca-verification flag to continue | __workaround:__ add `--discovery-token-unsafe-skip-ca-verification` to `/vagrant/join.sh` and `vagrant ssh workwr1` and run `sudo /vagrant/koin.sh`. <br/> __possible solution__: use `kubeadm token create --print-join-command` to generate join command with hashes |
| cluster apps provisioning | done | - | add argocd installation to provisioning process |
| kubeconfig is not available outside cluster | done | `vagrant ssh master` and `cp ~/.kube/config /vagrant`  is required to bring kubeconfig to the host machine | copy kubeconfig to `/vagrant` during provisioning |

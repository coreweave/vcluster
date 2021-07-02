# Contributing to vcluster

Thank you for contributing to vcluster! Here you can find common questions around developing vcluster.

## Developing

We recommend to develop vcluster directly in a Kubernetes cluster as it makes feedback a lot quicker. For the quick setup, you'll need to install [devspace](https://github.com/loft-sh/devspace#1-install-devspace), kubectl, helm and make sure you have a local Kubernetes cluster (such as Docker Desktop, minikube, KinD or similar) installed.

Clone the vcluster project into a new folder and make sure the variable `SERIVE_CIDR` in the `devspace.yaml` fits your local clusters service cidr. You can find out the service cidr with:

```
kubectl create -f ./hack/wrong-cluster-ip-service.yaml 
The Service "service-simple-service" is invalid: spec.clusterIPs: Invalid value: []string{"1.1.1.1"}: failed to allocated ip:1.1.1.1 with error:provided IP is not in the valid range. The range of valid IPs is 10.96.0.0/12
```

In this case the service cidr would be `10.96.0.0/12`. Please make also sure you use an adequate `K3S_IMAGE` version that matches your local cluster version.

After adjusting the variables and you can run:

```
devspace run dev
```

Which should produce an output similar to:

```
[info]   Using namespace 'vcluster'
[info]   Using kube context 'docker-desktop'
[done] √ Created image pull secret vcluster/devspace-auth-ghcr-io   
[info]   Building image 'ghcr.io/loft-sh/loft-enterprise/dev-vcluster:szFLbkA' with engine 'docker'
Sending build context to Docker daemon  52.71MB
Step 1/14 : FROM golang:1.15 as builder
 ---> 53b7b7a65524
...
Step 14/14 : ENTRYPOINT ["go", "run", "-mod", "vendor", "cmd/vcluster/main.go"]
 ---> Running in 7156169fe7d7
 ---> b14eacaa5e29
Successfully built b14eacaa5e29
Successfully tagged ghcr.io/loft-sh/loft-enterprise/dev-vcluster:szFLbkA
[info]   Skip image push for ghcr.io/loft-sh/loft-enterprise/dev-vcluster
[done] √ Done processing image 'ghcr.io/loft-sh/loft-enterprise/dev-vcluster'                         
[info]   Execute 'helm upgrade vcluster ./chart --namespace vcluster --values /var/folders/bc/qxzrp6f93zncnj1xyz25kyp80000gn/T/079539791 --install --kube-context docker-desktop'
[info]   Execute 'helm list --namespace vcluster --output json --kube-context docker-desktop'
[done] √ Deployed helm chart (Release revision: 1)              
[done] √ Successfully deployed vcluster with helm               
                                             
#########################################################
[info]   DevSpace UI available at: http://localhost:8090
#########################################################

[0:sync] Waiting for pods...
[0:sync] Starting sync...
[0:sync] Sync started on /Users/fabiankramm/Programmieren/go-workspace/src/github.com/loft-sh/vcluster <-> . (Pod: vcluster/vcluster-0)
[0:sync] Waiting for initial sync to complete
[info]   Opening shell to pod:container vcluster-0:syncer
root@vcluster-0:/vcluster#
```

Then you can start vcluster with
```
go run -mod vendor cmd/vcluster/main.go
```

Now if you change a file locally, DevSpace will automatically sync the file into the container and you just have to rerun `go run -mod vendor cmd/vcluster/main.go` within the terminal to apply the changes.

You can access the vcluster by running `devspace enter` in a separate terminal:

```
devspace enter -n vcluster

? Which pod do you want to open the terminal for? vcluster-0:syncer
[info]   Opening shell to pod:container vcluster-0:syncer
root@vcluster-0:/vcluster# kubectl get ns
NAME              STATUS   AGE
default           Active   2m18s
kube-system       Active   2m18s
kube-public       Active   2m18s
kube-node-lease   Active   2m18s
root@vcluster-0:/vcluster#
```

`kubectl` within the syncer container will point to the virtual cluster and you can access it from there. If you need to recreate the vcluster, delete the `vcluster` namespace and rerun `devspace run dev` again. 

## Tests

You can run the test suite with `./hack/test.sh` which should execute all the vcluster tests.
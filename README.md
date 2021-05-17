# kc - Swiss army knife for kubectl

## Namespace & Pod Patterns

In the following, `namespace` and `pods` can be a sub-pattern which is used by `grep` to determine the correct namespace and/or pod name. 

**Example**:

The following will match `kube-apiserver-k8s-prod-m-1` in the namespace `kube-system`:
```
$ kc sys/api
Containers in kube-system/kube-apiserver-k8s-prod-m-1:
- kube-apiserver: k8s.gcr.io/kube-apiserver:v1.20.4 2
```

## Usage

### List & Watch Resources

* List pods of matching namespace:
  ```
  kc namespace
  ```

  **Example**:
  ```
  $ kc sys
  Pods in kube-system:
  NAME                                   READY   STATUS    RESTARTS   AGE    IP             NODE           NOMINATED NODE   READINESS GATES
  cilium-27wvg                           1/1     Running   3          74d    10.11.0.12     k8s-prod-w-2   <none>           <none>
  cilium-62n46                           1/1     Running   2          74d    10.11.0.2      k8s-prod-m-2   <none>           <none>
  cilium-bxnbp                           1/1     Running   1          51d    10.11.0.1      k8s-prod-m-1   <none>           <none>
  cilium-chkrb                           1/1     Running   3          74d    10.11.0.14     k8s-prod-w-4   <none>           <none>
  cilium-dw2xt                           1/1     Running   0          11d    10.11.0.11     k8s-prod-w-1   <none>           <none>
  ```

* To list containers and their status from namespace/pod:
  ```
  kc namespace/pod
  ```

  **Example**:
  ```
  $ kc sys/cilium
  Containers in kube-system/cilium-27wvg:
  - (init) clean-cilium-state: docker.io/cilium/cilium:v1.7.6 0
  - cilium-agent: docker.io/cilium/cilium:v1.7.6 3
  ```
  
  This will show you the used image, version and the restart count for `containers` and `initContainers`.


* To delete the selected pod:
  ```
  kc namespace/pod delete
  ```

* Describe the selected pod:
  ```
  kc namespace/pod describe
  ```

* Show the labels of the selected pod:
  ```
  kc namespace/pod labels
  ```

* To watch resources having the given status or not, default is "pod:!Succeeded":
  ```
  kc namespace watch resource[:status] [resource[:!status]]
  ```

  **Example**:
  Watching deployments and pods in namespace `kube-system`:
  ```
  kc sys watch deploy pods
  ```

### Executing  

* Exec into a container: (see kubectl exec --help)
  ```
  kc namespace/pod[/container] exec [command]  
  ```

  By default this command will try to use `bash` and fallback to `sh`.

  **Example**:

  Execute to container `main` of the first matching pod:
  ```
  kc playground/wordpress/main exec
  ```

### Logging

* Following log of a container (see kubectl logs --help):  
  ```
  kc namespace/pod[/container] logs
  ```

* If you have the excellent [stern tool](https://github.com/wercker/stern) installed, `kc` will use `stern` by default:
  ```
  kc namespace/pod logs [stern params ...]
  ```

  **Example**:

  To get the logs from all pods matching `api` in namespace `kube-system` since 10s, do as follows:
  ```
  kc sys/api logs -s 10s
  ```

### Delete Namespaces

* To delete the selected namespace. Use "force" to force-delete by leaving possible dangling resources after wait duration has expired (default 20s):
  ```
  kc namespace delete [force [wait_duration]]
  ```

  **Example**:
  This will delete the namespace `playground`. After 10s, it will force-delete the namespace.
  ```
  kc playground delete force 10
  ```

### Starting Maintenance Containers

* To start a maintenance container (default: praqma/network-multitool):
  ```
  kc namespace maintenance [container] [command]
  ```

* To start the maintenance container on a specifiy node, export NODE as follows:
  ```
  export NODE='"kubernetes.io/hostname": "k8s-prod-m-1"'
  kc namespace maintenance [container] [command]
  ```

* To override node tolerations, export TOLERATIONS as follows:
  ```
  export TOLERATIONS='[{"effect": "NoSchedule", "key": "node-role.kubernetes.io/master"}]'
  kc namespace maintenance [container] [command]
  ```

### Files Upload & Editing

* Local edit of the file /path/to/edit in the selected container:
  ```
  kc namespace/pod/container edit /path/to/edit
  ```

* Upload a file into the container:
  ```
  kc namespace/pod/container (upload|up) /local/file /remote/file
  ```

* Download a file from the container:
  ```
  kc namespace/pod/container (download|down) /remote/file /local/file
  ```

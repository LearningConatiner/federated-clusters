
# Federated cluster set up

## Steps

1. create clusters with required properties.

2. Do below,
```
$ gcloud config set container/use_client_certificate True
$ export CLOUDSDK_CONTAINER_USE_CLIENT_CERTIFICATE=True
$ gcloud container clusters get-credentials <cluster name> --zone=us-west1-a --project=<project name>
```

3. Install kubefed
https://kubernetes.io/docs/tasks/federation/set-up-cluster-federation-kubefed/

4. To get contexts `kubectl config get-contexts`
5. To unset contexts `kubectl config unset contexts.gke_prabhatrial_us-east1-c_p` where gke_prabhatrial_us-east1-c_p is context name.

## clusters cleanup

While deleting cluster, node, 

delete loadbalancer entries, kube config entries.


## Debugging
https://github.com/kubernetes/kubernetes/issues/42559

### To search contents in specific context

```kubefed init fellowship \
>    --host-cluster-context=us-central \
>    --dns-provider="google-clouddns" \
>    --dns-zone-name="example.com."
Error from server (AlreadyExists): namespaces "federation-system" already exists
# kubectl get namespaces --context=us-central
NAME                STATUS    AGE
default             Active    5d
federation-system   Active    34m
kube-public         Active    5d
kube-system         Active    5d
# kubectl delete namespace federation-system  --context=us-central
```

### Authorization error

```  "message": "roles.rbac.authorization.k8s.io \"federation-system:federation-controller-manager\" is forbidden: attempt to gra
nt extra privileges: [{[get] [] [secrets] [] []} {[list] [] [secrets] [] []} {[watch] [] [secrets] [] []}] user=\u0026{prabha.
techsub@gmail.com  [system:authenticated] map[]} ownerrules=[{[create] [authorization.k8s.io] [selfsubjectaccessreviews] [] []
} {[get] [] [] [] [/api /api/* /apis /apis/* /healthz /swaggerapi /swaggerapi/* /version]}] ruleResolutionErrors=[]",
  "reason": "Forbidden",
  "details": {
    "name": "federation-system:federation-controller-manager",
    "group": "rbac.authorization.k8s.io",
    "kind": "roles"
  },
  "code": 403
}]
F0622 06:24:35.521292     546 helpers.go:119] Error from server (Forbidden): roles.rbac.authorization.k8s.io "federation-syste
m:federation-controller-manager" is forbidden: attempt to grant extra privileges: [{[get] [] [secrets] [] []} {[list] [] [secr
ets] [] []} {[watch] [] [secrets] [] []}] user=&{prabha.techsub@gmail.com  [system:authenticated] map[]} ownerrules=[{[create]
 [authorization.k8s.io] [selfsubjectaccessreviews] [] []} {[get] [] [] [] [/api /api/* /apis /apis/* /healthz /swaggerapi /swa
ggerapi/* /version]}] ruleResolutionErrors=[] 
```

*Ensure to do this*
After cluster creation,

```
$ gcloud config set container/use_client_certificate True
$ export CLOUDSDK_CONTAINER_USE_CLIENT_CERTIFICATE=True
$ gcloud container clusters get-credentials pv2 --zone=us-west1-a --project=prabhatrial
```
pv2 -> cluster name
prabhatrial -> project name

Then install kubefed as in https://kubernetes.io/docs/tasks/federation/set-up-cluster-federation-kubefed/
and deploy federation control plane.

```
$ kubectl get ns
NAME                STATUS    AGE
default             Active    9m
federation-system   Active    44s
kube-public         Active    9m
kube-system         Active    9m
$ kubectl get svc --namespace federation-system
NAME               CLUSTER-IP      EXTERNAL-IP     PORT(S)         AGE
prabha-apiserver   10.59.253.121   35.185.237.66   443:31898/TCP   1m
$ kubectl describe namespace federation-system
Name:           federation-system
Labels:         <none>
Annotations:    <none>
Status:         Active
No resource quota.
No resource limits.
$ kubectl get deployment --namespace federation-system
NAME                        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
prabha-apiserver            1         1         1            1           1m
prabha-controller-manager   1         1         1            1           1m
$ kubectl get pods --namespace federation-system
NAME                                        READY     STATUS    RESTARTS   AGE
prabha-apiserver-2578827812-80fdf           2/2       Running   0          2m
prabha-controller-manager-225968991-37f92   1/1       Running   1          2m
```


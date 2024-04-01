# local dev


1. 使用 Kind 创建本地 cluster
2. 在 Kind cluster 运行 OLM
3. 本地运行 OLM 

```shell
kind create cluster
kind export kubeconfig

make run-local
```

## example
通过创建资源，调试代码

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: devworkspace-operator-catalog
  namespace: openshift-operators
spec:
  displayName: DevWorkspace Operator Catalog
  image: 'quay.io/devfile/devworkspace-operator-index:release'
  publisher: Red Hat
  sourceType: grpc
  updateStrategy:
    registryPoll:
      interval: 5m
---
# global operatorgroup (watch all namespaces)
kind: OperatorGroup
apiVersion: operators.coreos.com/v1
metadata:
  name: og-test
  namespace: openshift-operators
spec: {}
---

apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: devworkspace-operator
  namespace: openshift-operators
spec:
  channel: fast
  installPlanApproval: Automatic
  name: devworkspace-operator
  source: devworkspace-operator-catalog
  sourceNamespace: openshift-operators
  startingCSV: devworkspace-operator.v0.26.0
```


## reference
- https://duyanghao.github.io/kubernetes-remote-debug/
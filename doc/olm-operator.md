# olm-operator

OLM Operator 负责 `ClusterServiceVersion`、`OperatorGroup` CRDs资源。
OLM Operator 负责安装 ClusterServiceVersion 中定义的应用程序。

## code overview

### entrypoint

[启动 OLM Operator 程序](../cmd/olm)。

### core

#### 同步 ClusterServiceVersion 逻辑
监听 `ClusterServiceVersion` 资源进行 [control loop 处理](../pkg/controller/operators/olm/operator.go#L1284)。

CSV 同步逻辑(`syncClusterServiceVersion()`)：
1. 判断是否是 Copied CSV，如果是不做处理。
2. CSV 状态机逻辑
3. 判断 2 处理后 CSV 状态是否更新，如果是更新 CSV 对象信息
4. 根据 CSV 获取 operator group，以及判断 CSV 有没有 target namespace，如果没有结束同步
   i. CSV annotation 中包含 operator group 相关标识信息
5. 存在 target namespace 则需要处理 RBAC

CSV 状态机逻辑(`transitionCSVState()`)：
1. 根据 CSV `spec.customresourcedefinitions.owned`, `spec.customresourcedefinitions.required` 信息创建 `operatorSurface` 对象
2. 更新 CSV label
3. 验证 operatorgroup 或更新 csv annotation
   i. 根据 CSV namespace 查询 operator group，之后在 CSV annotation 添加 operator group 相关信息
   i. 限制 CSV namespace 下只能有 1 个 operator group
4. 判断 CSV 是否支持，operatorgroup 所配置的命名空间(`spec.targetNamespaces`)，当没有定义 `spec.targetNamespaces`、`spec.selector` 时是 global，监听所有命名空间
5. recocile
6. 判断 CSV status Phase 对应处理。
   i. 当状态为 `InstallReady`，处理 `spec.install` 进行部署。
   i. 根据 `spec.install.strategy` 创建 `StrategyDeploymentInstaller` 对象，进行部署

CSV: 部署逻辑
1. 获取 CSV `spec.install.spec` 信息
2. 创建与 api server 认证证书
   i. 循环 `spec.install.spec.deployments` 信息，生成证书，创建 `service`, `secret`, `role`, `rolebinding`, `ClusterRoleBinding` 资源
   i. 将证书 mount 至 deployment 资源
3. 部署自定义资源
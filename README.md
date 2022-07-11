# AzureKubernetesService
How to create azure kubernetes service.

## 事前準備
本手順は、 Azure CLI にて AKS をデプロイする手順となるため、事前に Azure CLI のインストールおよび `az login` で azure にログインしておく

## Azure CLI から AKS をデプロイ（From Docker Registory）

### kubernetes のクラスター作成
Azure portal か Azure CLI をインストールしたクライアント端末から実施   

```powershell
$NAME_PREFIX="azure"
$AKS_CLUSTER_NAME=$NAME_PREFIX + "-aks"
$RG_AKS=$NAME_PREFIX + "-aks-rg"
$LOCATION="japaneast"

// aks用のリソースグループの作成
az group create --name $RG_AKS --location $LOCATION

// kubernetes クラスターの作成
az aks create --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --generate-ssh-key

// Master API 呼び出しのための管理者権限つきクレデンシャルの取得
az aks get-credentials --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --admin
```

### yaml ファイルから k8s のリソース作成
kubectl applyを実行して、クラスター内のリソースを作成および更新　　　

#### yaml ファイル
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: akssample
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: depweb
  namespace: akssample
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - image: httpd
        name: aspnetapp
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: svcweb
  namespace: akssample
spec:
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
---
```

```powershell
PS> kubectl apply -f .\app.yaml
namespace/akssample unchanged
deployment.apps/depweb created
service/svcweb unchanged
```

## Azure CLI から AKS をデプロイ（From コンテナー レジストリ）

### Azure Container registory（ACR）
ACR に任意の Docker イメージをデプロイ

### kubernetes のクラスター作成
Azure portal か Azure CLI をインストールしたクライアント端末から実施   

```powershell
$NAME_PREFIX="azure"
$AKS_CLUSTER_NAME=$NAME_PREFIX + "-aks"
$RG_AKS=$NAME_PREFIX + "-aks-rg"
$LOCATION="japaneast"

// aks用のリソースグループの作成
az group create --name $RG_AKS --location $LOCATION

// kubernetes クラスターの作成
az aks create --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --generate-ssh-key

// Master API 呼び出しのための管理者権限つきクレデンシャルの取得
az aks get-credentials --name $AKS_CLUSTER_NAME --resource-group $RG_AKS --admin
```

### 既存の ACR と既存の AKS クラスターを統合
既存の ACR と既存の AKS クラスターを統合します。
```powershell
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-name>
```

### yaml ファイルから k8s のリソース作成
kubectl applyを実行して、クラスター内のリソースを作成および更新　　　
※ 以下は、ACR へ push 済みの log4j のイメージを使用した yaml

#### yaml ファイル
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: log4j-deployment
  labels:
    app: log4j-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: log4j
  template:
    metadata:
      labels:
        app: log4j
    spec:
      containers:
      - name: log4j
        image: mycontainerregistryyy.azurecr.io/log4j:v1
        ports:
        - containerPort: 8080
```

```powershell
PS> kubectl apply -f 任意.yaml
```

## 便利コマンド
* ネームスペース一覧取得
```powershell
PS> kubectl get namespaces
NAME                STATUS   AGE
akssample           Active   15m
default             Active   39m
gatekeeper-system   Active   30m
kube-node-lease     Active   39m
kube-public         Active   39m
kube-system         Active   39m
```

* 特定名前空間上の pods を取得
```powershell
PS> kubectl get pods --namespace akssample
NAME                      READY   STATUS    RESTARTS   AGE
depweb-6549fb5858-9js4x   1/1     Running   0          2m15s
depweb-6549fb5858-hmcmr   1/1     Running   0          2m15s
depweb-6549fb5858-jh9kt   1/1     Running   0          2m15s
```

* 特定名前空間上にあるすべてのサービスのリストを表示します
```powershell
NAME     TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
svcweb   LoadBalancer   xx.xx.xx.xx   xx.xx.xx.xx   80:30141/TCP   3m43s
```

* 特定名前空間上にある pod の詳細状態の取得
```powershell
kubectl describe pod depweb-6549fb5858-9js4x --namespace akssample
```

### 参考 URL
- [Azure Kubernetes Service から Azure Container Registry の認証を受ける
](https://docs.microsoft.com/ja-jp/azure/aks/cluster-container-registry-integration?tabs=azure-cli)

# AzureKubernetesService
How to create azure kubernetes service.

## 準備
事前に `az login` で azure にログインしておく

## kubernetes のクラスター作成
Azure portal か Azure CLI をインストールしたクライアント端末から実施   

```
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

## yaml ファイルから k8s のリソース作成
kubectl applyを実行して、クラスター内のリソースを作成および更新　　　

```
PS> kubectl apply -f .\app.yaml
namespace/akssample unchanged
deployment.apps/depweb created
service/svcweb unchanged
```

## 便利コマンド
* ネームスペース一覧取得
```
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
```
PS> kubectl get pods --namespace akssample
NAME                      READY   STATUS    RESTARTS   AGE
depweb-6549fb5858-9js4x   1/1     Running   0          2m15s
depweb-6549fb5858-hmcmr   1/1     Running   0          2m15s
depweb-6549fb5858-jh9kt   1/1     Running   0          2m15s
```

* 特定名前空間上にあるすべてのサービスのリストを表示します
```
NAME     TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
svcweb   LoadBalancer   xx.xx.xx.xx   xx.xx.xx.xx   80:30141/TCP   3m43s
```

* 特定名前空間上にある pod の詳細状態の取得
```
kubectl describe pod depweb-6549fb5858-9js4x --namespace akssample
```

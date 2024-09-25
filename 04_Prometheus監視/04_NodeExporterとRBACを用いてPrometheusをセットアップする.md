# ゴール
- 04_NodeExporterとRBACを用いてPrometheusをセットアップする
  - helmを入れ使い方を理解する
  - RBACについてRBACロールをつくる


## kubernetes（kind, kubectl）の動作確認
```
$ sudo snap install helm --classic
helm 3.16.1 from Snapcrafters✪ installed
$ helm version
version.BuildInfo{Version:"v3.16.1", GitCommit:"5a5449dc42be07001fd5771d56429132984ab3ab", GitTreeState:"clean", GoVersion:"go1.22.7"}

$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "prometheus-community" chart repository
Update Complete. ⎈Happy Helming!⎈




$ kind get clusters
No kind clusters found.

$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.31.0) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂


# クラスタの確認
$ kind get clusters
kind

# ノードの確認
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:34107
CoreDNS is running at https://127.0.0.1:34107/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.


$ kubectl config current-context
kind-kind

$ kubectl config use-context kind-kind
Switched to context "kind-kind".

$ kubectl create namespace monitoring
namespace/monitoring created

$ kubectl apply -f prometheus-rbac.yml
serviceaccount/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created


# インストール
$ helm install prometheus prometheus-community/prometheus \
  --namespace monitoring \
  -f prometheus-values.yml
NAME: prometheus
LAST DEPLOYED: Thu Sep 26 02:00:59 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-server.monitoring.svc.cluster.local


Get the Prometheus server URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace monitoring port-forward $POD_NAME 9090


The Prometheus alertmanager can be accessed via port 9093 on the following DNS name from within your cluster:
prometheus-alertmanager.monitoring.svc.cluster.local


Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace monitoring port-forward $POD_NAME 9093
#################################################################################
######   WARNING: Pod Security Policy has been disabled by default since    #####
######            it deprecated after k8s 1.25+. use                        #####
######            (index .Values "prometheus-node-exporter" "rbac"          #####
###### .          "pspEnabled") with (index .Values                         #####
######            "prometheus-node-exporter" "rbac" "pspAnnotations")       #####
######            in case you still need it.                                #####
#################################################################################


The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
prometheus-prometheus-pushgateway.monitoring.svc.cluster.local


Get the PushGateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace monitoring -l "app=prometheus-pushgateway,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace monitoring port-forward $POD_NAME 9091

For more information on running Prometheus, visit:
https://prometheus.io/




# 再度確認
$ kubectl get pods -n monitoring
NAME                                                 READY   STATUS    RESTARTS   AGE
prometheus-alertmanager-0                            1/1     Running   0          30s
prometheus-kube-state-metrics-75b5bb4bf8-9g6fc       1/1     Running   0          30s
prometheus-prometheus-node-exporter-b4mjr            1/1     Running   0          30s
prometheus-prometheus-pushgateway-84557d6c79-7wd4h   1/1     Running   0          30s
prometheus-server-68d7b67c8-d86xh                    1/2     Running   0          30s


# ポートフォワード
$ kubectl port-forward -n monitoring deploy/prometheus-server 9090
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090

この状態でlocalhost:9090にアクセスすると、Prometheusの画面が表示される




# リソース停止
$ kubectl scale deploy prometheus-server --namespace monitoring --replicas=0
kubectl scale statefulset prometheus-alertmanager --namespace monitoring --replicas=0
kubectl scale deploy prometheus-kube-state-metrics --namespace monitoring --replicas=0
kubectl delete daemonset prometheus-prometheus-node-exporter --namespace monitoring
kubectl scale deploy prometheus-prometheus-pushgateway --namespace monitoring --replicas=0

↓daemonsetはscaleできず消すか再度インストールしかない
deployment.apps/prometheus-server scaled
statefulset.apps/prometheus-alertmanager scaled
deployment.apps/prometheus-kube-state-metrics scaled
daemonset.apps "prometheus-prometheus-node-exporter" deleted
deployment.apps/prometheus-prometheus-pushgateway scaled


# すべて削除
helm uninstall prometheus --namespace monitoring
kubectl delete pvc -n monitoring -l app=prometheus
kubectl delete namespace monitoring




# 再度起動
$ kubectl scale deploy prometheus-server --namespace monitoring --replicas=1
kubectl scale statefulset prometheus-alertmanager --namespace monitoring --replicas=1
kubectl scale deploy prometheus-kube-state-metrics --namespace monitoring --replicas=1
helm upgrade prometheus prometheus-community/prometheus --namespace monitoring --set nodeExporter.enabled=true
kubectl scale deploy prometheus-prometheus-pushgateway --namespace monitoring --replicas=1

```



### 参考リンク
[helm（公式）](https://helm.sh/ja/)
[helm（github）](https://github.com/helm/helm)



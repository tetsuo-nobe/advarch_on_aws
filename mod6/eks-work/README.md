# Amazon EKS を触ってみよう
* マニフェストの内容や、実行結果を確認しながら進めてみましょう。
* 1 つの EKS クラスタを複数の受講者で共用します。割当てられた IAM ユーザー 名や EKS クラスタの namespaceを正しく使用しましょう。
* このワークは、**AWS Load Balancer Controller を適用した Amazon EKS クラスターを使用する想定** です。
---

## 環境への接続について

**このワーク環境は、ワーク実施時だけの一時的な環境になります。**

1. 講師が指定する AWS アカウント、IAM ユーザー、パスワードを使用して AWS マネジメントコンソールにサインインします。なお、IAMユーザー名は、EKS クラスタでハンズオン用に指定する namespaceの名前と同じになります。
1. Cloud 9 のページを表示します。IAM ユーザー毎に 1つの Cloud 9 IDE が用意されているので、[**開く**] をクリックします。

---

## Cloud 9 の一時認証情報の無効化
1. Cloud 9 画面の右上にある**歯車アイコン**をクリックします。
1. Preferences タブ の左側で **AWS Settings** をクリックします。
1. 右側の **Credentials** にある **AWS managed temporary credentials** トグルを OFFにします。
  ![codepipeline-demo-img](https://eks.nobelabo.net/images/mod7-cloud9.png)
1. Preferences のタブを閉じます。

---

## 現在の IAM ロールの確認

1. Cloud 9 のターミナルで次のコマンドを実行します。 
   ```
   aws sts get-caller-identity
   ```
1. 出力された Arn に、**my-EC2-EKS-DesclibeCluster-Role** という文字が含まれていることを確認します。

---

   ## kubectl のインストール

1. Cloud 9 のターミナルで次のコマンドを実行します。 (下記のコマンドを 1つずつ実行してください）
   ```
   curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.3/2025-08-03/bin/linux/amd64/kubectl
   ```

   ```
   sudo chmod +x ./kubectl
   ```

   ```
   mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
   ```

   ```
   echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
   ```

   ```
   kubectl version --client
   ```
   
1. 次の例のようなバージョンが表示されることを確認します。 (バージョン番号は異なっていても問題ありません。)
   ```
   Client Version: v1.33.3-eks-59bf375
   Kustomize Version: v5.4.2b
   ```
---

## ハンズオン用の Amazon EKS クラスタへの接続設定

1. Cloud 9 のターミナルで次のコマンドを実行します。 
   ```
   aws eks --region ap-northeast-1 update-kubeconfig --name tgb-cluster
   ```
1. 次のような出力が表示されることを確認します。 (arn の内容は異なっていても問題ありません。)
   ```
   Added new context arn:aws:eks:ap-northeast-1:123412341234:cluster/tgb-cluster to /home/ec2-user/.kube/config
   ```

1. Cloud 9 のターミナルで次のコマンドを実行します。 
   ```
   kubectl get all
   ```
1. 次のような出力が表示されることを確認します。 (下記は抜粋した例です。)
   ```
   NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   
   service/kubernetes   ClusterIP   172.20.0.1   <none>        443/TCP 
   ```

---

## ワーク用リポジトリの取得

1. ワーク用リポジトリをクローンして移動します。
   ```
   git clone https://github.com/tetsuo-nobe/running_containers_on_amazon_eks.git
   cd running_containers_on_amazon_eks/mod6
   ```

---
## ハンズオン用の Web アプリのデプロイ

1. Cloud9 画面左側から 下記の Service のマニフェストのファイルをダブルクリックして開きます。
   ```
   running_containers_on_amazon_eks/mod6/deployment-python-web-ec2.yaml
   ```
1. マニフェストの中で **namespace の値を自分に割当てられた値に変更**してファイルを保存します。

1. Cloud 9 のターミナルで次のコマンドを実行して Deployment を作成します。
   - Deployment とは、Pod を指定した数の分だけ維持し、Pod の柔軟な更新機能を提供する Kubernetes のオブジェクトです。
   ```
   kubectl apply -f deployment-python-web-ec2.yaml
   ```
1. Deployment を表示します。**-n の後には自分の Namespace を指定します。** 2つの Pod が起動したことを確認します。(READY が 2/2 になるまで繰り返し実行して下さい。)
   ```
   kubectl get deploy -n student99
   ```

---

## Service （NodePort タイプ）の作成

1. Cloud9 画面左側から 下記の Service のマニフェストのファイルをダブルクリックして開きます。
   ```
   running_containers_on_amazon_eks/mod6/service/service-nodeport.yaml
   ```
1. マニフェストの中で **namespace の値を自分に割当てられた値に変更**してファイルを保存します。

1. Cloud 9 のターミナルで次のコマンドを実行して Service を作成します。
   ```
   kubectl apply -f service/service-nodeport.yaml 
   ```
1. Service を表示します。**-n の後には自分の Namespace を指定します。出力から PORT(S)の内容を確認します。80: の右横にある数字が Node Port の値になるので、メモしておきます。**
   ```
   kubectl get service -n student99
   ```
1. AWS マネジメントコンソールで EC2 のページを表示して、次の名前のインスタンスの**パブリック IPv4 アドレス**の値をメモしておきます。同じ名前が複数ある場合は、どれでもかまいません。
   ```
   tgb-cluster-tgb-nodes-Node
   ```
1. Webブラウザで新しいタブを開き、次のように URL を指定して **Top Page** という文字を含んだ Web ページが表示されることを確認します。
   ```
   http://<パブリックIPv4アドレス>:<NodePort>
   ```
1. (ご使用のネットワークによりアクセスが制限される場合は、Cloud 9 のターミナルから次のように curl コマンドを実行してアクセスを確認して下さい。)
   ```
   curl <パブリックIPv4アドレス>:<NodePort>
   ```
1. Service を削除します。
   ```
   kubectl delete -f service/service-nodeport.yaml 
   ``` 
1. Service の削除を確認します。**-n の後には自分の Namespace を指定します。**
   ```
   kubectl get service -n student99
   ```

---

## ワークの終了
1. Cloud 9 のターミナルで次のコマンドを実行して Deployment を削除します。
   ```
   kubectl delete -f deployment-python-web-ec2.yaml
   ```
1. Web ブラウザで Cloud 9 IDE のタブを閉じます。

1. AWS マネジメントコンソールで、右上に表示されている IAM ユーザー名をクリックして、メニューからサインアウトをクリックします。
* お疲れ様でした！
* **このワーク環境は、ワーク実施時だけの一時的な環境になります。**  



# Wordpressの冗長化


## 概要
Wordpress deploy by ansible.

<details><summary>AnsibleでWordpressを冗長構成でデプロイするためのプレイブックです。</summary>ロードバランサはHAproxy + KeepaAived、WebサーバはApache、DBはMariaDBです。</summary></details>


## システム構成
* Platform: AWS
* <details><summary>OS: Amazon Linux 2</summary>Amazon Linux 2 LTS Candidate AMI 2017.12.0 (HVM), SSD Volume Type - ami-428aa838</details>
* Web Server: Apache2.4.6-67
* DB Server: MariaDB5.5.56-2
* Load Balancer: HAproxy.5.18-6, KeepaAivedkeepalived-1.3.5-1
* Wordpress4.9


## VPC構成
ansibleサーバをパブリック（インターネット）に配置し、同じプライベートNWにNATインスタンスを配置しています。  
プライベートNWに各サーバを配置し、NATインスタンス経由でインターネットへ接続します。(yumの実行など)  
冗長構成を組むだけならNATインスタンスは不要です。  

通信の流れは、クライアント→wp_vip(lbs01 or lbs02)→wp01 or wp02→db_vip(mariadb1 or mariadb2)

#### ネットワーク構成
| サーバ      | IPアドレス       | 用途             |
|:-----------|:----------------|:----------------|
| ansible    | 172.16.1.10/24  | ansible実行サーバ |
| nat        | 172.16.1.254/24 | ルータ           |
| lbs01      | 172.16.0.21     | ロードバランサ1   |
| lbs02      | 172.16.0.22     | ロードバランサ2   |
| wp01       | 172.16.0.31     | wordpress1      |
| wp02       | 172.16.0.32     | wordpress2      |
| mariadb01  | 172.16.0.41     | mariadb1        |
| mariadb02  | 172.16.0.42     | mariadb2        |
| wp_vip     | 10.0.0.30       | wordpress用VIP  |
| db_vip     | 10.0.0.40       | mariadb用VIP    |      

#### ルートテーブル
| 名前        | サブネット     |
|:-----------|:--------------|
| private    | 172.16.0.0/24 |
| public     | 172.16.1.0/24 |

#### privateのルートテーブル
| 送信先         | ターゲット        |
|:--------------|:----------------|
| 172.16.0.0/16 | local           |
| 0.0.0.0/0     | eni-XXX / i-XXX |
| 10.0.0.30/32  | eni-XXX / i-XXX |
| 10.0.0.40/32  | eni-XXX / i-XXX |

#### privateのルートテーブル
| 送信先         | ターゲット        |
|:--------------|:----------------|
| 172.16.0.0/16 | local           |
| 0.0.0.0/0     | igw-xxx         |

---

## Playbookの実行手順

#### VPC設定
ネットワーク構成に合致したVPCのCIDR、サブネットを設定しておく。

#### EC２インスタンスのデプロイ
1. EC2から仮想マシンを作成。  
   デプロイ時にプライベートIPを付与。  
   インスタンスタイプはt2.nanoでOK
2. 各インスタンス作成後、VPCのルートテーブルを設定

#### ansibleのインストール
ansibleサーバにansibleをインストールします。  

* ansibleのインストール
```
# easy_install pip
# pip install -y ansible
```

* その他パッケージのインストール
```
# yum install git
```

* CentOS7系の場合epelレポジトリが使えるので下記でもOKです。
```
# yum install epel-release
# yum install ansible
# yum install git
```

#### playbookのダウンロード
ansibleサーバでansibleプレイブックをダウンロードします。

```
# mkdir /etc/ansible
# cd /etc/ansible
# git clone https://github.com/alessiareya/ansible.git
```

#### スクリプトの修正
/etc/ansible/wordpress/roles/keepalived/files/以下のスクリプトを修正します。  
IAMでVPC用のロールを作成し、取得したアクセスキー、シークレットキー、リージョンを書き込みます。

* lbs_route_table_update.sh
```
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_DEFAULT_REGION=
```

* mariadb_route_table_update.sh
```
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_DEFAULT_REGION=
```

#### playbook実行

下記コマンドを実行してください。15分ほどでデプロイが完了します。

```
# ansible-playbook -i inventory/inventory.ini deploy.yml
```

## ハマりやすいポイント

#### playbookの実行に失敗した場合
* 実行内容の詳細を確認する
```
# ansible-playbook -i inventory/inventory.ini deploy.yml -vvvvv
```
* 文法チェックは --syntax-check、ドライランは--checkで実行可能です。

#### keepalivedとhaproxyの起動順
* 先にkeepalivedを起動する必要があります。

#### keepalivedのVIP
* 常にマスターサーバのみがVIPを保持します。全てのサーバがVIPを保持している状態（スプリットブレイン）は異常な状態です。  

#### keepalivedの切り替え
* マスターサーバがダウンすると、スレーブサーバ側で/etc/keepalived/lbs_route_table_update.shが実行され、VPCのルートテーブルが更新され、VIPへのルーティングが修正されます。  

#### mariadbの起動順番
* mariadbクラスタのうち、最後にmariadbがシャットダウンされたサーバのsafe_to_bootstrapが1であるサーバから起動する必要があります。  
全てのサーバが0の場合、代表のサーバをvi等で意図的に1に書き換え、起動させます。  
まずマスターサーバを起動した後、スレーブサーバを起動します。

* 状態確認
```
# /var/lib/mysql/grastate.dat
safe_to_bootstrap: 1
```
* 起動方法(マスタサーバ)
```
# service mysql start --wsrep_cluster_address=gcomm://
```

* 起動方法(スレーブサーバ)
```
# service mysql start
```

## 作者
Hatano Yoshihiko  
* [twitter](https://twitter.com/hatanoyoshihiko)

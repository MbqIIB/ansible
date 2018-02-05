# Wordpressの冗長化
* ※2018/2/5現在、作成中です


## 概要
Wordpress deploy by ansible.

<details><summary>AnsibleでWordpressを冗長構成でデプロイするためのプレイブックです。</summary>ロードバランサはHAproxy + KeepaAived、WebサーバはNginx + Php-fpm、DBはMariaDBです。</summary></detail>


## システム構成
* Platform: AWS
* <details><summary>OS: Amazon Linux 2</summary>Amazon Linux 2 LTS Candidate AMI 2017.12.0 (HVM), SSD Volume Type - ami-428aa838</details>
* Web Server: Nginx with php-fpm
* DB Server: MariaDB
* Load Balancer: HAproxy, KeepaAived

| test       | test        | test         |
|:-----------|:------------|:------------ |
| column     | column      | column       |
| column     | column      | column       |
| column     | column      | column       |

## システム概要図
整理中。

- [x] test
- [x] test
- [ ] test

## Playbookの実行手順

### 事前準備
* ansibleのインストール
```
# easy_install pip
# pip install -y ansible
```

* その他パッケージのインストール
```
# yum install git
```


### 環境
整理中。


### VPCの設定
1. プライベートNWのセグメントを作成
2. 「サブネット」から「サブネットの作成」を実行
3.


### EC2のデプロイ
1. EC2から仮想マシンをデプロイ

### playbookのダウンロード
ansibleプレイブックをダウンロードします。

```
# cd /etc/ansible
# git clone https://github.com/farend/redmine-centos-ansible.git
```

### playbook実行

下記コマンドを実行してください。XX分ほどでデプロイが完了します。

```
# ansible-playbook -i inventory/inventory.ini deploy.yml
```

## 作者
Hatano Yoshihiko  
* [twitter](https://twitter.com/hatanoyoshihiko)
* [slack]()
* [github](https://github.com/alessiareya/ansible)

# ansibleの使い方

## 動作環境

### ansibleサーバ
OS: Linux  
Python: 2.6以上

### ansibleクライアント
Python: 2.4以上
※サーバと合わせておいた方が良い

## ansibleのインストール
### yumの場合
```
# yum install epel-release
# yum install ansible
```

### epelが使えない場合
```
# easy_install pip
# pip install ansible
```

---

## ansibleのディレクトリ・ファイル構成
### よく使う構成

```
/etc  
  |-ansible  
    |-ansible.cfg  
    |-hosts            
    |-deploy.yml  
    |-webservers.yml  
    |-dbservers.yml  
  |-roles/  
    |-common/  
      |-tasks/  
        |-main.yml  
        |-include.yml  
      |-handlers/  
        |-main.yml  
      |-templates/  
        |-httpd.conf.j2  #ansible独自のjinjaテンプレート
      |-files/  #コンフィグファイル等を配置
        |-httpd.conf  
        |-configure.sh  
      |-vars/  
        |-main.yml  
        |-include.yml
      |-defaults/  
        |-main.yml  
    |-httpd
    |-mysql  
```   

### inventoryファイルの記述例
yumでインストールするとansible.cfgが生成され、デフォルトのインベントリファイルは/etc/ansible/hostsになる。
```
/etc/ansible/hosts
[web]
192.168.100.1

[ap]
192.168.101.[1:5]

[oracle]
hostname01 ansible_host=192.168.102.1 #ansible_hostはssh接続先IPC
hostname02 ansible_host=192.168.102.2

[mysql]
mysql-[a:f]

[db_servers:children]
oracle
mysql

[all:vars]
ansible_port=22
ansible_user=ansible #ssh接続先ユーザ
ansible_ssh_pass=password #ssh接続先パスワード
ansible_become=true #ssh接続先でsudoするかどうか。デフォルトはfalse
ansible_become_pass=password #ssh接続先sudoパスワード
```

---

### roleの構造
ロール名を指定してプレイブックを実行するとき、tasksやvars配下のmain.ymlのみが読み込まれる点に注意。同じ階層の他ymlを読み込み時は、main.yml内にimport_tasks:やinclude_tasks:で読み込むymlファイルを指定すること。

```
/roles
  |-common
    |-files
      |-common.txt  
    |-handlers
      |-main.yml  
    |-tasks
      |-main.yml
      |-install.yml
      |-configure.yml  
    |-templates
      |-common.j2
    |-vars
      |-main.yml
      |-include.yml
  |-httpd
  |-mariadb  
  |-wordpress
```  

## roleの実行方法
### プレイブック例
```
/etc/ansbile/roles/deploy.yml
- name: deploy test
  hosts: databases
  become: true
  roles:
    - { role: common, tags: common }
    - { role: httpd, tags: httpd }
    - { role: mariadb, tags: mariadb }
    - { role: wordpress, tags: wordpress ß}
```

### role実行例
tagsで指定したroleのみを実行するときは-tで指定する。

```
$ ansible-playbook deploy.yml -i hosts --ask-become-pass --ask-pass -t common,httpd
```

### ホスト指定実行例
インベントリファイルのホストを指定するときは-l（エル）で指定する
/etc/ansible/hosts
[mysql]
192.168.100.1

```
$ ansible-playbook deploy.yml -i hosts --ask-become-pass --ask-pass -l mysql
```

---

## 主なモジュールの使い方

###
```

```

###
```

```

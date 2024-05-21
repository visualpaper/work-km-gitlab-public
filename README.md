# work-km-gitlab-public

```
自分の レンタルサーバで試したところ OOM 
https://qiita.com/k_nakayama/items/9f083a4700915d02104a

※ 4GB も必要らしく 2GB しかないので落ちていた。

自分の AWS アカウントで 4GB マシンを用意し実験

sudo yum update -y
sudo yum install -y docker
sudo docker service start

sudo docker run --detach \
  --hostname gitlab.example.com \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://{public ip}:8888'; gitlab_rails['gitlab_shell_ssh_port'] = 2222; nginx['listen_port'] = 80; postgresql['shared_buffers'] = '1024MB'; gitlab_rails['db_prepared_statements'] = false" \
  --publish 8888:80 --publish 2222:22 \
  --name gitlab \
  --restart always \
  gitlab/gitlab-ce:latest
```

↓

rpm でもできそうだったので検証
https://dev.classmethod.jp/articles/amazon-linux-2023-gitlab-ce/
※ 公式 (https://about.gitlab.com/ja-jp/install/#centos-7) とほぼ同じなので公式の通りやってみる
※ 4v CPU/16GiB Memory でやってみる

sudo yum update -y
sudo yum install -y curl policycoreutils-python-utils.noarch openssh-server perl
※ policycoreutils-python は RHEL ではダメっぽいので↑

sudo systemctl enable sshd
sudo systemctl start sshd
※ ssh アクセスを開く

sudo yum -y install firewalld
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo systemctl is-enabled firewalld
※ firewalld がなかったので入れて自動起動するようにしておく。

sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
※ http/https を開く

メール通知用に以下コマンドもあったが一旦無視 (通知する必要があるなら入れれば OK
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
※ ce のほう (無料のほう) をインストール

sudo EXTERNAL_URL="http://13.231.111.37" yum install -y gitlab-ce
※ 最新版で不具合があるらしい (https://forum.gitlab.com/t/update-gitlab-17-failed-gitlab-kas-runtimeerror-execution-of-the-command-opt-gitlab-embedded-bin-gitlab-kas-version-failed/104696) ので、
   gitlab-kas を enabled = false にし再起動
   > gitlab-ctl reconfigure
※ 年次で毎月 5 月に更新されるらしい (https://qiita.com/Hachiyou-Renge/items/6479a003950481ec70de) ので、
   その場合は上記のようにバージョン指定して更新すれば OK

https 化する場合以下の手順で実施可能なはず。

https://docs.gitlab.com/omnibus/settings/ssl/index.html#configure-https-manually
※ デフォだと letsencrypt で世に公開されているドメイン名であれば OK となっているらしい。
   ので、https 化した後に DNS にレコード追加すればたぶんできるはず。


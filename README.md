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

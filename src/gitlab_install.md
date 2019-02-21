## gitlab安装

---

- 安装docker
```shell
yum install docker -y
```
- 安装docker-compose [安装地址](https://docs.docker.com/compose/install/)
- docker-compose配置文件
```shell
mkdir gitlab && cd gitlab
vim docker-compose.yml
docker-compose up -d
```
docker-compose.yml内容如下：
```yml
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'xx.xx.xx.xx'
  privileged: true
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://xx.xx.xx.xx:9090'
      gitlab_rails['gitlab_shell_ssh_port'] = 10022
  ports:
    - '9090:9090'
    - '10022:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
```

# 1.安装使用

使用docker安装

- `docker pull rabbitmq`

- 开启容器并设置用户名密码，注意一定要设置端口方便后面网页管理

  ```sh
  docker run -d --hostname my-rabbit --name some-rabbit \
  -p 15672:15672 -p 5672:5672 \
  -e RABBITMQ_DEFAULT_USER=user -e RABBITMQ_DEFAULT_PASS=password \
  rabbitmq
  ```

  

- 开启插件以网页管理

  - 查询该容器id：`docker ps`
  - 进入容器：`docker exec -it 镜像ID /bin/bash`
  - 开启：`rabbitmq-plugins enable rabbitmq_management`
  - 退出容器：`exit`

远程访问：http://x.x.x.x:15762，默认用户名密码都为`guest`


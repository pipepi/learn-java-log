# docker-compose
```shell
docker-compose -f docker-compose-env.yml up -d # 执行服务创建脚本
docker-compose -f docker-compose-env.yml stop # 停止服务
docker-compose -f docker-compose-env.yml ps # 查看服务状态
sudo cp mall.sql /mydata/ # 拷贝文件
docker exec -it mysql /bin/bash # 进入mysql容器
exit  # or press ctl+d 退出容器（不会关闭容器）
docker logs mall-auth # 查看某容器的日志（可查看docker-compse ps 列出的失败容器的失败原因）
```

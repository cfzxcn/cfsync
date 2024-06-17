https://github.com/minio/minio/tree/master/docs/orchestration/docker-compose
模拟4节点，每节点有2设备（磁盘），下图为部署后效果
![[Pasted image 20240530175737.png]]

![[Pasted image 20240418212731.png|875]]
![[Pasted image 20240530172847.png]]
![[Pasted image 20240530173125.png]]
data1-1：是宿主机目录；data1：是容器中的目录；
4个services模拟了4个主机；会启动4个容器
![[Pasted image 20240530173516.png]]
![[Pasted image 20240530173605.png]]
![[Pasted image 20240530173647.png]]
![[Pasted image 20240530173730.png]]
![[Pasted image 20240530173910.png]]
![[Pasted image 20240530173929.png]]
docker compose up [-d]


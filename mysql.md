#### mysql处理
1.初始化 mysql 数据库密码（刚开始没有密码）
```bash
docker exec -it mysql bash
# mysql
mysql > show databases;
      > set password = password('****');
```

------
问题1：
root@db:/# mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
解决：
root@db:/# mysql -u root -p
Enter password:密码和docker-compose.yml相同

2.redmine远程不允许访问
```bash
mysql > use mysql;
      > update user set host='%' where use='root';
```

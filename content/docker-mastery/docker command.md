- 刪除沒在用的 Image (這會刪掉 MySQL, Mongo, Elasticsearch 等)
```
docker image prune -a
```

- 停止所有背景容器

```
docker stop $(docker ps -q)
```
- 清理磁碟與記憶體快取
```
docker system prune -a -f
```
- 強制停止並刪除所有容器
```
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```
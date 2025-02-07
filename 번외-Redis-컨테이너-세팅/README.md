

```
docker rm -f docker_redis
docker rmi my-redis:latest
docker build -t my-redis .
docker run -d -p 6380:6379 -v C:\workspace\lab-redis\volume\data:/data --name docker_redis my-redis
docker ps
docker exec -it docker_redis /bin/bash
redis-cli
```
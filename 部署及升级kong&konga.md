## 部署kong&konga

### step1. 部署kong:2.0.3

```
docker network create kong-net

docker run -d --name kong-database --network=kong-net -p 5432:5432 -e "POSTGRES_USER=kong" -e "POSTGRES_DB=kong" -e "POSTGRES_PASSWORD=kong" postgres:11.0

docker run --rm --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kong" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" kong:2.0.3 kong migrations bootstrap

docker run -d --name kong1 --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kong" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" -e "KONG_PROXY_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" -e "KONG_PROXY_LISTEN=0.0.0.0:8000, 0.0.0.0:8443 ssl" -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" -p 8000:8000 -p 8443:8443 -p 127.0.0.1:8001:8001 -p 127.0.0.1:8444:8444 kong:2.0.3
```

### step2. 部署konga

```
docker run -d -p 1337:1337 --network kong-net -e "DB_ADAPTER=postgres" -e "DB_HOST=kong-database" -e "DB_USER=kong" -e "DB_PASSWORD=kong" --name konga pantsel/konga
```

```
查询kong容器ip地址
docker exec -it kong1 ip addr
```

## 升级kong

### step3. 开启kong蓝绿迁移模式

```
docker run --rm --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kong" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" kong:2.2.0 kong migrations up
```

### step4. 部署新版本kong

```
docker run -d --name kong2 --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kong" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" -e "KONG_PROXY_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" -e "KONG_PROXY_LISTEN=0.0.0.0:9000, 0.0.0.0:9443 ssl" -e "KONG_ADMIN_LISTEN=0.0.0.0:9001, 0.0.0.0:9444 ssl" -p 9000:9000 -p 9443:9443 -p 127.0.0.1:9001:9001 -p 127.0.0.1:9444:9444 kong:2.2.0
```

### step5. 关闭蓝绿迁移模式

```
docker run --rm --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_PG_USER=kong" -e "KONG_PG_PASSWORD=kong" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" kong:2.2.0 kong migrations finish
```

### step6. 修改konga管理api地址

```
docker exec -it kong2 ip addr
```
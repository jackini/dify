## 创建网络
docker network create my_network

## 启动postgres容器
docker run --name postgres15 -e POSTGRES_PASSWORD=123456 -d --network=my_network -p 5432:5432 postgres:15

## 启动redis容器
docker run -d 
--name redis6 
--network=my_network 
-p 6379:6379 
-v D:\\tools\\redis-5.0.14.1\\redis.linux.conf:/etc/redis/redis.conf  
redis:6.2.17 redis-server /etc/redis/redis.conf --appendonly yes

## 启动dify-web容器
docker run --name dify-web -p 3000:3000 -e CONSOLE_API_URL=http://127.0.0.1:5001 -e APP_API_URL=http://127.0.0.1:5001 langgenius/dify-web:latest

## 启动dify-plugin-daemon容器
docker run -d 
--name dify-plugin-daemon 
--network=my_network 
-p 5002:5002 
-e DB_USERNAME=postgres 
-e DB_PASSWORD=123456 
-e DB_HOST=postgres15 
-e DB_PORT=5432 
-e DB_DATABASE=dify_plugin  
-e REDIS_HOST=redis6  
-e REDIS_PORT=6379 
-e REDIS_USERNAME= 
-e REDIS_PASSWORD=yEJFyNvLIBtDvE92q8R4 
-e REDIS_USE_SSL=false 
-e REDIS_DB=5 
-e SERVER_PORT=5002 
-e SERVER_KEY=lYkiYYT6owG+71oLerGzA7GXCgOT++6ovaezWAjpCjf+Sjc3ZtU+qUEi 
-e MAX_PLUGIN_PACKAGE_SIZE=15728640 
-e PPROF_ENABLED=false 
-e DIFY_INNER_API_URL=http://localhost:5001 
-e DIFY_INNER_API_KEY=QaHbTe77CtuXmsfyhR7+vRjI/+XbV1AaFy691iy+kGDv2Jvy0/eAh8Y1 
-e PLUGIN_REMOTE_INSTALLING_HOST=localhost 
-e PLUGIN_REMOTE_INSTALLING_PORT=5003 
-e PLUGIN_WORKING_PATH=/app/storage/cwd 
-e FORCE_VERIFYING_SIGNATURE=true 
langgenius/dify-plugin-daemon:0.0.6-local

poetry run python -m flask db upgrade

poetry run python -m flask run --host 0.0.0.0 --port=5001 --debug

poetry run python -m celery -A app.celery worker -P gevent -c 1 --loglevel INFO -Q dataset,generation,mail,ops_trace,app_deletion
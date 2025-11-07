# UdyogSet - Docker Runbook

This stack is managed from this `config-repo` folder. All commands below assume your shell is in this directory.

## Prerequisites
- Docker Desktop (with Docker Compose V2)
- Ports used: 5432, 8761, 8888, 8764, 9411, 9412, 8081, 8001, 8002, 5672, 15672, 6379

## Start the full stack (build + up)
```bash
# Windows PowerShell or any shell
cd ./config-repo

docker compose up -d --build
```

## Stop the stack (keep data volume)
```bash
docker compose down
```

## View logs
```bash
# All services (follow)
docker compose logs -f

# Single service (e.g., api-gateway)
docker compose logs -f api-gateway
```

## Rebuild and restart a single service
```bash
# Example: api-gateway
docker compose up -d --no-deps --build api-gateway
```

## Scale a stateless service
```bash
# Example: run 2 instances of user-service
docker compose up -d --scale user-service=2
```

## Health check URLs
- Eureka Discovery: http://localhost:8761
- Config Server: http://localhost:8888/actuator/health
- API Gateway: http://localhost:8764/actuator/health
- Zipkin UI: http://localhost:9411
- Admin Service: http://localhost:8081/actuator/health
- Auth Service: http://localhost:8001/actuator/health
- User Service: http://localhost:8002/actuator/health

## RabbitMQ
- RabbitMQ Broker: `localhost:5672`
- RabbitMQ Management UI: http://localhost:15672 (username: admin, password: admin)

## Redis
- Redis Server: `localhost:6379`

### Quick start (CLI)
```bash
# Open a Redis CLI shell inside the container
docker compose exec -it redis redis-cli

# Example commands
SET foo bar
GET foo
PING
```

### Quick start (exchanges, queues, produce, consume)
```bash
# Access RabbitMQ Management UI
# Open browser: http://localhost:15672
# Login: admin / admin

# Or use RabbitMQ CLI (from inside the rabbitmq container)
docker compose exec rabbitmq rabbitmqctl list_exchanges
docker compose exec rabbitmq rabbitmqctl list_queues
docker compose exec rabbitmq rabbitmqctl list_bindings

# Publish a test message (using rabbitmqadmin or Management UI)
# Or use Spring AMQP RabbitTemplate in your services
```

## Clean up images and volumes
```bash
# Stop and remove containers
docker compose down

# Also remove the named data volume (postgres)
docker compose down -v

# Prune dangling images/layers (use with care)
docker system prune -f
```

## If you previously ran from repo root
If you earlier started the stack from the repository root, container names may conflict. Run the following once to remove old containers by name:
```powershell
# PowerShell
$names = @('postgresql','discovery-server','config-server','admin-server','api-gateway','auth-service','user-service','zipkin','zipkin-server');
foreach ($n in $names) { docker rm -f $n 2>$null }
```
Then start again from this folder:
```bash
docker compose up -d --build
```

## Notes
- Compose file paths in this folder reference service contexts using `..` to reach service directories.
- Services use profile `docker` via `SPRING_PROFILES_ACTIVE=docker`.
- The API Gateway routes via Eureka to internal service IDs and container DNS names.



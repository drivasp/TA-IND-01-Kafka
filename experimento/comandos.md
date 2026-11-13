# Experimento: tolerancia a fallos y CAP/PACELC en Apache Kafka

**Concepto de la Unidad 1:** tolerancia a fallos (replicación e ISR) y el compromiso entre consistencia y disponibilidad (CAP), y entre latencia y consistencia (PACELC).

**Entorno:** Windows con PowerShell y Docker Desktop. El clúster tiene 3 brokers KRaft (`docker-compose.yml`).

> Al inicio de cada captura ejecute `whoami; hostname; Get-Date` para que se vean su usuario, su máquina y la fecha y hora.
> Cada salida se guarda en `resultados/` con `Tee-Object`.

---

## Paso 0. Levantar el clúster

```powershell
cd experimento
whoami; hostname; Get-Date
docker compose up -d
docker ps --format "table {{.Names}}\t{{.Status}}"
```

📸 **Captura 1:** los tres contenedores corriendo.

## Paso 1. Ver el quórum de controladores (KRaft)

```powershell
docker exec kafka1 /opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-server kafka1:9092 describe --status | Tee-Object resultados/01_quorum.txt
```

## Paso 2. Crear un tópico replicado (factor de réplica 3, min.insync.replicas 2)

```powershell
docker exec kafka1 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka1:9092 --create --topic pedidos --partitions 3 --replication-factor 3 --config min.insync.replicas=2

docker exec kafka1 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka1:9092 --describe --topic pedidos | Tee-Object resultados/02_describe_inicial.txt
```

📸 **Captura 2:** el `Leader`, las `Replicas` y el `Isr` de cada partición.

## Paso 3. Enviar y recibir mensajes

```powershell
"pedido-1","pedido-2","pedido-3" | docker exec -i kafka1 /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server kafka1:9092 --topic pedidos --producer-property acks=all

docker exec kafka2 /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka2:9092 --topic pedidos --from-beginning --timeout-ms 10000 | Tee-Object resultados/03_consumo_inicial.txt
```

📸 **Captura 3:** el mensaje se produce en un broker y se consume desde otro (transparencia de ubicación y de acceso).

## Paso 4. Tumbar un broker (fallo de 1 de 3 nodos)

```powershell
whoami; hostname; Get-Date
docker stop kafka1
docker exec kafka2 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka2:9092 --describe --topic pedidos | Tee-Object resultados/04_describe_sin_kafka1.txt

"pedido-4" | docker exec -i kafka2 /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server kafka2:9092 --topic pedidos --producer-property acks=all
docker exec kafka3 /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka3:9092 --topic pedidos --from-beginning --timeout-ms 10000 | Tee-Object resultados/04_consumo_sin_kafka1.txt
```

📸 **Captura 4:** el nuevo líder y el ISR reducido. El sistema sigue aceptando escrituras.

## Paso 5. Tumbar un segundo broker (solo queda 1 réplica y min.insync = 2)

```powershell
docker stop kafka2
"pedido-5" | docker exec -i kafka3 /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server kafka3:9092 --topic pedidos --producer-property acks=all --producer-property delivery.timeout.ms=15000 --producer-property request.timeout.ms=10000 2>&1 | Tee-Object resultados/05_escritura_rechazada.txt
```

📸 **Captura 5:** el error que devuelve el productor. Anote el texto exacto.

> Anote qué se observa aquí y relaciónelo con CAP: ¿qué eligió Kafka, consistencia o disponibilidad?

## Paso 6. Recuperación

```powershell
docker start kafka1 kafka2
Start-Sleep -Seconds 20
docker exec kafka3 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka3:9092 --describe --topic pedidos | Tee-Object resultados/06_describe_recuperado.txt
```

📸 **Captura 6:** el ISR vuelve a tener 3 réplicas.

## Paso 7 (PACELC). Latencia con `acks=1` frente a `acks=all`, sin fallos

```powershell
docker exec kafka1 /opt/kafka/bin/kafka-producer-perf-test.sh --topic pedidos --num-records 50000 --record-size 200 --throughput -1 --producer-props bootstrap.servers=kafka1:9092 acks=1 | Tee-Object resultados/07_perf_acks1.txt

docker exec kafka1 /opt/kafka/bin/kafka-producer-perf-test.sh --topic pedidos --num-records 50000 --record-size 200 --throughput -1 --producer-props bootstrap.servers=kafka1:9092 acks=all | Tee-Object resultados/07_perf_acksall.txt
```

Repita cada medición 3 veces y anote los valores:

| Corrida | acks=1: latencia media (ms) | acks=all: latencia media (ms) |
|---|---|---|
| 1 | | |
| 2 | | |
| 3 | | |

> Interprete los resultados: ¿cuánta latencia cuesta la consistencia cuando no hay partición (la parte "ELC" de PACELC)?

## Apagar el clúster

```powershell
docker compose down -v
```

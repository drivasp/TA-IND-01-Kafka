# Experimento: tolerancia a fallos y CAP/PACELC en Apache Kafka

**Concepto de la Unidad 1:** tolerancia a fallos (replicación e ISR) y el compromiso entre consistencia y disponibilidad (CAP), y entre latencia y consistencia (PACELC).

**Entorno:** Windows con PowerShell y Docker Desktop. El clúster tiene 3 brokers KRaft (`docker-compose.yml`).

Este archivo recoge los comandos que se ejecutaron de verdad (jueves 12 y sábado 14 de noviembre de 2026), tal como aparecen en las capturas de `informe/capturas/sistema/`.

> Al inicio de cada captura se ejecutó `whoami; hostname; Get-Date` para que se vean el usuario, la máquina y la fecha y hora.
> Las salidas se guardaron en `resultados/` con nombres fijos. Desde el sábado se usó
> `... 2>&1 | Tee-Object -Variable r; $r | Out-File -Encoding utf8 resultados\<archivo>.txt`
> para guardar también los mensajes de error y escribir el archivo en UTF-8.

---

## Paso 0. Levantar el clúster (jueves 12/11)

```powershell
cd C:\Users\Deyver\Documents\TA-IND-01-Kafka\experimento
whoami; hostname; Get-Date
docker compose up -d
docker ps --format "table {{.Names}}\t{{.Status}}"
```

📸 `captura1_cluster_levantado.jpg`: los tres contenedores corriendo.

## Paso 1. Ver el quórum de controladores KRaft (jueves 12/11)

```powershell
docker exec kafka1 /opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-server kafka1:9092 describe --status | Tee-Object resultados/01_quorum.txt
```

📸 `captura2_quorum_kraft.jpg` → `resultados/01_quorum.txt` (líder del quórum: nodo 1; votantes: 1, 2 y 3).

## Paso 2. Crear el tópico replicado (sábado 14/11, 17:13)

```powershell
cd C:\Users\Deyver\Documents\TA-IND-01-Kafka\experimento
docker exec kafka1 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka1:9092 --create --topic pedidos --partitions 3 --replication-factor 3 --config min.insync.replicas=2

whoami; hostname; Get-Date
docker exec kafka1 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka1:9092 --describe --topic pedidos 2>&1 | Tee-Object -Variable r; $r | Out-File -Encoding utf8 resultados\02_describe_inicial.txt
```

📸 `capturaA_topico_estado_inicial.png` → `resultados/02_describe_inicial.txt` (líderes: P0 en el broker 3, P1 en el 1, P2 en el 2; ISR completo).

## Paso 3. Enviar los primeros mensajes

```powershell
"pedido-1","pedido-2","pedido-3" | docker exec -i kafka1 /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server kafka1:9092 --topic pedidos --producer-property acks=all
```

No se guardó un archivo de salida para este paso; los tres mensajes aparecen después al consumir en el Paso 4.

## Paso 4. Prueba 1: tumbar el líder de la partición 0 (`kafka3`) (sábado 14/11, 17:19)

Estos comandos se ejecutaron en una ventana de PowerShell como administrador, que abrió en `C:\Windows\system32`.

```powershell
whoami; hostname; Get-Date
docker stop kafka3
Start-Sleep -Seconds 10
docker exec kafka1 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka1:9092 --describe --topic pedidos 2>&1 | Tee-Object -Variable r; $r | Out-File -Encoding utf8 resultados\04_describe_sin_lider.txt
"pedido-4" | docker exec -i kafka1 /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server kafka1:9092 --topic pedidos --producer-property acks=all
docker exec kafka1 /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092 --topic pedidos --from-beginning --timeout-ms 10000
```

📸 `capturaB_caida_lider_kafka3.png`: el broker 1 pasa a ser líder de la partición 0, el ISR baja a 2 réplicas, `pedido-4` se acepta y se consumen 4 mensajes.

> ⚠️ `Out-File` falló con `DirectoryNotFoundException` porque en `C:\Windows\system32` no existe la carpeta `resultados`. Por eso `resultados/04_describe_sin_lider.txt` es una transcripción de la captura B, no la salida original. Para repetirlo, haga primero `cd` a la carpeta `experimento`.

## Paso 5. Prueba 2: tumbar un segundo broker (`kafka2`) (sábado 14/11, 17:20)

```powershell
whoami; hostname; Get-Date
docker stop kafka2
Start-Sleep -Seconds 10
"pedido-5" | docker exec -i kafka1 /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server kafka1:9092 --topic pedidos --producer-property acks=all --producer-property delivery.timeout.ms=15000 --producer-property request.timeout.ms=10000 2>&1 | Tee-Object -Variable r; $r | Out-File -Encoding utf8 resultados\05_escritura_rechazada.txt
```

📸 `capturaC_escritura_rechazada_not_enough_replicas.png`: el productor agota sus reintentos con `NOT_ENOUGH_REPLICAS` y termina con `NotEnoughReplicasException`.

> ⚠️ Mismo problema de ruta que en el Paso 4: `resultados/05_escritura_rechazada.txt` es una transcripción de la captura C.

## Paso 6. Recuperación (sábado 14/11)

```powershell
docker start kafka3 kafka2
```

📸 `capturaD_reinicio_brokers.png`.

Después, desde la carpeta `experimento`, se volvió a describir el tópico:

```powershell
cd C:\Users\Deyver\Documents\TA-IND-01-Kafka\experimento
docker exec kafka1 /opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka1:9092 --describe --topic pedidos 2>&1 | Tee-Object -Variable r; $r | Out-File -Encoding utf8 resultados\06_describe_recuperado.txt
```

→ `resultados/06_describe_recuperado.txt`: ISR de nuevo con 3 réplicas y la partición 0 otra vez liderada por el broker 3.

## Paso 7 (PACELC). Latencia con `acks=1` frente a `acks=all` — NO EJECUTADO

Queda como trabajo futuro (ver Conclusiones del informe).

```powershell
docker exec kafka1 /opt/kafka/bin/kafka-producer-perf-test.sh --topic pedidos --num-records 50000 --record-size 200 --throughput -1 --producer-props bootstrap.servers=kafka1:9092 acks=1 2>&1 | Tee-Object -Variable r; $r | Out-File -Encoding utf8 resultados\07_perf_acks1.txt

docker exec kafka1 /opt/kafka/bin/kafka-producer-perf-test.sh --topic pedidos --num-records 50000 --record-size 200 --throughput -1 --producer-props bootstrap.servers=kafka1:9092 acks=all 2>&1 | Tee-Object -Variable r; $r | Out-File -Encoding utf8 resultados\07_perf_acksall.txt
```

## Apagar el clúster

```powershell
docker compose down -v
```

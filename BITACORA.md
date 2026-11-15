# Bitácora de trabajo — TA-IND-01

Hay una entrada por cada día trabajado. Las fechas coinciden con los commits del repositorio.

---

## 2026-11-12 (jueves)

- **Qué hice:**
elegí Kafka frente a RabbitMQ;
subí la carátula al SGA;
creé y cloné el repositorio;
instalé Docker y WSL;
levanté el clúster de 3 brokers;
verifiqué el quórum KRaft (líder: nodo 1; votantes: 1, 2 y 3).

- **Problema  encontrado:**
Docker dio el error “Virtual Machine Platform is not enabled”;
se cortó la luz en mi casa.


- **Cómo lo resolví:**
entré con correo y contraseña;
ejecuté wsl --install como administrador;
dejé el resto para el viernes.

- **Tiempo dedicado:**
Aproximadamente 1 hora

---

## 2026-11-14 (sábado)

- **Qué hice:**
creé el tópico pedidos (3 particiones, factor de réplica 3, min.insync.replicas=2) y guardé su estado inicial (17:13, captura A);
Prueba 1: detuve kafka3, que era líder de la partición 0; el broker 1 asumió el liderazgo, el ISR bajó a 2 réplicas y pedido-4 se aceptó; consumí los 4 mensajes (17:19, captura B);
Prueba 2: detuve también kafka2; la escritura de pedido-5 fue rechazada con NOT_ENOUGH_REPLICAS (17:20, captura C);
reinicié kafka3 y kafka2 y comprobé que el ISR volvió a 3 réplicas y que la partición 0 regresó al broker 3 (captura D y 06_describe_recuperado.txt);
redacté la Sección 6 (Experimentación propia) con el entorno, las dos pruebas y la recuperación;
revisé que el informe coincida con las capturas A, B y C y con experimento/resultados/06_describe_recuperado.txt (la partición 0 volvió a tener como líder al broker 3);
agregué la referencia Wang2015 a referencias.bib y la cité en la Prueba 2;
compilé el PDF y verifiqué que la cita salga como [1] y no como [?];
por la noche, con Claude, integré en el informe mis puntos técnicos para las Secciones 2, 3, 4, 5, 8, 9, 10 y 11, añadí las referencias oficiales de RabbitMQ, la documentación de diseño de Kafka y el post-mortem de PagerDuty, y actualicé comandos.md con los comandos que realmente ejecuté.

- **Problema encontrado:**
al principio guardé los cambios en una copia sin Git (carpeta "Tarea individual de Gleisston1") y no en el repositorio;
mi equipo no tiene biber, así que la bibliografía no compilaba localmente;
06_describe_recuperado.txt aparecía modificado solo por los saltos de línea de Windows;
en las Pruebas 1 y 2 abrí PowerShell como administrador en C:\Windows\system32, así que Out-File no encontró la carpeta resultados y no se guardaron 04 y 05.

- **Cómo lo resolví:**
pasé la Sección 6 y la referencia al repositorio real (Documentos\TA-IND-01-Kafka) y borré la copia;
compilé el informe en otro entorno con biber;
descarté el cambio de salto de línea con git restore;
transcribí las salidas de 04 y 05 desde las capturas B y C y lo indiqué en cada archivo.

- **Tiempo dedicado:**
Aproximadamente 3 horas y media (primera captura a las 17:13; últimos cambios pasadas las 20:30)

---

## 2026-11-15 (domingo)

- **Qué hice:**
- **Problema encontrado:**
- **Cómo lo resolví:**
- **Tiempo dedicado:**

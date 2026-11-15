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
por la noche, con Claude, integré en el informe mis puntos técnicos para las Secciones 2, 3, 4, 5, 8, 9, 10 y 11, añadí las referencias oficiales de RabbitMQ, la documentación de diseño de Kafka y el post-mortem de PagerDuty, y actualicé comandos.md con los comandos que realmente ejecuté;
escribí la Sección 7 con dos ejemplos de mi entorno (SGA de la UTEQ y WhatsApp);
volví a tomar las tres capturas de lectura (Raptis, Maharjan y Kamiński) marcando la frase completa;
con Claude revisé en Crossref los DOIs de las referencias y corregí el de Wang2015, que llevaba a otro artículo;
completé la Introducción (quién usa Kafka, a qué escala y por qué lo elegí) y agregué al informe las capturas 1, 2 y 4 del clúster;
subí los cambios al repositorio en varios commits separados por tema.

- **Problema encontrado:**
al principio guardé los cambios en una copia sin Git (carpeta "Tarea individual de Gleisston1") y no en el repositorio;
mi equipo no tiene biber, así que la bibliografía no compilaba localmente;
06_describe_recuperado.txt aparecía modificado solo por los saltos de línea de Windows;
en las Pruebas 1 y 2 abrí PowerShell como administrador en C:\Windows\system32, así que Out-File no encontró la carpeta resultados y no se guardaron 04 y 05;
el DOI que tenía para Wang2015 abría otro artículo de la misma revista.

- **Cómo lo resolví:**
pasé la Sección 6 y la referencia al repositorio real (Documentos\TA-IND-01-Kafka) y borré la copia;
compilé el informe en otro entorno con biber;
descarté el cambio de salto de línea con git restore;
transcribí las salidas de 04 y 05 desde las capturas B y C y lo indiqué en cada archivo;
busqué el artículo en Crossref y puse el DOI correcto (10.14778/2824032.2824063).

- **Tiempo dedicado:**
Aproximadamente 4 horas (primera captura a las 17:13; últimos cambios pasadas las 21:00)

---

## 2026-11-15 (domingo)

- **Qué hice:**
pegué en el README.md el enlace del video de sustentación (https://youtu.be/o0u45hrLPao), que grabé el sábado por la noche, y lo subí al repositorio;
revisé que el repositorio quedara sin cambios pendientes y que el PDF compilado fuera la última versión.

- **Problema encontrado:**
el commit del enlace se llamaba "Agrego el enlace del video y la bitácora del domingo", pero solo incluía el README.md; la bitácora no tenía la entrada de este día.

- **Cómo lo resolví:**
agregué esta entrada a BITACORA.md y la subí en un commit aparte.

- **Tiempo dedicado:**
Aproximadamente 20 minutos

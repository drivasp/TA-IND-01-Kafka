# TA-IND-01 — Análisis crítico de Apache Kafka frente a RabbitMQ

**Universidad Técnica Estatal de Quevedo** · Facultad de Ciencias de la Computación · Carrera de Software
**Asignatura:** Aplicaciones Distribuidas (ISR-701) · Séptimo nivel, Paralelo B
**Estudiante:** Rivas Piloso Dayver Yahir
**Docente:** Gleiston Cicerón Guerrero Ulloa, PhD
**Periodo:** SPA 2026-2027 · Semana III

## Video de sustentación

🎥 **Enlace:** _PEGAR AQUÍ el enlace de YouTube (no listado) o Google Drive (acceso abierto)_

## Estructura del repositorio

```
.
├── README.md               # Este archivo
├── BITACORA.md             # Registro fechado de cada sesión de trabajo
├── informe/
│   ├── main.tex            # Archivo principal del informe
│   ├── referencias.bib     # Bibliografía (biblatex, estilo IEEE)
│   ├── main.pdf            # PDF compilado
│   ├── figuras/            # Diagrama de arquitectura (draw.io o TikZ)
│   ├── capturas/sistema/   # Capturas de la ejecución de Kafka
│   ├── capturas/fuentes/   # Capturas de los artículos subrayados (Anexo B)
│   └── anexos/             # Foto del esquema a mano (Anexo A)
└── experimento/
    ├── docker-compose.yml  # Clúster Kafka de 3 brokers (KRaft)
    ├── comandos.md         # Comandos ejecutados, paso a paso
    └── resultados/         # Salidas obtenidas (.txt)
```

## Compilación del informe

| Elemento | Valor |
|---|---|
| Compilador | `pdflatex` (TeX Live 2023 o superior, o MiKTeX) |
| Archivo principal | `informe/main.tex` |
| Gestor de bibliografía | `biber` |

Orden de comandos (desde la carpeta `informe/`):

```bash
cd informe
pdflatex main.tex
biber main
pdflatex main.tex
pdflatex main.tex
```

### Dependencias LaTeX

`babel` (spanish), `biblatex` con `biblatex-ieee`, `biber`, `graphicx`, `hyperref`, `booktabs`, `tabularx`, `xcolor`, `geometry`, `float`, `caption`, `csquotes`, `tikz`.

En MiKTeX los paquetes se instalan solos al compilar. En TeX Live instale `texlive-bibtex-extra` y `biber`.

## Reproducir el experimento

Requisitos: Docker Desktop y Docker Compose.

```bash
cd experimento
docker compose up -d
docker ps
```

Los pasos completos y la interpretación están en [`experimento/comandos.md`](experimento/comandos.md).

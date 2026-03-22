# Observability

Repository MVP per la baseline di observability del sistema.

Questa repository non implementa la logica di business delle metriche applicative.
Il suo scopo e':
- avviare lo stack di monitoring;
- raccogliere metriche con Prometheus;
- visualizzarle con Grafana;
- documentare le metriche attese dalle altre repository;
- preparare l'integrazione futura con Gateway, DataConsumer-TimescaleDB e Dashboard.

## Stato attuale

La baseline attuale monitora NATS tramite `nats-exporter` e rende disponibili in Grafana le metriche di traffico e salute del broker.

Metriche gia disponibili:
- connessioni attive NATS;
- messaggi al secondo;
- byte al secondo;
- stato target Prometheus (`up`).

Metriche target definite nella repository:
- gateway online offline;
- dimensioni payload;
- data staleness;
- throughput dati;
- errori, messaggi processati e ACK del Data Consumer;
- valori out of range;
- metriche base del backend Dashboard, se esposte in futuro.

## Dipendenze
- Infrastructure
- NATS
- Prometheus
- Grafana

## Versioni fissate
- `natsio/prometheus-nats-exporter:0.19.1`
- `prom/prometheus:v3.10.0`
- `grafana/grafana:12.4.1`

## Setup

### 1. Clonare la repository Infrastructure
`git clone https://github.com/GlitchHub-Team/Infrastructure`

### 2. Creare il file `.env`
Creare `Infrastructure/.env` a partire dai valori condivisi dal gruppo.

### 3. Completare i secret richiesti da Infrastructure
- `Infrastructure/nats/certs/server.key`
- `Infrastructure/natsManager/stream_setupper.creds`

## Avvio

Dalla root di questa repository:
`docker compose --env-file ./Infrastructure/.env up -d`

## Dashboard baseline

Grafana viene provisionato automaticamente con:
- datasource Prometheus;
- dashboard baseline di monitoring NATS e sistema.

Le dashboard si trovano nella cartella `grafana/dashboards`.

## Scope della repository

Questa repository deve fare:
- stack Docker di observability;
- configurazione Prometheus;
- provisioning Grafana;
- dashboard;
- documentazione delle metriche da raccogliere.

Questa repository non deve fare:
- implementazione delle metriche custom nei microservizi;
- logica di dominio per staleness, out of range o stato logico dei gateway;
- esposizione degli endpoint `/metrics` delle altre repository.

## Documentazione interna

Per il piano delle metriche consultare:
- `docs/metriche.md`

Il set di metriche adottato segue il flusso architetturale principale del progetto:
`Gateway simulati -> NATS -> Data Consumer -> TimescaleDB -> Dashboard`

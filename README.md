# Observability

repository MVP per observability dei dati

Questa repository monitora il progetto principalmente tramite metriche native di NATS e JetStream, senza richiedere endpoint applicativi nelle altre repository.

## Dipendenze
- NATS
- Prometheus
- Grafana

# Setup
## Clonare la repository Infrastructure
`git clone https://github.com/GlitchHub-Team/Infrastructure`

La compose di questa repository si aspetta `Infrastructure` come cartella sorella di `Observability`.

## Creare .env file
Creare un file `.env` nella cartella `Infrastructure` del progetto e prendere il contenuto da `.env` su *Bitwarden*, nella cartella *Infrastructure*

## Creare server.key
Creare un file `server.key` nella cartella `Infrastructure/nats/certs` prendere il contenuto da `server.key` su *Bitwarden*, nella cartella *Infrastructure*

## Creare stream_setupper.creds
Creare un file `stream_setupper.creds` nella cartella `Infrastructure/natsManager` prendere il contenuto da `stream_setupper.creds` su *Bitwarden*, nella cartella *Infrastructure*

# Avviare i container
`docker compose --env-file ../Infrastructure/.env up -d`

## Metriche attualmente raccolte
- stato del target Prometheus del broker NATS (`up`)
- stato health del broker NATS (`healthz`)
- connessioni attive NATS
- messaggi al secondo in ingresso e in uscita
- byte al secondo in ingresso e in uscita
- subscriptions totali via `connz`
- slow consumers via `varz`
- statistiche per account via `accstatz`
- metriche JetStream di server, stream e consumer via `jsz`

## Dashboard Grafana
Le dashboard disponibili sono:
- `NATS Overview`
  - stato del broker
  - stato health del broker
  - connessioni attive, subscriptions e slow consumers
  - throughput di messaggi e byte
- `JetStream Overview`
  - messaggi, byte, stream e consumer globali di JetStream
  - messaggi, byte e consumer per stream
  - backlog dei consumer JetStream con `num_pending`, `num_ack_pending`, `num_redelivered`, `num_waiting`
  - sequenze consumer JetStream con `delivered_*` e `ack_floor_*`

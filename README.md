# Observability

repository MVP per observability dei dati

## Dipendenze
- NATS
- Prometheus
- Grafana

# Setup
## Clonare la repository Infrastructure
`git clone https://github.com/GlitchHub-Team/Infrastructure`

## Creare .env file
Creare un file `.env` nella cartella `Infrastructure` del progetto e prendere il contenuto da `.env` su *Bitwarden*, nella cartella *Infrastructure*

## Creare server.key
Creare un file `server.key` nella cartella `Infrastructure/nats/certs` prendere il contenuto da `server.key` su *Bitwarden*, nella cartella *Infrastructure*

## Creare stream_setupper.creds
Creare un file `stream_setupper.creds` nella cartella `Infrastructure/natsManager` prendere il contenuto da `stream_setupper.creds` su *Bitwarden*, nella cartella *Infrastructure*

# Avviare i container
`docker compose --env-file ./Infrastructure/.env up -d`
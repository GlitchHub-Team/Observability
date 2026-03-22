# Metriche Observability

## Scopo

Questo documento raccoglie:
- il set di metriche per l'MVP;
- la loro motivazione architetturale e funzionale;
- la traduzione tecnica in termini Prometheus e Grafana;
- il contratto delle metriche che dovranno essere esposte dalle altre repository.

La repository `Observability` non implementa la logica di business di queste metriche.
Il suo ruolo e':
- raccoglierle con Prometheus;
- visualizzarle con Grafana;
- documentare in modo chiaro cosa deve essere monitorato e chi deve esporlo.

## Contesto architetturale

Il flusso principale del sistema e':
`Gateway simulati -> NATS -> Data Consumer -> TimescaleDB -> Dashboard`

I sensori non sono un microservizio separato: sono simulati all'interno del Gateway.
Per questo motivo molte metriche di dominio su sensori e gateway verranno esposte dal Gateway o dal Data Consumer.

## Metriche per l'MVP

| Priorita' | Metrica | Requisito o motivazione | Cosa misura |
|---|---|---|---|
| Obbligatoria | Dashboard metriche di sistema | `RF-Monitoraggio-metriche-di-sistema` | Presenza di una dashboard dedicata al monitoraggio |
| Obbligatoria | Gateway online/offline | `RF-Visualizzazione-gateway-offline-online` | Numero di gateway attivi e non attivi |
| Obbligatoria | Dimensioni payload pacchetti | `RF-Visualizzazione-dimensioni-payload-pacchetti` | Min, media e max delle dimensioni dei payload inviati |
| Obbligatoria | Data staleness | `RF-Visualizzazione-data-staleness` | Tempo dall'ultimo messaggio ricevuto per sensore |
| Desiderabile | Throughput dati | `RF-Visualizzazione-throughput-dati` | Messaggi o byte al secondo nel sistema |
| Desiderabile | Errori del Data Consumer | Coerenza con il ruolo centrale di persistenza del Data Consumer | Errori di scrittura o conferma nella pipeline di persistenza |
| Desiderabile | Messaggi processati e ACK del Data Consumer | Coerenza con il ruolo centrale di persistenza del Data Consumer | Capacita' del servizio di processare e confermare i dati |
| Opzionale | Valori out-of-range | `RF-Visualizzazione-valori-out-of-range` | Numero di valori anomali per sensore o profilo |
| Opzionale | Metriche backend Dashboard | Salute del backend cloud e dei flussi WebSocket | Richieste HTTP, errori, latenza, connessioni real-time |

## Suddivisione logica delle metriche

### 1. Metriche core del dominio IoT
Sono quelle piu' vicine ai requisiti funzionali.
- gateway online/offline;
- data staleness;
- dimensioni payload;
- valori out-of-range.

### 2. Metriche di flusso dati
Misurano se i dati scorrono correttamente nella pipeline.
- throughput dati;
- messaggi inviati dai gateway;
- messaggi processati dal Data Consumer;
- messaggi confermati dal Data Consumer;
- errori di persistenza.

### 3. Metriche di broker e baseline
Misurano lo stato del broker e della baseline di monitoring disponibile subito.
- connessioni NATS;
- messaggi al secondo;
- byte al secondo;
- stato target Prometheus.

### 4. Metriche backend cloud
Sono secondarie rispetto al dominio IoT ma utili alla salute complessiva del sistema.
- richieste HTTP;
- errori HTTP;
- latenza delle API;
- connessioni WebSocket attive.

## Traduzione tecnica in metriche Prometheus

| Categoria | Metrica funzionale | Priorita' | Nome tecnico proposto | Tipo | Labels suggerite | Servizio sorgente | Disponibile ora |
|---|---|---|---|---|---|---|---|
| Dashboard | Dashboard metriche di sistema | Obbligatoria | n/a | n/a | n/a | Grafana + Prometheus | Parzialmente si' |
| Dominio IoT | Gateway online/offline | Obbligatoria | `gateway_online_status` | gauge | `gateway_id`, `tenant_id` | Gateway o backend cloud | No |
| Dominio IoT | Dimensioni payload pacchetti | Obbligatoria | `gateway_payload_size_bytes` | histogram | `gateway_id`, `tenant_id` | Gateway o DataConsumer | No |
| Dominio IoT | Data staleness | Obbligatoria | `sensor_data_staleness_seconds` | gauge | `sensor_id`, `gateway_id`, `tenant_id` | DataConsumer o backend cloud | No |
| Flusso dati | Throughput dati | Desiderabile | `nats_varz_in_msgs`, `nats_varz_out_msgs`, `nats_varz_in_bytes`, `nats_varz_out_bytes` | counter | `job` | nats-exporter | Si' |
| Flusso dati | Messaggi inviati dai gateway | Desiderabile | `gateway_messages_sent_total` | counter | `gateway_id`, `tenant_id` | Gateway | No |
| Flusso dati | Messaggi processati dal Data Consumer | Desiderabile | `data_consumer_messages_processed_total` | counter | `consumer_id` | DataConsumer | No |
| Flusso dati | Messaggi confermati dal Data Consumer | Desiderabile | `data_consumer_messages_acked_total` | counter | `consumer_id` | DataConsumer | No |
| Flusso dati | Errori di scrittura su DB | Desiderabile | `data_consumer_write_errors_total` | counter | `consumer_id` | DataConsumer | No |
| Dominio IoT | Valori out-of-range | Opzionale | `sensor_out_of_range_total` | counter | `sensor_id`, `tenant_id`, `metric_type` | DataConsumer o backend cloud | No |
| Backend cloud | Richieste HTTP backend Dashboard | Opzionale | `http_requests_total` | counter | `route`, `method`, `status` | Dashboard backend | No |
| Backend cloud | Latenza HTTP backend Dashboard | Opzionale | `http_request_duration_seconds` | histogram | `route`, `method`, `status` | Dashboard backend | No |
| Backend cloud | Errori backend Dashboard | Opzionale | `http_requests_errors_total` | counter | `route`, `method` | Dashboard backend | No |
| Backend cloud | Connessioni WebSocket attive | Opzionale | `websocket_connections_active` | gauge | `tenant_id` | Dashboard backend | No |

## Metriche subito disponibili nella baseline

Con lo stack attuale `Observability` puo' gia' monitorare:
- throughput dati tramite `nats-exporter`;
- connessioni attive NATS;
- byte al secondo;
- stato target Prometheus tramite `up`.

## Specifica delle metriche da esporre

### Gateway

Il Gateway e' la sorgente naturale delle metriche sullo stato logico dei gateway e sulla produzione dei payload.

#### `gateway_online_status`
- Tipo: `gauge`
- Significato: stato logico del gateway simulato
- Valori attesi:
  - `1` = online
  - `0` = offline
- Labels consigliate:
  - `gateway_id`
  - `tenant_id`

#### `gateway_messages_sent_total`
- Tipo: `counter`
- Significato: numero totale di messaggi pubblicati verso NATS
- Labels consigliate:
  - `gateway_id`
  - `tenant_id`

#### `gateway_payload_size_bytes`
- Tipo: `histogram`
- Significato: dimensione dei payload inviati dal gateway
- Labels consigliate:
  - `gateway_id`
  - `tenant_id`

### DataConsumer-TimescaleDB

Il Data Consumer e' la sorgente naturale delle metriche sulla persistenza e sulla staleness dei dati.

#### `data_consumer_messages_processed_total`
- Tipo: `counter`
- Significato: numero totale di messaggi letti e processati
- Labels consigliate:
  - `consumer_id`

#### `data_consumer_messages_acked_total`
- Tipo: `counter`
- Significato: numero totale di messaggi confermati al broker con ACK
- Labels consigliate:
  - `consumer_id`

#### `data_consumer_write_errors_total`
- Tipo: `counter`
- Significato: numero totale di errori di scrittura verso TimescaleDB
- Labels consigliate:
  - `consumer_id`

#### `sensor_data_staleness_seconds`
- Tipo: `gauge`
- Significato: tempo trascorso dall'ultimo dato ricevuto per un sensore
- Labels consigliate:
  - `sensor_id`
  - `gateway_id`
  - `tenant_id`

#### `sensor_out_of_range_total`
- Tipo: `counter`
- Significato: numero totale di valori fuori range rilevati
- Labels consigliate:
  - `sensor_id`
  - `tenant_id`
  - `metric_type`

### Dashboard

La Dashboard non e' la sorgente principale delle metriche di dominio IoT, ma puo' esporre metriche utili sul backend cloud.

#### `http_requests_total`
- Tipo: `counter`
- Labels consigliate:
  - `route`
  - `method`
  - `status`

#### `http_request_duration_seconds`
- Tipo: `histogram`
- Labels consigliate:
  - `route`
  - `method`
  - `status`

#### `http_requests_errors_total`
- Tipo: `counter`
- Labels consigliate:
  - `route`
  - `method`

#### `websocket_connections_active`
- Tipo: `gauge`
- Labels consigliate:
  - `tenant_id`

## Dashboard Grafana previste

### 1. `System Metrics - Baseline`
Dashboard da usare oggi.
Mostra solo metriche gia' disponibili tramite `nats-exporter`.

Contiene:
- stato target Prometheus del job `nats`;
- connessioni attive NATS;
- ingress ed egress msg/s;
- ingress ed egress bytes/s;
- trend temporale di throughput e connessioni.

### 2. `System Metrics - Target MVP`
Dashboard da usare quando le metriche custom saranno esposte dalle altre repository.

Contiene:
- gateway online;
- gateway offline;
- worst sensor staleness;
- data consumer write errors;
- gateway message rate;
- payload size p50/p95;
- top stale sensors;
- data consumer processed vs acked.

## Priorita' di integrazione suggerita

1. Baseline NATS gia' disponibile
2. Gateway online/offline
3. Data staleness
4. Dimensioni payload
5. Errori e ACK del Data Consumer
6. Metriche backend Dashboard
7. Out-of-range

## Note operative

- Le metriche di dominio non devono essere calcolate nella repository `Observability`.
- La repository `Observability` deve raccogliere, visualizzare e documentare tali metriche.
- Le repository applicative devono esporre gli endpoint `/metrics` con le metriche custom richieste.
- Per il MVP immediato la baseline realistica resta fondata sulle metriche di NATS gia' disponibili.
- Gli alert restano una fase successiva rispetto alla sola disponibilita' delle metriche.

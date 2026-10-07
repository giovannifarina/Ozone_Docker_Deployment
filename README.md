# Apache Ozone Docker Deployment

Ambiente containerizzato per l'orchestrazione locale di un cluster **Apache Ozone (versione 2.2.1)** e di un workspace analitico **JupyterLab** integrato con protocollo S3 / Boto3 e CLI nativa Ozone.

---

## Architettura del Sistema

L'infrastruttura è orchestrata tramite [docker-compose.yml](file:///Users/giovannifarina/Ozone_Docker_Deployment/docker-compose.yml) e comprende i seguenti nodi di servizio:

| Servizio | Container | Immagine | Porta Host | Descrizione Funzionale |
|---|---|---|---|---|
| **SCM** | `ozone-scm` | `apache/ozone:2.2.1` | `9876` | Storage Container Manager (gestione container e blocchi fisici) |
| **OM** | `ozone-om` | `apache/ozone:2.2.1` | `9874` | Ozone Manager (gestione namespace logico, volumi, bucket, chiavi) |
| **DataNode** | `ozone-datanode` | `apache/ozone:2.2.1` | `9864` | Nodo di memorizzazione e persistenza dei blocchi dati |
| **S3 Gateway** | `ozone-s3g` | `apache/ozone:2.2.1` | `9878` | Gateway REST compatibile con Amazon S3 API |
| **Recon** | `ozone-recon` | `apache/ozone:2.2.1` | `9888` | Dashboard centralizzata di telemetria, diagnostica e monitoraggio |
| **JupyterLab** | `ozone-jupyter` | Build locale (`Dockerfile.jupyter`) | `8888` | Workspace interattivo provvisto di Python 3, JRE, CLI Ozone e Boto3 |

> **Nota di versione**: Le immagini dei servizi Ozone sono vincolate alla release ufficiale `apache/ozone:2.2.1` per garantire stabilità e riproducibilità operativa.

---

## Guida Operativa

### 1. Avvio dell'Infrastruttura
Eseguire il bootstrap dell'infrastruttura tramite Docker Compose:
```bash
docker compose up -d
```

### 2. Verifica dello Stato Operativo
Controllare che tutti i container siano operativi e in stato di esecuzione (`Up`):
```bash
docker compose ps
```

---

## Accesso a JupyterLab e Laboratorio di Test

1. Aprire l'interfaccia web di JupyterLab nel browser:
   **[http://localhost:8888](http://localhost:8888)**
   *(Accesso diretto preconfigurato senza autenticazione token o password)*

2. Nel file manager di JupyterLab è disponibile il notebook operativo:
   [ozone_interaction_test.ipynb](file:///Users/giovannifarina/Ozone_Docker_Deployment/ozone_interaction_test.ipynb)

3. Struttura del percorso guidato nel notebook:
   - **Parte 1 - Volume, Bucket, Key**: Gestione della gerarchia a tre livelli tramite la CLI nativa `ozone sh`.
   - **Parte 2 - Benchmark e Risoluzione dello "Small Files Problem"**: Analisi del sovraccarico di memoria heap nei tradizionali NameNode HDFS rispetto all'architettura basata su motore **RocksDB** in Ozone Manager, con test di ingestione parallela concorrente.
   - **Parte 3 - Compatibilità Nativa con l'API S3**: Interazione programmatico-applicativa tramite client Boto3 e mapping nel namespace Ozone (`/s3v/<bucket>`).
   - **Parte 4 - Monitoraggio con Apache Ozone Recon**: Verifica delle metriche di storage, nodi e pipeline.

---

## Interfacce Web e Console di Amministrazione

| Servizio | Indirizzo Web | Descrizione |
|---|---|---|
| **JupyterLab** | [http://localhost:8888](http://localhost:8888) | Ambiente di sviluppo analitico |
| **Ozone Recon** | [http://localhost:9888](http://localhost:9888) | Telemetria, ispezione container e topologia cluster |
| **Ozone Manager (OM)** | [http://localhost:9874](http://localhost:9874) | Metriche di stato e configurazione del namespace |
| **Storage Container Manager (SCM)** | [http://localhost:9876](http://localhost:9876) | Stato nodi, safemode e pipeline di replica |
| **S3 Gateway (S3G)** | [http://localhost:9878](http://localhost:9878) | Endpoint REST per integrazione API S3 |

---

## Comandi di Manutenzione e Diagnostica

- **Consultazione dei log di servizio**:
  ```bash
  docker compose logs -f s3g
  docker compose logs -f jupyter
  ```

- **Invocazione della CLI Ozone all'interno dei container**:
  ```bash
  docker exec -it ozone-scm ozone admin safemode status
  docker exec -it ozone-om ozone sh volume list
  ```

- **Arresto dei servizi**:
  ```bash
  docker compose down
  ```

- **Arresto completo con deallocazione dei volumi persistenti**:
  ```bash
  docker compose down -v
  ```
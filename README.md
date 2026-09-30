# Apache Ozone Docker Compose Lab

Questo repository contiene l'ambiente completo per avviare un cluster **Apache Ozone (versione 2.2.1)** e un container con **JupyterLab** per interagire direttamente con lo storage tramite protocollo S3 / Boto3.

---

## 🏗️ Architettura del Laboratorio

Il cluster è orchestrato tramite [docker-compose.yml](file:///Users/giovannifarina/Ozone_Docker_Deployment/docker-compose.yml) ed è composto dai seguenti container:

| Servizio | Container | Immagine | Porta Host | Descrizione |
|---|---|---|---|---|
| **SCM** | `ozone-scm` | `apache/ozone:2.2.1` | `9876` | Storage Container Manager (gestione container e blocchi) |
| **OM** | `ozone-om` | `apache/ozone:2.2.1` | `9874` | Ozone Manager (gestione namespace, volumi, bucket, chiavi) |
| **DataNode** | `ozone-datanode` | `apache/ozone:2.2.1` | `9864` | Nodo di memorizzazione blocchi dati |
| **S3 Gateway** | `ozone-s3g` | `apache/ozone:2.2.1` | `9878` | Gateway compatibile AWS S3 (REST API) |
| **Recon** | `ozone-recon` | `apache/ozone:2.2.1` | `9888` | Dashboard Web e diagnostica del cluster |
| **Jupyter** | `ozone-jupyter` | `ozone-jupyter:2.2.1` | `8888` | JupyterLab con Python 3.11, `boto3` e notebook di test |

> **Nota sulla versione**: L'immagine ufficiale utilizzata è fissata alla versione **`apache/ozone:2.2.1`** (senza ricorrere al tag `latest`).

---

## 🚀 Avvio Rapido

### 1. Avviare tutti i servizi
Dalla directory principale del progetto:
```bash
docker compose up -d
```

### 2. Verificare lo stato dei container
```bash
docker compose ps
```
Tutti i container (`ozone-scm`, `ozone-om`, `ozone-datanode`, `ozone-s3g`, `ozone-recon`, `ozone-jupyter`) devono risultare nello stato `Up`.

---

## 📓 Accesso a JupyterLab e Interazione con Ozone

1. Apri il browser all'indirizzo:
   👉 **[http://localhost:8888](http://localhost:8888)**
   *(Nessuna password o token richiesta)*

2. Nel file browser di Jupyter troverai il notebook di test:
   👉 [ozone_interaction_test.ipynb](file:///Users/giovannifarina/Ozone_Docker_Deployment/ozone_interaction_test.ipynb)

3. Il notebook interagisce con Ozone tramite S3 Gateway. Il container Jupyter è configurato con:
   - **Parte 1 - La Nuova Gerarchia (Volume, Bucket, Key)**: Dimostrazione pratica con la CLI `ozone sh` della struttura multi-tenant disaccoppiata.
   - **Parte 2 - Lo \"Small File Stress Test\" (Killer Feature ⭐⭐⭐)**: Spiegazione teorica del collasso del NameNode HDFS (metadati in RAM) vs Ozone (metadati su **RocksDB** in OM) e benchmark ad alte prestazioni con migliaia di piccoli file.
   - **Parte 3 - Compatibilità Nativa con l'API S3**: Integrazione con Boto3, mapping bidirezionale con i volumi Ozone (`/s3v/<bucket>`) e gestione dei metadati personalizzati.
   - **Parte 4 - Monitoraggio con Recon**: Visualizzazione grafica dello stato del cluster su `http://localhost:9888`.
   - Endpoint preconfigurato: `http://s3g:9878` (all'interno della rete Docker) e `http://localhost:9878` (dall'host).
   - Libreria `boto3`, CLI Ozone e Java JRE preinstallati nel container Jupyter.
   - Volume montato per persistere automaticamente ogni modifica apportata ai notebook sul filesystem locale.

---

## 🌐 Dashboard e Interfacce Web

| Servizio | URL Web | Note |
|---|---|---|
| **JupyterLab** | [http://localhost:8888](http://localhost:8888) | Ambiente interattivo Python |
| **Ozone Recon** | [http://localhost:9888](http://localhost:9888) | Dashboard di monitoraggio e panoramica cluster |
| **Ozone Manager (OM)** | [http://localhost:9874](http://localhost:9874) | Metriche e configurazione OM |
| **Storage Container Manager (SCM)** | [http://localhost:9876](http://localhost:9876) | Stato pipeline, nodi e safemode SCM |
| **S3 Gateway** | [http://localhost:9878](http://localhost:9878) | Endpoint REST compatibile S3 |

---

## 🛠️ Comandi Utili

- **Visualizzare i log di un servizio** (es. S3 Gateway o Jupyter):
  ```bash
  docker compose logs -f s3g
  docker compose logs -f jupyter
  ```

- **Eseguire un comando CLI Ozone nel cluster**:
  ```bash
  docker exec -it ozone-scm ozone admin safemode status
  docker exec -it ozone-om ozone sh volume list
  ```

- **Arrestare il laboratorio**:
  ```bash
  docker compose down
  ```

- **Arrestare rimuovendo anche i volumi dati**:
  ```bash
  docker compose down -v
  ```
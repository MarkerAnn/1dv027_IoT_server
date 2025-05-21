
# 🌿 IoT Environment Monitor with TIG Stack & Raspberry Pi Pico

Monitor air and soil conditions using a **Raspberry Pi Pico WH**, stream sensor data via **MQTT**, store in **InfluxDB**, and visualize live dashboards with **Grafana** — all securely deployed on your VPS via **GitHub Actions**.

---

## 🚀 Live System

* **Grafana Dashboard**: [angelicamarker.online/1dv027/iot/grafana](https://angelicamarker.online/1dv027/iot/grafana)
* **InfluxDB UI**: [influx.angelicamarker.online](https://influx.angelicamarker.online)

---

## 🧩 Full Stack Overview

| Component                | Role                                                   |
| ------------------------ | ------------------------------------------------------ |
| **Raspberry Pi Pico WH** | Publishes JSON sensor data via MQTT (`pico/sensors/#`) |
| **SCD40 Sensor**         | CO₂, temperature, and relative humidity                |
| **STEMMA Soil Sensor**   | Soil moisture (analog → JSON)                          |
| **RGB + CO₂ LEDs**       | Signal sensor states visually                          |
| **Mosquitto**            | MQTT broker (receives data from Pico)                  |
| **Telegraf**             | Listens to MQTT → writes to InfluxDB                   |
| **InfluxDB v2.7**        | Time-series storage of sensor metrics                  |
| **Grafana OSS**          | Visualizes InfluxDB data with custom dashboards        |
| **GitHub Actions**       | CI/CD for deploying stack to your VPS automatically    |
| **Nginx + SSL**          | Routes Grafana & InfluxDB through public subdomains    |

---

## 📡 Architecture Flow

```plaintext
[SCD40]    [Soil Sensor]
   │              │
   ▼              ▼
┌────────────────────┐
│ Raspberry Pi Pico  │
│ Publishes via MQTT │
└────────┬───────────┘
         ▼
   ┌────────────┐
   │ Mosquitto  │ ← MQTT broker
   └────┬───────┘
        ▼
   ┌────────────┐
   │  Telegraf  │ ← MQTT Consumer Plugin
   └────┬───────┘
        ▼
   ┌────────────┐
   │  InfluxDB  │ ← Time-series DB
   └────┬───────┘
        ▼
   ┌────────────┐
   │  Grafana   │ ← Visualization Dashboards
   └────────────┘
```

---

## ⚙️ Setup Instructions

### 🔐 1. Create `.env` File

At the root of your project (next to `docker-compose.yml`):

```env
INFLUXDB_USERNAME=admin
INFLUXDB_PASSWORD=your-password
INFLUXDB_ORG=1dv027-iot
INFLUXDB_BUCKET=sensor-data
INFLUXDB_TOKEN=your-long-admin-token
```

---

### 🐳 2. Start with Docker Compose

From your project root:

```bash
docker compose up -d
```

This brings up:

* Mosquitto (port 1883)
* InfluxDB (port 8086)
* Telegraf (MQTT ➝ InfluxDB)
* Grafana (port 3000)

---

### 🛰️ 3. GitHub Actions CI/CD Flow

Deployment is automated via GitHub Actions when pushing to the `main` branch.

**Required secrets in GitHub repo:**

| Secret Name       | Description                    |
| ----------------- | ------------------------------ |
| `SSH_PRIVATE_KEY` | Deploy key for VPS             |
| `VPS_HOST`        | IP/domain of your VPS          |
| `VPS_USER`        | VPS username (e.g. `angelica`) |

Workflow file: `.github/workflows/deploy.yml`

It runs:

```bash
ssh your-vps 'git pull && docker compose up -d'
```

---

### 💡 4. Simulate MQTT Sensor Data (Without Pico)

To test everything before plugging in your Pico:

1. **Install** MQTT client:

```bash
sudo apt install mosquitto-clients
```

2. **Send fake data:**

```bash
mosquitto_pub -h localhost -t pico/sensors/test -m '{"temp": 21.7, "humidity": 40.5}'
```

Telegraf will pick it up and forward it to InfluxDB.

---

### 📈 Grafana Dashboard (Coming Soon)

Create a new dashboard at:

➡️ `https://angelicamarker.online/1dv027/iot/grafana`

* **Data source**: InfluxDB
* **Query**: From `sensor-data` bucket
* Panels: CO₂ level, temperature, humidity, soil moisture

---


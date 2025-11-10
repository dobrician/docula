# Arhitectură Microservicii – Rezumat Tehnic

## 1. Obiectiv
Construirea unei arhitecturi de microservicii self-hosted, rulată pe un singur server bare-metal (128 GB RAM), cu posibilitate de scalare ulterioară pe mai multe servere sau datacentere.

---

## 2. Componente principale

### 2.1. **Containerizare**
- **Docker** pentru fiecare microserviciu.
- **Docker Compose** pentru orchestrare inițială.
- Avantaje:
  - izolare completă;
  - scalare ușoară (replici);
  - deployment rapid.

---

## 3. Load Balancing & Reverse Proxy

### 3.1. **Nginx Proxy Manager (NPM)**
- Folosit ca reverse proxy + load balancer.
- Rutează traficul API către multiple instanțe ale aceluiași microserviciu.
- Permite scaling orizontal:
  - containere multiple pe același host;
  - containere pe host-uri diferite în viitor.

### 3.2. Tipuri de load balancing în NPM
- Round-robin
- Least connections
- Weighted (manual prioritization)

---

## 4. Sistem de procesare task-uri masive

### 4.1. Problemă
- Milioane de job-uri JSON necesită procesare paralelă.

### 4.2. Soluție: **Message Broker**
- **Valkey** (alternativa modernă open-source la Redis)
- **RabbitMQ** (alternativă matură și stabilă)

### 4.3. Beneficii
- Cozi de task-uri distribuite.
- Mai multe instanțe docker ale aceluiași worker procesează în paralel.
- Fiecare worker “claim-uiește” un job și îl marchează ca procesat.

---

## 5. Bază de date document-based (JSON)

### 5.1. **MongoDB / Community Edition**
- Model: document-based.
- Suportă **JSON nativ**.
- Replicare + sharding integrate.

### 5.2. Scalare
- Poți porni pe o singură instanță.
- Poți extinde ulterior:
  - replicare pe 3+ servere;
  - sharding pentru volume mari;
  - georeplicare între datacentere.

### 5.3. Avantaj
- Trecere simplă de la single-node la cluster multi-server.

---

## 6. Arhitectură inițială (One-Server Setup)


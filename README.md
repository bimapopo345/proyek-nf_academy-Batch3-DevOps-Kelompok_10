# File Upload Application with Monitoring and Alerting

## Deskripsi
Aplikasi upload file dengan monitoring disk usage dan error rate, dilengkapi dengan sistem alerting.

## Komponen Sistem
- Upload App (Port 8000): Aplikasi utama untuk upload file
- Prometheus (Port 9091): Monitoring metrics
- AlertManager (Port 9093): Pengelolaan alert
- Grafana (Port 3000): Visualisasi metrics

## Cara Menjalankan Aplikasi

1. Clone repository dan masuk ke direktori:
```bash
git clone [repository-url]
cd Project-akhir-kel10
```

2. Jalankan aplikasi:
```bash
docker-compose up -d
```

3. Verifikasi semua container berjalan:
```bash
docker-compose ps
```

## Panduan Testing Lengkap

### 1. Testing Upload File

#### a. Akses Web Interface
- Buka http://localhost:8000
- Interface upload file akan ditampilkan

#### b. Test Upload File Normal
```bash
# 1. Upload file < 100MB melalui web interface
# 2. Verifikasi di CLI:
docker exec upload-app ls -lh /app/uploads

# 3. Verifikasi metrics di Prometheus:
# Buka http://localhost:9091
# - Masuk ke tab "Graph"
# - Masukkan query: upload_total
# - Klik "Execute"
# Atau via CLI:
curl "http://localhost:9091/api/v1/query?query=upload_total"
```

#### c. Test Batasan Ukuran File
```bash
# 1. Coba upload file > 100MB
# Hasil yang diharapkan: 
# - Pesan error "File too large"
# - Metrics upload_failed bertambah

# Verifikasi via CLI:
curl "http://localhost:9091/api/v1/query?query=upload_failed"
```

### 2. Testing Error Rate Alert

#### a. Memicu Error 500
```powershell
# Generate error 500:
1..30 | ForEach-Object { 
    curl http://localhost:8000/test500
    Start-Sleep -Milliseconds 20 
}
```

#### b. Verifikasi di Multiple Interface

1. CLI Verification:
```bash
# Cek rate error:
curl "http://localhost:9091/api/v1/query?query=rate(http_requests_total{status=\\"500\\"}[1m])"

# Cek status alert:
curl http://localhost:9093/api/v2/alerts
```

2. Web Interface Verification:
- Prometheus (http://localhost:9091):
  - Buka tab "Alerts"
  - Cari alert "High500ErrorRate"
  - Status harus "FIRING"

- AlertManager (http://localhost:9093):
  - Alert harus muncul di halaman utama
  - Klik alert untuk detail lengkap

- Grafana (http://localhost:3000):
  - Login dengan admin/admin
  - Import dashboard baru
  - Add panel dengan query: rate(http_requests_total{status="500"}[1m])

### 3. Testing Disk Usage Alert

#### a. Memicu High Disk Usage
```bash
# Buat file besar di dalam container:
docker exec upload-app dd if=/dev/zero of=/app/uploads/bigfile.dat bs=1M count=120
```

#### b. Verifikasi Multi-Interface

1. CLI Verification:
```bash
# Cek penggunaan disk:
curl "http://localhost:9091/api/v1/query?query=disk_usage_percent"

# Cek status alert:
curl http://localhost:9093/api/v2/alerts
```

2. Web Interface Verification:
- Prometheus (http://localhost:9091):
  - Graph query: disk_usage_percent
  - Tab Alerts: Cek "HighDiskUsage"

- AlertManager (http://localhost:9093):
  - Verifikasi alert "HighDiskUsage"
  - Severity: warning

- Grafana (http://localhost:3000):
  - Add panel baru
  - Query: disk_usage_percent
  - Set threshold line di 80%

### 4. Monitoring di Grafana

#### a. Setup Initial
1. Akses http://localhost:3000
2. Login dengan:
   - Username: admin
   - Password: admin
3. Add Prometheus data source:
   - URL: http://prometheus:9091
   - Access: Browser

#### b. Create Dashboards
1. Create new dashboard
2. Add panels untuk:
   - Disk Usage: disk_usage_percent
   - Error Rate: rate(http_requests_total{status="500"}[1m])
   - Upload Success: rate(upload_total[5m])
   - Upload Failed: rate(upload_failed[5m])

### 5. AlertManager Configuration

#### a. Akses Web Interface
- Buka http://localhost:9093
- Review active alerts
- Cek alert history

#### b. Verifikasi Webhook
```bash
# Cek log aplikasi untuk webhook alerts:
docker-compose logs upload-app | grep "Alert received"
```

## Troubleshooting

### 1. Alert Tidak Muncul
```bash
# 1. Cek rules loaded:
curl http://localhost:9091/api/v1/rules

# 2. Cek targets up:
curl http://localhost:9091/api/v1/targets

# 3. Verifikasi metrics:
curl http://localhost:8000/metrics
```

### 2. File Upload Gagal
```bash
# 1. Cek permission:
docker exec upload-app ls -la /app/uploads

# 2. Cek logs:
docker-compose logs upload-app

# 3. Verifikasi storage:
docker exec upload-app df -h /app/uploads
```

### 3. Metrics Tidak Muncul
```bash
# 1. Cek endpoint metrics:
curl http://localhost:8000/metrics

# 2. Cek Prometheus targets:
curl http://localhost:9091/api/v1/targets

# 3. Cek Prometheus logs:
docker-compose logs prometheus
```

## Expected Results

1. Upload File:
- File < 100MB: Success, stored in uploads/
- File > 100MB: Rejected with error
- Invalid format: Rejected with error

2. Error 500 Alert:
- Trigger: > 0.05 errors/second
- Alert visible in Prometheus & AlertManager
- Webhook notification received

3. Disk Usage Alert:
- Trigger: > 80% usage
- Alert visible after 5 minutes
- Alert resolves when usage decreases

4. Monitoring:
- All metrics visible in Prometheus
- Grafana shows real-time data
- AlertManager displays active alerts
## Screenshot Examples

### 1. Upload Interface
![Upload Interface](docs/images/upload-interface.png)

### 2. Prometheus Alert Rules
![Prometheus Rules](docs/images/prometheus-rules.png)

### 3. AlertManager Interface
![AlertManager](docs/images/alertmanager.png)

### 4. Grafana Dashboard
![Grafana Dashboard](docs/images/grafana-dashboard.png)

*Note: Screenshots akan ditambahkan saat implementasi. Gambar di atas hanya placeholder.

# OTOBO Helm Chart

Kubernetes Deployment für OTOBO Open Source Ticket System.

## Voraussetzungen

- Kubernetes Cluster
- Helm 3.x
- MySQL Datenbank erreichbar (z.B. als Kubernetes Service)
- StorageClass verfügbar (default: `your-storage-class`)

## Installation

```bash
# Mit eigenem Passwort
helm install otobo ./charts/otobo \
  --set secret.password=deinPasswort \
  --set db.host=otobo-service-mysql

# Mit bestehendem Secret
helm install otobo ./charts/otobo \
  --set secret.existingSecret=mein-secret \
  --set secret.existingSecretKey=OTOBO_DB_PASSWORD \
  --set db.host=otobo-service-mysql
```

## Lokales Testen mit Minikube

```bash
minikube start

helm install otobo ./charts/otobo \
  --set secret.password=test \
  --set db.host=otobo-service-mysql \
  --set persistence.storageClass=standard \
  --set nginx.service.type=NodePort

# Installer aufrufen
minikube service otobo-nginx
```

## Nach der Installation

Installer aufrufen:
```
http://<nginx-ip>/otobo/installer.pl
```

Im Installer bei **Database Host** eintragen:
```
otobo-service-mysql
```

## Werte

| Parameter | Beschreibung | Default |
|-----------|-------------|---------|
| `image.tag` | OTOBO Image Tag | `latest-10_1` |
| `db.host` | MySQL Hostname | `otobo-service-mysql` |
| `db.port` | MySQL Port | `3306` |
| `db.name` | Datenbankname | `otobo` |
| `db.user` | Datenbankuser | `otobo` |
| `secret.password` | DB Passwort | `changeme` |
| `secret.existingSecret` | Bestehendes Secret nutzen | `""` |
| `persistence.storageClass` | StorageClass | `your-storage-class` |
| `persistence.size` | Volume Größe | `10Gi` |
| `nginx.service.type` | Service Typ | `LoadBalancer` |

## Struktur

```
web + daemon → laufen im selben Pod (shared PVC, kein RWX nötig)
nginx        → eigener Pod als Reverse Proxy
```

## Debugging

```bash
# Logs web
kubectl logs <pod> -c web

# Logs daemon  
kubectl logs <pod> -c daemon

# In Container rein
kubectl exec -it <pod> -c web -- bash

# Entrypoint analysieren
kubectl exec -it <pod> -c web -- cat /entrypoint.sh

# Verzeichnis prüfen
kubectl exec -it <pod> -c web -- ls -la /opt/otobo/
```

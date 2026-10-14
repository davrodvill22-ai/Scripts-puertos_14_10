# 🛡️ Script de Prueba de Carga HTTP / DDoS (Fail2ban Testing)

Este repositorio contiene el script de prueba de sobrecarga HTTP concurrente (`test_ddos.py`) diseñado para evaluar y demostrar la efectividad de las reglas de protección contra inundaciones HTTP (**HTTP Flood / DDoS**) implementadas con **Fail2ban** e **iptables** sobre los contenedores Docker de la práctica (Frontend, Backend y Keycloak).

---

## 🎯 Objetivos de la Prueba (Puertos Evaluados)

El script apunta secuencialmente a los tres servicios protegidos expuestos a través del Proxy Inverso:
- **Frontend Dashboard (React / Nginx):** `http://localhost:5173/`
- **Backend API (Node.js / Express):** `http://localhost:3001/`
- **Servidor de Autenticación (Keycloak / LDAP):** `http://localhost:8081/`

---

## 🚦 Parámetros y Límites de Tráfico Configurados

El servicio Fail2ban en el proxy monitorea el log de accesos (`/var/log/nginx/access.log`) con las siguientes directivas:

* **Tolerancia máxima (`maxretry`):** 60 peticiones dentro de la ventana de tiempo.
* **Ventana de tiempo (`findtime`):** 10 segundos.
* **Tiempo de baneo inicial (`bantime`):** 10 minutos (con incremento exponencial para reincidentes).

> **Comportamiento esperado:** Al sobrepasar las 60 peticiones en menos de 10 segundos, Fail2ban inserta una regla de bloqueo en el firewall (`iptables-allports`). A partir de ese momento, las peticiones fallarán con código `000` (Connection Refused o Timeout) y el reporte mostrará un **100% de peticiones bloqueadas/fallidas**.

---

## 🚀 Cómo Ejecutar la Prueba

### Requisitos
- Python 3.8 o superior (utiliza exclusivamente la biblioteca estándar, sin requerir paquetes externos).

### Ejecución Básica
Desde la terminal en el directorio del script:

```bash
python test_ddos.py
```

### Configuración de Parámetros mediante Variables de Entorno

Puedes ajustar el comportamiento del test definiendo variables de entorno antes de ejecutarlo:

| Variable | Por Defecto | Máximo | Descripción |
|---|---|---|---|
| `DURATION` | `20` | `300` | Duración de la prueba en segundos por cada objetivo |
| `CONCURRENCY` | `40` | `500` | Número de trabajadores/hilos paralelos simultáneos |
| `REQUESTS_PER_SECOND` | `50` | `500` | Tasa máxima de peticiones por segundo |

#### Ejemplo en PowerShell (Windows):
```powershell
$env:DURATION=15; $env:CONCURRENCY=50; $env:REQUESTS_PER_SECOND=100; python test_ddos.py
```

#### Ejemplo en Bash (Linux / macOS):
```bash
DURATION=15 CONCURRENCY=50 REQUESTS_PER_SECOND=100 python3 test_ddos.py
```

---

## 📊 Interpretación de Resultados

Durante la ejecución, el script reporta periódicamente:
```text
--- Target: localhost:5173 (40 workers, 50 requests/s max, 20s) ---
  [14:15:03] requests so far for localhost:5173: total=60 ok(200)=60 blocked/failed=0
  [14:15:06] requests so far for localhost:5173: total=210 ok(200)=60 blocked/failed=150
  RESULT for localhost:5173: total=850 ok(200)=60 blocked/failed=790
  -> 92% of responses were non-200
```
- `ok(200)`: Peticiones que respondieron con éxito antes del bloqueo.
- `blocked/failed`: Peticiones que fueron bloqueadas por el firewall o rechazadas por límite de tasa.

---

## 🔓 Cómo Desbloquear una IP Baneada (Unban Manual)

Si necesitas remover el baneo antes de que expire la ventana de 10 minutos para continuar con pruebas o desarrollo, ejecuta en el host:

```bash
docker exec fail2ban-proxy fail2ban-client unban --all
```

Para verificar el estado actual de la jaula y las IPs actualmente bloqueadas:
```bash
docker exec fail2ban-proxy fail2ban-client status http-flood
```

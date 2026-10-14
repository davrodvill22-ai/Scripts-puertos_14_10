# 🔍 Scripts de Verificación de Puertos

Colección de scripts para verificar rápidamente si un puerto específico se encuentra abierto (escuchando) o cerrado en el sistema local (`localhost`).

---

## 📂 Contenido del Repositorio

- **`Check-Port.ps1`**: Script para entornos **Windows** (PowerShell) utilizando `Test-NetConnection`.
- **`check_port.sh`**: Script para entornos **Linux** (Bash) utilizando la utilidad `ss`.

---

## 💻 Uso en Windows (PowerShell)

Ejecuta el script pasando el parámetro `-Puerto`:

```powershell
.\Check-Port.ps1 -Puerto 8080
```

O de forma interactiva (solicitará el número de puerto si no se especifica):
```powershell
.\Check-Port.ps1
```

---

## 🐧 Uso en Linux (Bash)

1. Otorgar permisos de ejecución (si no los tiene):
   ```bash
   chmod +x check_port.sh
   ```

2. Ejecutar pasando el número de puerto como argumento:
   ```bash
   ./check_port.sh 8080
   ```

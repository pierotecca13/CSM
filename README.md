# CSM Dashboard v2

Dashboard de Customer Success con integración JIRA Cloud, exportaciones (Excel/PDF/PPT) y soporte opcional para MySQL.

---

## Requisitos previos

- Python 3.8 o superior
- Acceso a un proyecto en JIRA Cloud
- Un [API token de Atlassian](https://id.atlassian.com/manage-profile/security/api-tokens)

---

## Instalación y ejecución

```bash
# 1. Instalar dependencias
pip install -r requirements.txt

# 2. Iniciar el servidor
python app.py
```

Luego abre en tu navegador: **http://localhost:5050**

---

## Configuración inicial (Login)

En el tab **Configuración** del dashboard ingresa:

| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| URL JIRA | URL base de tu instancia | `https://tuempresa.atlassian.net` |
| Email | Email de tu cuenta Atlassian | `tu@email.com` |
| API Token | Token generado en Atlassian | `ATATTxxx...` |
| Proyecto | Clave corta del proyecto | `SV`, `IT`, `SUPPORT` |
| Campo SLA | Campo SLA (dejar en `auto`) | `auto` |
| Días | Rango de análisis | `90` |

> La clave del proyecto aparece en las URLs de JIRA: `https://empresa.atlassian.net/browse/SV-123` → clave: `SV`

---

## Variables de entorno (opcional)

Para evitar ingresar credenciales manualmente cada vez:

```bash
export JIRA_URL=https://tuempresa.atlassian.net
export JIRA_EMAIL=tu@email.com
export JIRA_TOKEN=tu_api_token
python app.py
```

---

## Funcionalidades

### Dashboard principal
- KPIs de soporte: MTTR, SLA compliance, tickets abiertos/cerrados
- Percentiles de resolución (P50, P90, P95)
- Tickets envejecidos (+7 y +14 días)
- Tasa de reapertura y tasa de bugs
- Tendencias semanales (nuevos vs. cerrados)
- Vista por cliente con health score individual

### Exportaciones

| Formato | Librería requerida |
|---------|--------------------|
| Excel   | `pip install openpyxl` |
| PDF     | `pip install reportlab` |
| PPT     | Incluido (PptxgenJS vía CDN) |

### MySQL (opcional)

```bash
pip install mysql-connector-python
```

En el tab **MySQL** ingresa las credenciales de tu base de datos y ejecuta consultas personalizadas.

Query de ejemplo — productos despachados por trimestre:

```sql
SELECT
  cliente,
  CONCAT('Q', QUARTER(fecha_despacho), ' ', YEAR(fecha_despacho)) AS trimestre,
  COUNT(*) AS total_despachados,
  SUM(cantidad) AS unidades
FROM despachos
WHERE fecha_despacho >= DATE_SUB(NOW(), INTERVAL 12 MONTH)
GROUP BY cliente, trimestre
ORDER BY cliente, trimestre;
```

---

## Solución de problemas

### No aparecen tickets (Issues encontrados: 0)

1. **Verificar la clave del proyecto** — debe ser la sigla corta (ej. `SV`, no el nombre completo)
2. **Ejecutar el diagnóstico** — ve al tab **Debug**, ingresa la clave y haz clic en *Ejecutar diagnóstico*. El diagnóstico reporta:
   - Cuántos issues tiene el proyecto
   - Qué campos SLA tienen datos
   - Qué campo de organización/cliente usar
3. **Campo SLA** — mantén la opción `auto`; si el diagnóstico muestra un campo específico puedes usarlo (ej. `customfield_11324`)
4. **Ampliar el rango de días** — si el proyecto es nuevo, prueba con `365` en lugar de `90`

### El servidor no inicia

```bash
# Verificar que el puerto 5050 esté libre
# Windows:
netstat -ano | findstr :5050

# Instalar dependencias faltantes
pip install -r requirements.txt
```

---

## Estructura del proyecto

```
csm/
├── app.py              ← Backend Flask (API REST + lógica de métricas)
├── requirements.txt    ← Dependencias Python
├── cache.json          ← Caché de datos JIRA (se genera automáticamente)
├── README.md           ← Este archivo
└── static/
    └── index.html      ← Frontend completo (SPA con Chart.js)
```

---

## Stack tecnológico

**Backend:** Python · Flask · Requests · openpyxl · ReportLab

**Frontend:** HTML5 · CSS3 · JavaScript · Chart.js · PptxgenJS

**Integración:** JIRA Cloud API v3 · MySQL (opcional)

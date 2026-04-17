# n8n Workflows — Logos Night Runner

6 workflows de alto impacto, listos para importar en n8n.

---

## Cómo importar

1. Abrir n8n (http://tu-servidor:5678)
2. Ir a **Workflows** → **Import from JSON**
3. Pegar el contenido del archivo `.json` correspondiente
4. Configurar credenciales en **Credentials**
5. Activar el workflow

---

## Credential Setup Required

| Credential | workflows que lo usan |
|------------|----------------------|
| **Google Sheets** | 01, 02, 04, 05, 06 |
| **Gmail** | 02, 03, 04 |
| **Google Tasks** | 02, 03 |
| **Google Drive** | 05 |
| **Notion API** | 01, 03 |
| **OpenAI** (o MiniMax via API) | 02, 05, 06 |
| **Tavily** | 05 |
| **Pushover** | 01, 02, 03, 04, 05, 06 |

### Env vars a configurar en n8n:

```
NOTION_API_KEY=secret_xxx
NOTION_LEADS_DB=xxx
NOTION_CLIENTS_DB=xxx
GAS_ERROR_SCRIPT_ID=xxx
CLEARBIT_API_KEY=xxx
YT_WATCH_LATER_ID=xxx
```

---

## Workflows

### 01 — Lead Enrichment Pipeline
**Trigger:** Diario 8AM  
**Qué hace:**
- Lee spreadsheet de leads
- Deduplica contra lista ya procesada
- Enriquece emails (validación + company data)
- Scoring (0-100): tamaño empresa, industria, título, ubicación
- Routeo por tier (A/B/C)
- Tier A → Notion CRM + notificación push
- Actualiza spreadsheet de procesados

**Credentials:** Google Sheets, Notion, Pushover

---

### 02 — Email Intake → Task Pipeline
**Trigger:** Cada 1 minuto (Gmail polling)  
**Qué hace:**
- Detecta nuevos emails
- Filtra solo de contactos
- Envía body a AI para extraer action items
- Crea Google Tasks por cada action item
- Label email como "processed-by-logos"
- Notifica por push

**Credentials:** Gmail, Google Tasks, OpenAI/MiniMax, Pushover

---

### 03 — Client Onboarding Automation
**Trigger:** Webhook (`/webhook/client-onboard`)  
**Qué hace:**
- Recibe payload POST con datos del cliente
- Genera 8 tareas de onboarding predefinidas
- Envía email de bienvenida
- Crea página en Notion (cliente + empresa + servicio)
- Notifica a Anthony por push

**Payload ejemplo:**
```json
{
  "client_name": "María López",
  "client_email": "maria@empresa.com",
  "company": "TechCorp CR",
  "service": "Consultoría AI"
}
```

**Credentials:** Gmail, Google Tasks, Notion, Pushover

---

### 04 — Invoice + Payment Follow-up
**Trigger:** Diario 9AM  
**Qué hace:**
- Lee spreadsheet de facturas
- Filtra facturas vencidas (no pagadas)
- Envía recordatorios escalonados:
  - 1 día: recordatorio amable
  - 7 días: primera alerta
  - 14 días: segunda alerta (más firme)
  - 30 días: último aviso + notificación prioritaria
- Actualiza status en spreadsheet
- Notificaciones push para facturas críticas (30+ días)

**Credentials:** Google Sheets, Gmail, Pushover

---

### 05 — AI Research Agent Pipeline
**Trigger:** Cada 6 horas  
**Qué hace:**
- Lee cola de research (spreadsheet)
- Procesa hasta 3 queries pendientes
- Busca en web (Tavily)
- Resume hallazgos con AI (estructurado en español)
- Guarda reportes en Google Drive (.md)
- Actualiza status en cola
- Notifica por push

**Cola (spreadsheet columns):** `query`, `topic`, `priority`, `status`, `requested_at`

**Credentials:** Google Sheets, Google Drive, Tavily, OpenAI/MiniMax, Pushover

---

### 06 — Social Content Repurposing Pipeline
**Trigger:** Diario 6PM  
**Qué hace:**
- Lee RSS de YouTube Watch Later
- Obtiene transcript de videos
- AI extrae:
  - 3 tweets (puntos clave)
  - 1 post de LinkedIn
  - 1 snippet para newsletter
- Guarda todo en Content Calendar (Google Sheets)
- Notifica por push

**Credentials:** YouTube RSS, OpenAI/MiniMax, Google Sheets, Pushover

---

## Spreadsheet Templates

Crear estas hojas antes de activar los workflows:

### Lead Enrichment (Workflow 01)
**Hoja 1:** `leads` con columnas: `name`, `email`, `company`, `industry`, `company_size`, `job_title`, `location`, `source`, `created_at`  
**Processed Leads:** `email`, `name`, `score`, `processed_at`

### Research Queue (Workflow 05)
Columnas: `query`, `topic`, `priority` (high/medium/low), `status` (pending/completed), `requested_at`, `completed_at`

### Content Calendar (Workflow 06)
Columnas: `content_type`, `content`, `source_video`, `source_url`, `scheduled_date`, `status`

### Invoices (Workflow 04)
Columnas: `invoice_number`, `client_name`, `client_email`, `amount`, `due_date`, `status`, `last_reminder`

---

## Error Handling

Todos los workflows tienen:
- **Error trigger** → captura cualquier error
- **Notificación Pushover** → te avisa inmediatamente
- **Log en spreadsheet** → guarda el error para auditoría

---

## Recomendaciones

1. Empezar con **workflow 03** (onboarding) — es el más simple de testear (webhook)
2. Luego **workflow 04** (invoices) — muy alto ROI en tiempo salvado
3. Luego **workflow 02** (email → tasks) — si usás Gmail mucho
4. Los de research y content son para cuando tengas el sistema rodando

---

_Last updated: 2026-04-16 by Logos_

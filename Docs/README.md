# TutorBot - Optimizado (Telegram + Google Sheets + Google Gemini)

Este README explica cómo importar y configurar el workflow "TutorBot - Optimizado" en n8n. El JSON del workflow (workflow-tutorbot-optimizado.json) fue generado sin claves o secretos incrustados — debes crear las credenciales en tu instancia de n8n y asignarlas a los nodos correspondientes.

---

## 1. Requisitos previos
- Una instancia de n8n (self-hosted o n8n cloud) con acceso a Internet.
- Una cuenta de Google con acceso al Spreadsheet.
- Un bot de Telegram (token).
- Una API key para Google Gemini (o la integración que uses en n8n).
- Spreadsheet con ID: `1RTZiuOpuPfLvGUFarwTBgEM1qIZuySJh6iHOSdvnE9E`
- Hojas dentro del spreadsheet (nombres exactos, minúsculas):
  - `tutores`
  - `disponibilidad`
  - `tutorias`
  - `sessions`
  - (opcional) `error_log`, `activity_log`

Formato esperado para algunas hojas:
- `tutores`: id_tutor, nombre, especialidad_materias, estado, telegram_user
  - añade la columna `telegram_user` con el chat_id numérico del tutor (ej: 123456789).
- `disponibilidad`: id_dispo, id_tutor, dia_semana, hora_inicio, hora_fin, estado, row_number
- `tutorias`: id_tutoria, id_estudiante, id_tutor, materia, fecha, hora, estado, row_number_disp, created_at
- `sessions`: telegram_user, pantalla_actual, paso_actual, datos_parciales

---

## 2. Importar workflow en n8n
1. Abre n8n → Workflows → Import → Paste JSON.
2. Pega el contenido del archivo `workflow-tutorbot-optimizado.json` y confirma la importación.

---

## 3. Crear credenciales en n8n (no incrustar en JSON)
Crea las siguientes credenciales en n8n:

### A. Telegram account (Telegram Bot)
- Tipo: Telegram Bot
- Token: (tu bot token, por ejemplo `8869...`)
- Nombre de la credencial: `Telegram account`

### B. Google Sheets account
- Tipo: Google Sheets OAuth2
- Client ID: (tu IdClient)
- Client Secret: (tu SecretId)
- Nombre de la credencial: `Google Sheets account`
- Conecta la cuenta a la hoja con ID indicada.

### C. Google Gemini
- Crea una credencial según la integración disponible en tu n8n (PaLM/Gemini). Nombre sugerido: `Google Gemini`.
- Pega la API key en la credencial.

> Nota: No pongas las claves dentro del JSON. Esta es la forma segura: las credenciales se guardan en el entorno n8n.

---

## 4. Asignar credenciales a nodos
Después de importar:
- Telegram Trigger y todos los nodos Telegram → asigna `Telegram account`.
- Todos los nodos Google Sheets → asigna `Google Sheets account`.
- Nodos AI (AI Agent1, Motor IA Asignacion) → asigna `Google Gemini`.

---

## 5. Valores que debes revisar en el workflow
- `node-notificar-admin` → cambia `REEMPLAZAR_CHAT_ID_ADMIN` por el chat_id del admin para notificaciones de error.
- En la hoja `tutores` asegúrate de incluir la columna `telegram_user` (chat_id numérico) para que el workflow pueda notificar automáticamente al tutor. Si no la provees, el flujo envía la notificación al admin como fallback.

---

## 6. Pruebas recomendadas (ordenadas)
1. **Registro**
   - Envía un mensaje al bot desde Telegram con un usuario nuevo y verifica que AI Agent1 pida registro y que se guarde en `sessions` o `tutores` según tu lógica.
2. **Seleccionar materia**
   - Pide menú de materias, elige una materia (callback `MAT_...`) y verifica que el flujo haga `Guardar Materia` y pida fecha.
3. **Fecha inválida**
   - Envia un formato incorrecto y verifica el mensaje de error (validación).
4. **Asignación completa**
   - Selecciona fecha con disponibilidad en `disponibilidad`.
   - Verifica que Motor IA sugiere tutor, estudiante confirma, se marca `Pendiente`, se registra en `tutorias`, disponibilidad pasa a `Asignada`, tutor recibe mensaje y confirma.
5. **Timeout tutor**
   - Si el tutor no responde en 1 minuto, verifica que la disponibilidad se libera y que el estudiante recibe notificación.
6. **Errores**
   - Intencionalmente quita permisos de Sheets y verifica que `Error Trigger` notifica al admin y que `error_log` se crea.

---

## 7. Recomendaciones operativas
- Cambia el timeout de confirmación a 15–30 minutos para producción (1 minuto es útil para pruebas rápidas).
- Habilita logs y monitorización en n8n para capturar errores.
- No compartas públicamente los tokens o keys; si se exponen, rótalas.
- Considera usar batchUpdate en Google Sheets para operaciones de gran volumen.

---

## 8. Cómo proceder si quieres que lo integre totalmente
Puedo:
- A) Añadir el nodo lookup automático para obtener `telegram_user` del tutor (por id_tutor) y conectar la notificación del tutor — sin incrustar secretos.
- B) Proporcionarte los snippets de Code adicionales para copiar/pegar.
- C) Guiarte paso a paso para crear las credenciales en tu instancia n8n.

Escribe "A" para que actualice el JSON integrando el lookup tutor (volveré a entregar JSON actualizado, sin secretos) o elige B/C según prefieras.

---

## 9. Notas de seguridad
- El JSON no contiene tus tokens ni claves. Colócalas en las credenciales de n8n.
- Si ya compartiste claves públicamente, rótalas antes de poner el bot en producción.

---
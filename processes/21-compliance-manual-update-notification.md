# Notificación de Actualización del Manual de Compliance

## Business Purpose
Enviar un correo notificando a los destinatarios configurados que el manual de Compliance requiere revisión después de una actualización.

## Trigger / Frequency
- Trigger: Un workflow de GitHub o un llamador autenticado envía `POST /actions/send_compliance_manual_update_email`.
- Frequency: Después de cambios en `controls/` o `processes/`, y bajo demanda.

## Sistemas involucrados
- `agm-api`
- Conector de Gmail
- Plantilla HTML de actualización del manual de Compliance
- Workflow de GitHub en `.github/workflows/send_compliance_manual_update_email.yml`

## Roles / responsables
- Responsable principal: Andres Aguilar
- Responsable de respaldo: Hernan Castro
- Supervisión ejecutiva: Hernan Castro

## Entradas / prerrequisitos
- Credenciales de repositorio `AGM_API_EMAIL` / `AGM_API_PASSWORD` para la identidad existente `github@agmtechnology.com`; token obtenido mediante `POST /token`. Se verificó el permiso adicional `actions/send_compliance_manual_update_email` sin cambiar la contraseña ni los permisos anteriores. El repositorio compliance aún requiere acceso a la misma contraseña configurada para agm-api.
- Credenciales y conector de Gmail operativos
- Plantilla `compliance_manual_update.html`
- Contexto opcional del Pull Request: `pr_url`, título, commit y repositorio

## Flujo paso a paso
1. El workflow responde a Pull Requests abiertos o reabiertos que modifican controls, processes, Markdown o CSV. Valida las credenciales de repositorio, solicita un token y llama al endpoint. Un PR de fork sin secretos falla explícitamente; no se cambia a pull_request_target.
2. Cuando el evento es un Pull Request, el workflow envía su URL y metadatos básicos; en otros eventos el contexto puede estar vacío.
3. Los destinatarios actuales son aa@agmtechnology.com, cr@agmtechnology.com, hc@agmtechnology.com y as@agmtechnology.com. El servicio filtra el contexto permitido y el conector de Gmail envía el correo con asunto `Compliance Manual Update Requires Review` usando la plantilla de actualización.
4. Si existe `pr_url`, la plantilla incluye un enlace directo al Pull Request; si no existe, el correo no muestra botón.
5. La ruta devuelve `sent`, los destinatarios y el resultado del conector de Gmail.

## Salidas / registros creados
- Correo de notificación a los destinatarios configurados
- API response con estado de envío y resultado del conector
- El historial de revisión y aprobación queda en GitHub cuando se utiliza un Pull Request
- No se crea un registro separado de acuse dentro del API

## Excepciones / manejo de errores
- Un error de Gmail o de la plantilla devuelve un error de servicio mediante el manejo estándar del API.
- El workflow falla ante credenciales ausentes, fallo de transporte/autenticación, HTTP no 200 o ausencia de confirmación `status=sent` con ID de Gmail; publica un resumen sin credenciales ni cuerpos de error. No implementa reintentos, escalamiento ni recordatorios posteriores. Un resultado incierto exige revisar evidencia antes de repetir.
- El endpoint puede invocarse independientemente de un cambio documental real.

## Controles / puntos de verificación
- Control detectivo: cuando la invocación es exitosa, el endpoint produce un correo de notificación.
- Para eventos de Pull Request, la notificación queda vinculada al PR mediante `pr_url`.
- La aprobación y el merge se registran en el historial del Pull Request de GitHub.

## Evidencia a conservar
- Logs de solicitud y respuesta del API
- Correo de Gmail enviado
- Pull Request, revisión aprobatoria y registro de merge, cuando aplique

## Código / páginas / rutas relacionadas
- Entry surfaces: `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/tools/private/actions.py`, `agm-api/src/components/tools/public/email.py`
- Downstream side effects: `agm-api/src/lib/email_templates/compliance_manual_update.html`

## Última revisión
- Estado: borrador
- Fecha: 2026-11-19
- Revisor: reparación local del workflow; identidad GitHub existente habilitada; secreto de contraseña en compliance y autenticación real pendientes

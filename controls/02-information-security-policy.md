# Política de Seguridad de la Información

## Propósito

Establecer los requisitos mínimos de seguridad para proteger los datos, sistemas, operaciones e integraciones con terceros de AGM.

## Alcance

Aplica a empleados, contratistas, sistemas, aplicaciones, infraestructura, datos y servicios externos que apoyan las operaciones de AGM, incluidas las integraciones con IBKR e Interclear.

## Titularidad y aprobación

- Responsable de la política: Chief Compliance Officer.
- Aprobador ejecutivo: Junta Directiva.
- Método de aprobación: registro de aprobación y merge en el repositorio GitHub de Compliance de AGM, respaldado por el registro de comunicación de la Junta Directiva.

## Declaraciones de la política

1. El acceso se basa en mínimo privilegio, identidades autenticadas, roles definidos y segregación de funciones. Las cuentas compartidas están prohibidas salvo aprobación expresa.
2. El acceso administrativo y privilegiado utiliza controles de autenticación reforzada cuando están técnicamente disponibles.
3. Los cambios de producción se documentan, revisan y aprueban antes de su implementación. Los cambios de emergencia se documentan después de su implementación.
4. Las actividades relevantes para la seguridad se registran cuando es técnicamente viable y se conservan para revisión o investigación.
5. Los datos sensibles se protegen durante el almacenamiento y la transmisión mediante controles apropiados de la industria cuando están disponibles.
6. Los incidentes de seguridad o sospechas de incidentes se reportan oportunamente, se documentan, se escalan y se siguen hasta su cierre.
7. El acceso de terceros requiere aprobación expresa, alcance definido, requisitos de seguridad apropiados y revisión o revocación cuando deja de ser necesario.
8. El personal con acceso a sistemas recibe orientación de seguridad adecuada a su rol.
9. Los servicios cloud utilizan prácticas de configuración segura: solo se habilitan las APIs y servicios requeridos, los cambios de configuración se restringen al personal autorizado de desarrollo o administración y las credenciales o secretos se almacenan en servicios administrados como Google Secret Manager.
10. AGM realiza pruebas internas de vulnerabilidades de las aplicaciones relevantes aproximadamente una vez al año utilizando OWASP ZAP y monitorea los logs cloud y de aplicaciones disponibles para identificar indicios de debilidades de seguridad. Los parches de seguridad se aplican cuando se identifica una vulnerabilidad o defecto de seguridad. El objetivo operativo actual de AGM es atender los problemas identificados en aproximadamente uno o dos días; queda pendiente documentar un SLA formal basado en severidad.
11. Los incidentes de seguridad sospechados o confirmados se gestionan conforme a `compliance/controls/04-incident-response-policy.md`.
12. Los eventos relevantes para la seguridad de los servicios de AGM se envían a Google Cloud Logging y se revisan mediante Google Cloud Error Reporting, análisis automatizado diario e investigación manual cuando es necesario. Google Workspace proporciona alertas de seguridad, incluidas notificaciones de inicios de sesión sospechosos.

## Inventario de activos críticos

AGM mantiene un inventario de hardware, software y servicios críticos en `compliance/controls/logs/critical-asset-inventory.csv`. El inventario y el registro oficial de proveedores terceros identifican el activo o servicio, propósito, tipo, criticidad, alcance de dependencia, proveedor principal, fecha de revisión y estado. Se revisan al menos anualmente y después de cambios materiales.

AGM utiliza la configuración predeterminada de retención de Google Cloud Logging para sus logs cloud: el bucket `_Required` conserva los logs de auditoría aplicables durante 400 días y el bucket `_Default` conserva los logs de aplicaciones durante 30 días.

## Comunicación y acuse

- La política se comunica al personal con acceso a sistemas al emitirse y después de actualizaciones materiales. El registro de comunicación se conserva con la evidencia de la política.

## Revisión de adecuación

La política se revisa al menos anualmente, después de cambios materiales de negocio o tecnología, o después de un incidente de seguridad significativo. El revisor evalúa el tamaño y personal de la compañía, la criticidad de los sistemas, la sensibilidad de los datos, las obligaciones regulatorias, la exposición al acceso de proveedores y las amenazas o incidentes actuales. La decisión, justificación, brechas y acciones se registran en `compliance/controls/logs/security-policy-review-log.csv`. La Junta Directiva revisa el resultado y aprueba las actualizaciones materiales.

## Excepciones

Las excepciones requieren justificación de negocio documentada, evaluación de riesgos, controles compensatorios, fecha de vencimiento y aprobación de la Junta Directiva.

## Evidencia mínima a conservar

- Versión vigente y aprobada de la política.
- Historial de aprobaciones y merges de GitHub.
- Registro de aprobación de la Junta Directiva y comunicación de la política.
- Comunicación de la política y revisiones de adecuación completadas.
- Registros de excepciones, si existen.

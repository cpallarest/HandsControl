# HAND’S · Revisiones de tienda

Aplicación móvil para realizar visitas a tiendas y generar un informe PDF con fotos, observaciones y acciones acordadas. El informe se envía al cliente con copia al responsable de la visita cuando el correo está configurado.

## Uso

1. Abre una tienda y empieza una revisión, o continúa un borrador.
2. Completa los datos del local y los dos correos destinatarios.
3. Recorre los 15 puntos. Cada uno explica qué observar y cómo comprobarlo. Registra conforme, no conforme o no aplica, y añade notas o fotos.
4. Toda no conformidad requiere descripción, acción, responsable y fecha límite. No aplica requiere un motivo. Los puntos pendientes impiden finalizar.
5. Completa la conclusión y el próximo seguimiento, revisa el PDF y confirma los destinatarios.
6. Finaliza. El PDF queda guardado e inmutable. La app distingue entre informe generado, envío pendiente, resultado incierto y envío aceptado por el proveedor.

Incluye Salera, Villarreal, La Vall, Moncofar y Salou. Las fichas se guardan automáticamente con conexión. Una visita no finalizada se puede continuar desde otro dispositivo con la misma cuenta. No incorpora ventas ni cobros.

## Estado de la primera versión

- Base de datos propia y archivos privados. Los datos de Airtable no se sincronizan en esta versión.
- Envío implementado con Resend; requiere activar las variables de entorno y verificar un remitente. Sin configuración se puede generar y descargar el PDF, pero el correo queda pendiente y no se envía automáticamente al configurar el proveedor: hay que pulsar **Enviar informe**.
- Requiere conexión. No es una aplicación con funcionamiento sin conexión. Si falla el guardado, conserva la ficha abierta, recupera conexión y pulsa Guardar.
- Máximo dos fotografías por punto. El navegador comprime a JPG y el servidor valida un límite de 600 KB por foto.
- El historial muestra las últimas 100 visitas; las anteriores permanecen guardadas en la base de datos.
- El envío aceptado por Resend no garantiza que llegue a la bandeja de entrada. No se han implementado eventos de entrega o rebote.
- Los informes cerrados no se editan. Una nueva visita permite documentar la comprobación de las acciones anteriores, cuyo estado no se actualiza automáticamente.

## Desarrollo

Node 24 y pnpm (versión declarada en `package.json`).

```sh
corepack pnpm install --frozen-lockfile
pnpm test
pnpm typecheck
pnpm build
```

El proyecto utiliza React, Vinext, Cloudflare Workers, D1, R2 y pdf-lib. La configuración de despliegue de Sites está en `.openai/hosting.json`. Las migraciones SQL y sus metadatos están en `drizzle/`. El despliegue aplica las migraciones pendientes antes de activar la aplicación. No cambiar migraciones ya aplicadas: generar una nueva.

## Correo

Configurar secretos en el entorno del servidor, nunca en GitHub ni en el navegador:

- `RESEND_API_KEY`: clave con permiso de envío.
- `REPORT_FROM`: remitente autorizado en el dominio verificado de Resend, por ejemplo `HAND’S <revisiones@dominio-verificado>`.

La primera prueba real debe dirigirse a dos buzones de prueba controlados por el propietario. Verificar destinatarios, PDF adjunto, recepción y estado antes del uso con clientes. Las pruebas automáticas simulan el proveedor y no envían correos.

Cada intento tiene una clave de idempotencia y una reserva exclusiva en la base de datos. Ante un resultado incierto, el reintento conserva destinatarios, remitente, contenido y PDF. La app bloquea el reintento automático a partir de 23 horas: hay que comprobar el registro del proveedor para decidir una recuperación sin duplicados. Esta versión no tiene un proceso de envíos en segundo plano.

## Acceso y privacidad

El despliegue está pensado para Sites con acceso privado e identidad de ChatGPT. Cada consulta y modificación comprueba el propietario de la visita; las fotos y PDF requieren autenticación. Las escrituras también comprueban el origen de la petición. El guardado usa versiones para detectar cambios concurrentes.

`app/chatgpt-auth.ts` confía en las cabeceras de identidad verificadas por Sites. Si se aloja fuera de Sites, es necesario sustituir esa integración por autenticación verificada del servidor y configurar sus recursos D1/R2 equivalentes; no exponer el Worker directamente confiando en cabeceras aportadas por el cliente.

No subir claves, correos de clientes, fotografías reales ni exportaciones de la base de datos al repositorio. La imagen de `tests/fixtures/` es una muestra artificial sin datos de clientes.

## Verificación

`pnpm test` comprueba validación de fichas, PDF con fotos y caracteres especiales, control de versiones y propietario, destinatarios, reintentos, concurrencia e incertidumbre de los envíos. Las pruebas generan `qa/informe-prueba.pdf`, excluido del repositorio.

Antes de usar con clientes, comprobar en un móvil real: captura de cámara, regreso a un borrador, pérdida y recuperación de conexión, revisión del PDF y recepción del correo. El código y la compilación no sustituyen esa comprobación de uso real.

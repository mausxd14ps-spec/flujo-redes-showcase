# Flujo redes: panel para publicar Reels en Instagram

Panel privado para publicar contenido en redes sociales sin abrir cada app. Subes un video, escribes (o dictas) la descripción y eliges si publicar ahora o programar. Las publicaciones programadas salen solas, aunque mi computadora esté apagada.

> Proyecto personal de aprendizaje, desplegado en producción y usado en mi propia cuenta. Hoy solo **Instagram Reels** está conectado y probado.

<img width="1902" height="865" alt="image" src="https://github.com/user-attachments/assets/3b19d160-9e94-453a-b97c-b09f45a4a044" />
<img width="1896" height="857" alt="image" src="https://github.com/user-attachments/assets/4d46fdbd-e55c-404b-886b-7b9e43736495" />
<img width="1896" height="861" alt="image" src="https://github.com/user-attachments/assets/59380730-d3d7-4b91-b60b-fbf969445cb0" />




## Qué hace

- Sube videos o imágenes directo desde el navegador a un almacenamiento en la nube.
- Publica **ahora** o **programa** para una fecha y hora.
- Las programadas se publican solas gracias a un cron externo.
- Dictado de la descripción por voz (reconocimiento de voz del navegador, gratis).
- Historial de publicaciones con estado y resultado de cada intento.
- Botón de borrar que también elimina el archivo del almacenamiento para no gastar espacio.
- Acceso protegido con usuario y contraseña.

## Tecnologías

| Área | Herramienta |
| --- | --- |
| Framework | Next.js 16 (App Router) con TypeScript |
| Base de datos | PostgreSQL en Neon, con Drizzle ORM |
| Hosting | Vercel |
| Archivos | Vercel Blob (subida directa desde el navegador) |
| Redes | Meta Graph API (Instagram) |
| Programación | cron-job.org llamando a una ruta protegida |
| Voz | Web Speech API |

## Problemas técnicos que resolví

**1. Error 2207076 al publicar Reels.** La forma habitual (darle a Instagram la URL del video para que lo descargue) fallaba con ese error, incluso con videos pequeños y válidos. Lo resolví cambiando a **subida resumable**: el servidor crea el contenedor con `upload_type=resumable` y envía el archivo directamente al servidor de subida de Meta. También hice que el panel muestre el motivo real del error que devuelve Instagram, no un mensaje genérico.

**2. Tokens que caducaban en horas.** El token que devuelve el inicio de sesión es de corta duración. Agregué el intercambio por un token de larga duración antes de pedir las páginas, para que la conexión no se rompa a las pocas horas.

**3. Error 500 al conectar la cuenta.** Se debía a que `Response.redirect()` crea respuestas con encabezados inmutables, y el código intentaba agregarle cookies. Lo cambié por una respuesta 302 construida a mano.

**4. Límite de 4,5 MB en las funciones de Vercel.** Los videos no caben en el cuerpo de una petición. La solución fue subir el archivo **directo del navegador a Vercel Blob** y enviar al servidor solo la URL, que el servidor valida (solo acepta direcciones del propio almacén).

**5. Programar publicaciones sin servidor propio.** El plan gratuito de Vercel solo permite cron nativo una vez al día. Usé un cron externo que cada 5 minutos llama a una ruta que revisa qué publicaciones ya cumplieron su hora. Esa ruta acepta un secreto propio, y el resto del sitio queda protegido por contraseña.

**6. Bug de zona horaria.** El navegador enviaba la hora sin zona horaria y el servidor (en UTC) la interpretaba con 4 horas de diferencia, así que una programada se publicaba al instante. Lo corregí enviando la fecha en formato UTC y mostrando las fechas en la zona horaria del navegador. Además, el formulario avisa si la hora elegida ya pasó.

## Seguridad

- Sitio completo detrás de autenticación básica; en producción se bloquea si faltan las credenciales.
- Claves y tokens solo en variables de entorno, nunca en el repositorio.
- La ruta que recibe archivos valida el origen de la URL antes de guardarla.
- Verificación en dos pasos en las cuentas de Meta.

## Limitaciones conocidas

- Solo Instagram está conectado y probado. YouTube, TikTok, X, LinkedIn y Facebook existen en el código, pero no tienen claves configuradas.
- La app de Meta está en modo desarrollo, suficiente para uso personal. Para que otras personas la usen haría falta pasar la revisión de Meta.
- Las publicaciones programadas salen con hasta 5 minutos de retraso, por el intervalo del cron.

## Qué aprendí

Leer los errores reales de una API externa en vez de adivinar, diseñar alrededor de los límites de una plataforma serverless, y cuidar la seguridad desde el principio (claves, accesos y verificación en dos pasos).


## Contacto
Gabriel · Isla de Margarita, Venezuela
Instagram: @tu_usuario · Correo: tu_correo_de_trabajo

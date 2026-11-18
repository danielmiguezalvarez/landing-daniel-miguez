# Infraestructura de DM Integra

Este documento describe la infraestructura conocida del sitio y distingue la configuración comprobada en el repositorio de los datos que deben confirmarse en los paneles de cada proveedor. No contiene credenciales.

## Tecnologías del proyecto

Comprobado en `package.json`, `astro.config.mjs` y el código fuente:

- **Astro 7**: framework del sitio. Las páginas y componentes están en `src/`.
- **TypeScript**: configuración estricta heredada de Astro (`tsconfig.json`) y datos de contacto en TypeScript.
- **Tailwind CSS 4**: integrado mediante el plugin de Vite `@tailwindcss/vite`.
- **Node.js 22.23.4 o posterior**: versión mínima declarada en `package.json`.
- **npm**: hay un `package-lock.json`; usar `npm ci` para instalar las dependencias de forma reproducible.
- **Salida estática**: no hay adaptador de despliegue ni configuración de servidor en `astro.config.mjs`. Astro genera el sitio en `dist/` con `npm run build`.

No se han encontrado en el repositorio SDKs ni integración de Cloudflare, correo transaccional o backend propio. El formulario envía directamente a la API de Web3Forms.

## Proveedores y paneles

| Servicio | Uso conocido | Panel o acceso |
| --- | --- | --- |
| IONOS | Registro del dominio `dmintegra.es` | [login.ionos.com](https://login.ionos.com/) |
| Cloudflare | DNS, hosting en Cloudflare Pages y reenvío de correo de Cloudflare Email Routing | [Panel de Cloudflare](https://dash.cloudflare.com/) |
| GitHub | Repositorio y origen del código | [Repositorio del proyecto](https://github.com/danielmiguezalvarez/landing-daniel-miguez) |
| Gmail | Destino del reenvío de `contacto@dmintegra.es` | [Gmail](https://mail.google.com/) |
| Web3Forms | Recepción de envíos del formulario de contacto | [Web3Forms](https://web3forms.com/) |
| WhatsApp | Enlace externo voluntario desde la web; no hay widget integrado | [WhatsApp](https://wa.me/) |

DM Sans se aloja en `public/fonts/` y se configura en `src/styles/global.css`; la web ya no solicita fuentes a Google Fonts. La licencia de los archivos está en `public/fonts/OFL.txt`.

Los enlaces abren las páginas de acceso o producto; la cuenta, el proyecto y sus permisos dependen de la sesión iniciada.

## Dominio, DNS, hosting, HTTPS y correo

- **Dominio:** `dmintegra.es` ([https://dmintegra.es](https://dmintegra.es)), registrado en IONOS. URL alternativa de Pages: [https://landing-daniel-miguez.pages.dev](https://landing-daniel-miguez.pages.dev).
- **DNS:** gestionado en Cloudflare. Los registros concretos (por ejemplo, los que apuntan a Pages y los de correo) no están en el repositorio; consultar la zona DNS del dominio en el panel antes de modificarlos.
- **Hosting:** Cloudflare Pages.
- **HTTPS:** se gestiona desde Cloudflare Pages y la configuración del dominio en Cloudflare. Deben verificarse allí el dominio personalizado y el estado del certificado; no se han versionado esos datos.
- **Correo:** `contacto@dmintegra.es` se reenvía mediante Cloudflare Email Routing a Gmail. El destino y los registros DNS de correo deben comprobarse en Cloudflare. No consta un servicio de buzones alojados ni de envío de correo saliente.

## Despliegue desde GitHub

El repositorio de código es [danielmiguezalvarez/landing-daniel-miguez](https://github.com/danielmiguezalvarez/landing-daniel-miguez). El proyecto Cloudflare Pages `landing-daniel-miguez` está conectado a ese repositorio y realiza despliegues automáticos desde `main`. No hay un workflow de GitHub Actions, archivo `wrangler.toml` ni configuración del proyecto Pages versionada; los ajustes de Pages se gestionan en el panel de Cloudflare.

Para este proyecto, Pages ejecuta `npm run build` y publica la salida de `dist/`. La configuración de previews para pull requests y el estado del último despliegue se consultan en el panel de Cloudflare Pages.

Para comprobar un despliegue, revisar en Pages el commit y resultado de la compilación, y después abrir el dominio de producción. Si el dominio no carga, confirmar también el dominio personalizado, DNS y HTTPS.

## Ejecutar y compilar localmente

Requisitos: Node.js `22.23.4` o posterior y npm. Desde la raíz del repositorio:

```sh
npm ci
npm run dev
```

Astro muestra la URL local en la terminal (por defecto, `http://localhost:4321`). Para generar y previsualizar la versión de producción:

```sh
npm run build
npm run preview
```

La compilación estática se genera en `dist/`. `PUBLIC_WEB3FORMS_ACCESS_KEY` es necesaria para que el formulario funcione, tanto localmente como al compilar para Cloudflare Pages; la sección «Variables de entorno» explica dónde configurarla.

## Servicios gratuitos y costes conocidos

- Astro, Tailwind CSS y sus paquetes están declarados como dependencias del proyecto; el repositorio no registra pagos por software.
- No hay facturas, plan contratado ni datos de consumo en el repositorio. Por tanto, **no se puede confirmar el coste actual** del dominio, Cloudflare, correo u otros servicios.
- El plan y cualquier cargo de IONOS y Cloudflare, así como la renovación del dominio y límites de uso, deben verificarse en las respectivas cuentas.
- Web3Forms está integrado en el formulario. El plan, los límites y cualquier coste aplicable deben comprobarse en la cuenta de Web3Forms; no constan en el repositorio.

## Formulario de contacto y Web3Forms

El formulario de `src/components/Contact.astro` envía los campos obligatorios mediante `POST` a `https://api.web3forms.com/submit`. Valida nombre, email, tipo de proyecto y descripción; muestra estados de envío, éxito y error, y solo limpia los campos después de una respuesta correcta. Incluye el campo honeypot `botcheck`.

La Access Key se administra desde la cuenta de [Web3Forms](https://web3forms.com/). Su valor se lee de `PUBLIC_WEB3FORMS_ACCESS_KEY` durante la compilación; no se guarda en el código versionado. Configura esa variable:

- **Localmente:** en `.env` en la raíz del proyecto. Este archivo está excluido de Git.
- **Cloudflare Pages:** en el panel del proyecto Pages, en la configuración de variables de entorno para los entornos de producción y preview que se utilicen. Vuelve a desplegar después de añadir o cambiar la variable para que Astro la incorpore a la compilación estática.

La Access Key se incluye en el HTML generado y es visible desde el navegador; se usa como clave de acceso pública del formulario, no como contraseña o clave privada. El destinatario de notificaciones indicado para el formulario es `contacto@dmintegra.es`; comprueba su configuración efectiva en la cuenta de Web3Forms. El reenvío de esa dirección se configura en Cloudflare Email Routing y su destino se comprueba allí. La dirección final de recepción y el funcionamiento del reenvío deben verificarse en sus respectivos paneles.

El honeypot está incluido a petición del proyecto. La documentación de Web3Forms lo marca como obsoleto y menos eficaz que CAPTCHA; si el formulario recibe spam, revisar las opciones antispam actuales del proveedor.

## Privacidad, retención y cookies

- El formulario recoge nombre, email, tipo de proyecto y descripción. Los datos se envían desde el navegador a `https://api.web3forms.com/submit` para gestionar y responder consultas; no se usan para publicidad ni newsletters.
- Según la configuración comunicada para Web3Forms, los envíos se eliminan automáticamente a los 30 días. Confirmar esta opción y sus condiciones en la cuenta de Web3Forms.
- Las consultas recibidas por email que no terminan en contratación se eliminarán de Gmail a los 3 meses desde la última comunicación. Este plazo es la regla de retención indicada; no hay en el repositorio una configuración o automatización de borrado de Gmail. Verificar cómo se aplica en la cuenta.
- El correo profesional `contacto@dmintegra.es` se reenvía mediante Cloudflare Email Routing a Gmail. Email Routing, destino y DNS se gestionan en el panel de Cloudflare; el contenido recibido se consulta y elimina en Gmail.
- No se ha encontrado código de analítica, publicidad, píxeles de seguimiento, cookies propias ni uso de `localStorage` o `sessionStorage`. La comprobación puntual de `dmintegra.es` del 18/11/2026 no observó cookies accesibles desde la página, almacenamiento local/de sesión ni `Set-Cookie` en la respuesta de inicio. No descarta cookies HttpOnly, de terceros, las que pudiera crear una interacción con servicios externos o ajustes de Cloudflare.
- No se añade banner de cookies hasta verificar la configuración real de producción y los servicios externos. La revisión de cookies no constituye una conclusión sobre qué cookies, si las hubiera, requieren consentimiento.
- La política pública está en `src/pages/privacidad.astro`; su primera capa está junto al formulario en `src/components/Contact.astro`. La página `/aviso-legal` está en `src/pages/aviso-legal.astro`.

No asumir roles contractuales, plazos distintos de los comunicados, ubicaciones ni transferencias internacionales sin comprobar las condiciones y los ajustes de Web3Forms, Cloudflare y Google/Gmail.

### Bases jurídicas y ponderación del interés legítimo

- Las solicitudes de presupuesto y medidas precontractuales se tratan, cuando corresponda, bajo el artículo 6.1.b del RGPD.
- Las consultas generales iniciadas voluntariamente por la persona se limitan a recibir y responder el mensaje, bajo el interés legítimo del artículo 6.1.f. No se usan para publicidad ni para comunicaciones ajenas a la consulta.
- **Ponderación breve:** el interés perseguido es contestar el mensaje que la propia persona envía. El email y el contenido de la consulta permiten responder; el nombre y el tipo de proyecto aportan contexto para entenderla. Quien completa voluntariamente el formulario puede esperar una respuesta relacionada con su consulta. El tratamiento no incluye publicidad ni seguimiento ajeno, y las consultas sin contratación se eliminan de Gmail a los tres meses de la última comunicación. La persona puede ejercer sus derechos, incluida la oposición al tratamiento basado en interés legítimo.
- Con este alcance limitado, la ponderación respalda el interés legítimo para responder consultas generales. Revaluar si cambian los campos, los usos o los plazos; la revisión de condiciones y ubicaciones de proveedores sigue pendiente.

### Identificación legal

El aviso legal publica los datos facilitados: Daniel Míguez Álvarez como persona física, el nombre comercial DM Integra y el correo de contacto. El responsable ha indicado que aún no está dado de alta como autónomo ni ha constituido una sociedad, y ha pedido no publicar domicilio particular ni NIF. Por ello, la identificación legal queda incompleta temporalmente; no se afirma que el aviso cumpla íntegramente la LSSI. Confirmar los datos y requisitos aplicables antes de completar esa identificación.

## Auditoría de dependencias

El `npm audit` realizado el 18/11/2026 notificó 10 entradas de severidad alta asociadas a un único aviso: [`GHSA-fc5q-39xc-vjpf`](https://github.com/advisories/GHSA-fc5q-39xc-vjpf), relacionado con un posible ReDoS en `picomatch`. El aviso figura como **retirado** desde el 18/11/2026 y no indica una versión corregida; las 10 entradas corresponden a dependencias y rutas transitivas, no a 10 avisos independientes.

El árbol señalado incluye `astro@7.4.12`, `@astrojs/internal-helpers@0.13.2`, `@astrojs/markdown-satteri@0.4.10`, `@astrojs/vite-plugin-astro@0.1.3`, `vite@8.3.8`, `tinyglobby@0.2.17`, `fdir@6.5.0`, `unstorage@1.17.6`, `anymatch@3.1.3` y `picomatch@4.0.7` / `2.3.2`. El proyecto no declara `devDependencies`, por lo que npm las clasifica en el árbol de producción; en esta web estática se usan durante la instalación y compilación, no como código ejecutado por el navegador ni como servidor Astro en Cloudflare Pages. No se encontró una vía de explotación desde visitantes; el posible impacto de denegación de servicio dependería de procesar patrones maliciosos durante una operación de build afectada.

**Decisión:** no se ejecutó ni se recomienda `npm audit fix` para este resultado. La propuesta automática observada por npm baja Astro desde la versión 7 a `astro@5.3.0`, un cambio mayor hacia una versión anterior, no una actualización compatible. Dado que el aviso está retirado y no indica una versión corregida, no se cambian dependencias; volver a evaluar si el aviso o el árbol de dependencias cambia.

## Variables de entorno

La compilación del formulario requiere `PUBLIC_WEB3FORMS_ACCESS_KEY` para enviar consultas. `.env` y `.env.production` están excluidos de Git.

La variable lleva el prefijo `PUBLIC_` porque Astro la incorpora al formulario estático para enviarla desde el navegador; por tanto, su valor no es secreto. Para producción y previews, se configura en Cloudflare Pages, en las variables de entorno de compilación del proyecto, y requiere un nuevo despliegue. No subir archivos `.env` al repositorio ni incluir el valor de la clave en documentación o capturas.

## Mantenimiento y resolución de problemas

- **Cambios de contenido o código:** trabajar en una rama, ejecutar `npm ci` y `npm run build`, y después integrar los cambios mediante GitHub. Revisar la configuración real de despliegue de Pages antes de depender de una rama o flujo de previews.
- **Fallo de compilación:** consultar el log de Pages para el despliegue y reproducir localmente con `npm ci` y `npm run build`. Comprobar que la versión de Node cumple el mínimo de `package.json`.
- **Sitio no actualizado:** confirmar que el commit llegó a la rama configurada como producción y que el despliegue correspondiente terminó correctamente en Pages.
- **Dominio o HTTPS con problemas:** revisar en Cloudflare Pages el dominio personalizado y el certificado; después comprobar que los registros DNS de Cloudflare coinciden con la configuración que muestra Pages. No modificar registros a ciegas.
- **Correo no recibido:** revisar el estado de Email Routing, el destino de reenvío y los registros DNS de correo en Cloudflare; después comprobar Gmail, incluidos sus filtros y spam. El repositorio no permite verificar estos ajustes.
- **Formulario:** verificar que `PUBLIC_WEB3FORMS_ACCESS_KEY` está configurada en el entorno de compilación, que el despliegue usa el valor actualizado y que el destinatario configurado en Web3Forms recibe el mensaje. La clave puede ser visible en el sitio publicado.

No guardar credenciales, claves de acceso, datos personales ni información de recuperación en este documento o en el repositorio.

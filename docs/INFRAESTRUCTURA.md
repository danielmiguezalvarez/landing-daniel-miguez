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

Los enlaces abren las páginas de acceso o producto; la cuenta, el proyecto y sus permisos dependen de la sesión iniciada.

## Dominio, DNS, hosting, HTTPS y correo

- **Dominio:** `dmintegra.es`, registrado en IONOS.
- **DNS:** gestionado en Cloudflare. Los registros concretos (por ejemplo, los que apuntan a Pages y los de correo) no están en el repositorio; consultar la zona DNS del dominio en el panel antes de modificarlos.
- **Hosting:** Cloudflare Pages. El nombre del proyecto Pages y su configuración actual no constan en el repositorio.
- **HTTPS:** se gestiona desde Cloudflare Pages y la configuración del dominio en Cloudflare. Deben verificarse allí el dominio personalizado y el estado del certificado; no se han versionado esos datos.
- **Correo:** `contacto@dmintegra.es` se reenvía mediante Cloudflare Email Routing a Gmail. El destino y los registros DNS de correo deben comprobarse en Cloudflare. No consta un servicio de buzones alojados ni de envío de correo saliente.

## Despliegue desde GitHub

El repositorio de código es [danielmiguezalvarez/landing-daniel-miguez](https://github.com/danielmiguezalvarez/landing-daniel-miguez) y el hosting indicado es Cloudflare Pages. No hay un workflow de GitHub Actions, archivo `wrangler.toml` ni configuración del proyecto Pages versionada.

En un despliegue conectado a GitHub, Pages obtiene los cambios de la rama configurada, ejecuta el comando de compilación y publica los archivos generados. Para este proyecto, los valores coherentes con el código son `npm run build` y `dist/`. La rama de producción, la conexión activa con GitHub, la creación de previews para pull requests y el estado del último despliegue **deben confirmarse en el panel de Cloudflare Pages**; no se deducen del repositorio.

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

La compilación estática se genera en `dist/`. No es necesario configurar variables de entorno para la aplicación actual.

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

La Access Key se incluye en el HTML generado y es visible desde el navegador; se usa como clave de acceso pública del formulario, no como contraseña o clave privada. El destinatario de las notificaciones debe verificarse en la cuenta de Web3Forms; no consta si actualmente es `contacto@dmintegra.es`. Comprueba también que el reenvío de Cloudflare Email Routing a Gmail funciona.

El honeypot está incluido a petición del proyecto. La documentación de Web3Forms lo marca como obsoleto y menos eficaz que CAPTCHA; si el formulario recibe spam, revisar las opciones antispam actuales del proveedor. Revisar la información de privacidad y los avisos aplicables para los datos enviados antes de publicar.

## Variables de entorno

La aplicación requiere `PUBLIC_WEB3FORMS_ACCESS_KEY` para que el formulario pueda enviar consultas. `.env` y `.env.production` están excluidos de Git.

La variable lleva el prefijo `PUBLIC_` porque Astro la incorpora al formulario estático para enviarla desde el navegador; por tanto, su valor no es secreto. No subir archivos `.env` al repositorio ni incluir el valor de la clave en documentación o capturas.

## Mantenimiento y resolución de problemas

- **Cambios de contenido o código:** trabajar en una rama, ejecutar `npm ci` y `npm run build`, y después integrar los cambios mediante GitHub. Revisar la configuración real de despliegue de Pages antes de depender de una rama o flujo de previews.
- **Fallo de compilación:** consultar el log de Pages para el despliegue y reproducir localmente con `npm ci` y `npm run build`. Comprobar que la versión de Node cumple el mínimo de `package.json`.
- **Sitio no actualizado:** confirmar que el commit llegó a la rama configurada como producción y que el despliegue correspondiente terminó correctamente en Pages.
- **Dominio o HTTPS con problemas:** revisar en Cloudflare Pages el dominio personalizado y el certificado; después comprobar que los registros DNS de Cloudflare coinciden con la configuración que muestra Pages. No modificar registros a ciegas.
- **Correo no recibido:** revisar el estado de Email Routing, el destino de reenvío y los registros DNS de correo en Cloudflare; después comprobar Gmail, incluidos sus filtros y spam. El repositorio no permite verificar estos ajustes.
- **Formulario:** verificar que `PUBLIC_WEB3FORMS_ACCESS_KEY` está configurada en el entorno de compilación, que el despliegue usa el valor actualizado y que el destinatario configurado en Web3Forms recibe el mensaje. La clave puede ser visible en el sitio publicado.

No guardar credenciales, claves de acceso, datos personales ni información de recuperación en este documento o en el repositorio.

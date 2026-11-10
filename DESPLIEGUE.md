# Despliegue de Jach’a Puno

Copia del repositorio, revisión 7f083df. El sitio está listo en `dist/`.
HTML, CSS y JavaScript nativo, con Urbanist, imágenes, QR y documentos locales.
El README original conserva notas históricas. Las tarifas vigentes en esta copia
están en `dist/app.js`: S/ 250, 450, 650 y 900.

## Probar localmente

Desde la carpeta del ZIP ya extraído, ejecuta `python3 -m http.server 8000 --directory dist`
y abre http://localhost:8000. Python solo se usa para esta prueba opcional.

## Publicar en una VPS

1. Instala Caddy en una VPS Linux con IP pública. Para tráfico inicial, 1 vCPU,
   512 MB de RAM y 5 GB de disco son un punto de partida orientativo. Si tu proveedor
   ofrece 1 GB de RAM, tendrás más margen para el sistema y sus actualizaciones.
2. En tu proveedor DNS, crea un registro A para el dominio o subdominio elegido,
   apuntando a la IPv4 de la VPS. Usa AAAA solo si tienes IPv6 configurado.
   Permite conexiones entrantes TCP a los puertos 80 y 443.
3. Sube el ZIP, extráelo y, desde la carpeta extraída, ejecuta:

       sudo mkdir -p /var/www/jacha-puno
       sudo cp -a dist/. /var/www/jacha-puno/

4. En una instalación nueva de Caddy, usa el bloque de `Caddyfile.ejemplo`
   en `/etc/caddy/Caddyfile`, cambiando el dominio de ejemplo por el tuyo.
   Si ya alojas otros sitios en Caddy, agrega el bloque a la configuración existente.
   Valida y recarga:

       sudo caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile
       sudo systemctl reload caddy

5. Abre tu dominio por HTTPS y comprueba el visor, las descargas, el QR y los
   botones de WhatsApp desde un celular.

Con hosting estático administrado basta subir el contenido de `dist/` a la
raíz pública y conectar tu dominio siguiendo las indicaciones del proveedor.

Publica únicamente `dist/`. El resto del ZIP se conserva para referencia.
No se requiere Node.js, PHP, Docker ni base de datos en producción. Los
formularios preparan mensajes externos y el QR se usa para pagar en Yape.
El portal no almacena pedidos ni verifica pagos automáticamente. El tráfico
de los PDF es el principal factor al dimensionar la cuota de transferencia.

---

## ⚠️ DESPLIEGUE EN DOKPLOY ⚠️

> **Esta sección es exclusivamente para publicar el sitio en DOKPLOY.**
> Sustituye a los pasos de Caddy de arriba: en Dokploy **no instales Caddy**,
> porque Dokploy ya trae Traefik, que gestiona el dominio y el HTTPS.

### Revisión previa del código

- Sitio 100 % estático (`dist/index.html`, `app.js`, `styles.css` y `assets/`).
  Sin backend, base de datos ni compilación. `node --check dist/app.js` pasa sin errores.
- Todas las imágenes, PDF y DOCX referenciados existen en `dist/assets/`.
- Los formularios solo abren WhatsApp (`wa.me`), `mailto:` y `tel:`; no hay que
  configurar variables de entorno ni secretos.
- `.openai/hosting.json` es del hosting de OpenAI y Dokploy lo ignora.
- `dist/` pesa unos 50 MB. Hay archivos que no usa ninguna página y puedes borrar
  para aligerar la imagen: `muestra-jacha-puno.pdf` (6 MB), `muestra-revisada.pdf`
  (2,8 MB), `modalidades-patrocinio.docx`, `presupuesto-y-anexos.docx` y
  `lago-panoramica-barra.webp`. Conserva `OFL-Urbanist.txt` (licencia de la fuente).

### 1. Añadir dos archivos en la raíz del proyecto

`Dockerfile`:

```dockerfile
FROM nginx:1.30-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY dist/ /usr/share/nginx/html/
EXPOSE 80
```

`nginx.conf`:

```nginx
server {
    listen 80;
    server_name _;
    server_tokens off;
    root /usr/share/nginx/html;
    index index.html;

    gzip on;
    gzip_types text/css application/javascript image/svg+xml;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # app.js y styles.css no llevan hash en el nombre: caché corta
    location ~* \.(css|js)$ {
        add_header Cache-Control "public, max-age=3600";
    }

    # Imágenes, fuente y documentos: caché larga
    location /assets/ {
        add_header Cache-Control "public, max-age=604800";
    }
}
```

> Rama estable de nginx (1.30.x): al reconstruir recibe los parches de
> seguridad de esa rama. `server_tokens off` oculta la versión exacta.
>
> Ya creados en el repositorio, junto con un `.dockerignore` que deja fuera de la
> imagen los archivos sin uso listados arriba (siguen en el repo, no se borran).

Prueba opcional en local: `docker build -t jacha-puno . && docker run --rm -p 8080:80 jacha-puno`
y abre http://localhost:8080.

### 2. Subir el código a Dokploy

Opción A (recomendada): repositorio Git.

    git init && git add . && git commit -m "Jach'a Puno"
    git remote add origin <URL de tu repositorio en GitHub/GitLab/Gitea>
    git push -u origin main

Opción B: sin Git, comprime en un ZIP la raíz del proyecto (con `Dockerfile`,
`nginx.conf` y `dist/`) para subirlo con el proveedor **Drop** de Dokploy.

### 3. Crear la aplicación en Dokploy

1. En el panel de Dokploy: **Projects → Create Project** (por ejemplo, `jacha-puno`).
2. Dentro del proyecto: **Create Service → Application**.
3. Pestaña **General → Provider**:
   - Opción A: elige GitHub/GitLab/Gitea/Git, el repositorio y la rama `main`.
   - Opción B: elige **Drop** y arrastra el ZIP.
4. **Build Type: Dockerfile**, con *Docker File* = `Dockerfile` y
   *Docker Context Path* = `.` (raíz).
5. Pulsa **Save** y luego **Deploy**. Revisa la pestaña **Logs / Deployments**
   hasta que termine bien.

> Alternativa sin Dockerfile: **Build Type: Static** con *Publish Directory* =
> `dist`. Funciona, pero pierdes el control de caché y compresión de `nginx.conf`.

### 4. Dominio y HTTPS

1. En tu proveedor DNS, crea un registro **A** del dominio o subdominio
   (por ejemplo, `revista.tudominio.pe`) apuntando a la IP del servidor Dokploy.
2. En la aplicación: pestaña **Domains → Add Domain**:
   - *Host*: `revista.tudominio.pe`
   - *Path*: `/`
   - *Container Port*: **80**
   - *HTTPS*: activado, *Certificate*: **Let's Encrypt**
3. Guarda y, si hace falta, vuelve a pulsar **Deploy**. El certificado puede
   tardar unos minutos tras propagarse el DNS.
4. El servidor debe tener abiertos los puertos 80 y 443 (los usa Traefik).

### 5. Comprobación y actualizaciones

- Abre el dominio por HTTPS desde un celular y prueba el visor, las descargas,
  el QR de Yape y los botones de WhatsApp.
- Para actualizar: haz `git push` y pulsa **Deploy** (o activa **Auto Deploy**
  en la pestaña General para que se despliegue con cada push). Con **Drop**,
  sube un ZIP nuevo.
- Si cambias `app.js` o `styles.css` y no ves el cambio, fuerza recarga
  (Ctrl+F5): la caché de esos archivos dura una hora.
- Recursos: el contenedor nginx usa menos de 20 MB de RAM. Lo que más cuenta es
  la transferencia por las descargas del PDF de 15 MB (`muestra-ampliada.pdf`).

# Jach'a Puno — landing estática

Sitio 100 % estático en `dist/` (`index.html`, `app.js`, `styles.css`, `assets/`).
Se despliega en Dokploy con nginx (`Dockerfile`, `nginx.conf`) al hacer push a `main`,
y el dominio totoraytinta.com pasa por **Cloudflare**, que cachea JS, CSS e imágenes.

## Regla obligatoria: versionar los archivos al modificarlos (cache busting)

Cloudflare guarda `app.js`, `styles.css` y `assets/` en su caché, pero **no** guarda
`index.html`. Si un archivo cambia y conserva la misma URL, los visitantes siguen
viendo la versión vieja aunque Dokploy haya desplegado la nueva.

Por eso, **cada vez que se modifique `dist/app.js` o `dist/styles.css`**, en el mismo
cambio hay que actualizar el parámetro `?v=` con que los carga `dist/index.html`:

```html
<link rel="stylesheet" href="styles.css?v=AAAAMMDD">
<script src="app.js?v=AAAAMMDD" defer></script>
```

- Usar la fecha del día en formato `AAAAMMDD` (p. ej. `20261118`).
- Si ya se cambió ese mismo día, agregar una letra: `20261118b`, `20261118c`…
- Actualizar ambos parámetros a la vez es lo más simple.

**Imágenes, PDF u otros archivos de `assets/`:** si se reemplaza un archivo conservando
su nombre, agregar `?v=AAAAMMDD` a cada referencia de ese archivo en `index.html` y
`app.js` (o usar un nombre nuevo). nginx les da una caché de 7 días.

Con esto no hace falta purgar la caché de Cloudflare a mano después de desplegar.

## Otras notas

- Las tarifas publicitarias vigentes están en `dist/app.js` (`adOptions`, texto de
  `renderAd`) y en el texto `fine-print` de `dist/index.html`; mantenerlas sincronizadas.
- `README.md` contiene notas históricas; las tarifas vigentes se reflejan en `DESPLIEGUE.md`.
- Validar sintaxis con `node --check dist/app.js`.
- Servidor local: `php -S localhost:8000 -t dist`.

# Giro Cero — web oficial

Landing estática EN/ES. HTML, CSS y JavaScript sin dependencias de producción, backend ni compilación. Dominio previsto: **https://girocero.es/**. Contacto público: **oier@girocero.es**.

El repositorio contiene únicamente la web y sus herramientas de mantenimiento. GitHub Pages publica **solo `dist/`**, mediante `.github/workflows/pages.yml`. La configuración de Cloudflare queda sustituida por GitHub Pages.

## Estructura

```text
.github/workflows/pages.yml    Despliegue automático
.gitattributes                Normalización de archivos
.gitignore                    Exclusiones locales
README.md                     Mantenimiento y dominio
scripts/serve.cjs              Vista previa local con Node.js
 dist/
   index.html                 Contenido inglés y metadatos
   styles.css                 Diseño responsive
   site.js                    Traducciones al español
   404.html                   Página de error
   robots.txt
   sitemap.xml
   .nojekyll
   CNAME                      Dominio de producción (referencia)
   assets/
     favicon.svg
     character-study.webp
     combat-study.webp
     prototype.webp
```

`qa/`, `scripts/check.cjs`, `CONTENT-AUDIT.md` y `FUTURE-CONTENT.md` son material local excluido del repositorio. No subir el workspace del videojuego, tokens, contraseñas ni archivos `.env`. `dist/` es el código fuente publicable; no es una carpeta generada que deba ignorarse.

## Vista previa y mantenimiento

Con Node.js instalado, desde la raíz del repositorio:

```powershell
node scripts/serve.cjs
```

Abrir http://127.0.0.1:4173/ y detener con Ctrl+C. El servidor local no simula las redirecciones ni la emisión de certificados de GitHub Pages.

Editar inglés en `dist/index.html` y español en `dist/site.js`. Los metadatos, canonical, JSON-LD, sitemap y robots apuntan a `https://girocero.es/`. La alternativa sin JavaScript sigue siendo inglesa. No se añaden cookies ni analítica.

## Despliegue automático

Configuración prevista del repositorio: `Augernwf/giro-cero`, rama `main`, Settings → Pages → Source: **GitHub Actions**.

El workflow se ejecuta al enviar cambios en `dist/` o en el propio workflow a `main`, y también desde Actions → Deploy Giro Cero to GitHub Pages → Run workflow. Comprueba los archivos principales, sube `dist/` como artefacto y lo despliega al entorno `github-pages`. No ejecuta npm ni requiere secretos propios. Los cambios que solo afecten al README no provocan un despliegue.

Antes de asociar el dominio, comprobar el primer deployment y la URL `https://Augernwf.github.io/giro-cero/`. La URL exacta debe tomarse del resultado del job.

Fuente: [publicación con GitHub Actions](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) y [workflow estático oficial](https://github.com/actions/starter-workflows/blob/main/pages/static.yml).

## Dominio y HTTPS

Tras validar la URL inicial, configurar **`girocero.es`** en Settings → Pages → Custom domain. El archivo `dist/CNAME` contendrá exactamente una línea:

```text
girocero.es
```

Con Actions, el archivo CNAME sirve como referencia de mantenimiento; el dominio se configura además en Settings → Pages. No incluir `https://`, rutas ni `www` en ese archivo.

`www.girocero.es` se prepara mediante DNS apuntándolo al usuario de GitHub. Al usar el dominio raíz como custom domain, GitHub gestiona la redirección de `www` al dominio raíz cuando ambos resuelven correctamente. El certificado debe cubrir ambos nombres antes de dar por válido HTTPS.

Activar **Enforce HTTPS** cuando GitHub habilite la opción después de validar DNS y emitir el certificado. Puede tardar hasta 24 horas. No dar por terminado el lanzamiento mientras la opción siga pendiente o haya advertencias TLS. [HTTPS en GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https).

## Cambios DNS que debe realizar el titular en STRATO

**Mantener los servidores de nombres de STRATO y todos los registros de correo.** Los cambios web indicados abajo ya se han aplicado; el TXT de verificación se restauró como registro independiente del CNAME de `www`. Esta ruta no requiere delegar el dominio a Cloudflare.

Consulta pública previa, 8 de octubre de 2026 (no sustituye al inventario completo del panel):

| Tipo | Nombre | Valor observado | Acción |
| --- | --- | --- | --- |
| NS | girocero.es | shades11.rzone.de | Conservar |
| NS | girocero.es | docks04.rzone.de | Conservar |
| A | @ | 217.160.0.75 | Sustituir por 185.199.108.153 |
| AAAA | @ | 2001:8d8:100f:f000::200 | Sustituir por 2606:50c0:8000::153, o eliminar este AAAA web si se usa solo IPv4 |
| CNAME | www | girocero.es | Sustituir por Augernwf.github.io |
| MX | @ | smtp.rzone.de, prioridad 5 | Conservar sin cambios |

Configuración compatible con el panel de STRATO que admite una sola IP por tipo, **solo para la web**:

| Tipo | Host/nombre | Valor/destino | TTL |
| --- | --- | --- | --- |
| A | @ | 185.199.108.153 | 3600 s o predeterminado |
| AAAA | @ | 2606:50c0:8000::153 | 3600 s o predeterminado |
| CNAME | www | Augernwf.github.io. | 3600 s o predeterminado |

`@` significa el dominio principal; si STRATO muestra el dominio directamente, editar ese registro. El punto final del CNAME indica un nombre absoluto; si el formulario lo añade automáticamente, escribir `Augernwf.github.io`. No añadir `/giro-cero`, protocolo ni una IP en el CNAME. No crear CNAME en el dominio raíz ni comodines `*`.

En STRATO: Dominios → Administrar dominios → girocero.es → DNS. En Registro A, elegir IP propia e introducir únicamente `185.199.108.153`. GitHub permite configurar el dominio raíz con al menos un registro A; no introducir varias IP separadas por comas o espacios en un campo. La tabla oficial ofrece cuatro direcciones por tipo, pero no hace falta que STRATO admita todas para conectar la web. En Registro AAAA, sustituir la dirección antigua por `2606:50c0:8000::153`; IPv6 es opcional, por lo que también se puede eliminar exclusivamente el AAAA web antiguo si se usa solo IPv4. Nunca dejar IPv6 apuntando al alojamiento anterior.

En el subdominio `www`, sustituir el CNAME anterior por `Augernwf.github.io`; si existen A/AAAA propios de `www`, retirarlos al establecer el CNAME. No tocar otros subdominios ni registros de correo.

Preservar **MX, SPF, DKIM, DMARC**, así como registros de validación y los hosts de correo/autodiscover. No sustituir el MX observado por un ejemplo genérico del proveedor. Guardar una copia de la zona antes de editar; comprobar todos los registros del panel, ya que una consulta pública no enumera la zona completa. Si algún servicio de correo apunta al propio `girocero.es` como servidor, avisar antes de cambiar A/AAAA; el MX observado utiliza el host externo `smtp.rzone.de`.

Los valores web se contrastaron con [la tabla oficial de DNS de GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site). La gestión del panel se describe en [la documentación DNS de STRATO](https://www.strato.es/faq/dominios/que-registros-DNS-ofrece-STRATO-y-como-puedo-gestionarlos/).

### TXT de propiedad generado por GitHub

Añadir un registro adicional, sin sustituir ningún TXT existente:

| Tipo | Host/nombre | Valor | TTL |
| --- | --- | --- | --- |
| TXT | _github-pages-challenge-Augernwf | b7849179660b33404a517308134842 | 3600 s o predeterminado |

Nombre completo: `_github-pages-challenge-Augernwf.girocero.es`. El dominio ya está verificado en la cuenta Augernwf. Conservar este TXT para mantener la verificación. El 8 de octubre de 2026 se confirmó su presencia en ambos servidores autoritativos de STRATO. Es independiente de SPF, DKIM y DMARC. [Verificación de propiedad de GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages).

## Comprobación posterior a DNS

Comprobar que GitHub muestra **DNS check successful** en Settings → Pages. Activar Enforce HTTPS cuando esté disponible. Si aparece una restricción CAA, revisar primero los CAA existentes; no borrarlos indiscriminadamente. GitHub utiliza Let's Encrypt para emitir los certificados.

En PowerShell, estos comandos son consultas y no modifican nada:

```powershell
Resolve-DnsName girocero.es -Type A
Resolve-DnsName girocero.es -Type AAAA
Resolve-DnsName www.girocero.es -Type CNAME
Resolve-DnsName girocero.es -Type MX
Resolve-DnsName girocero.es -Type TXT
Resolve-DnsName _dmarc.girocero.es -Type TXT
curl.exe -I https://girocero.es/
curl.exe -I http://girocero.es/
curl.exe -I https://www.girocero.es/
curl.exe -I http://www.girocero.es/
curl.exe -I https://girocero.es/robots.txt
curl.exe -I https://girocero.es/sitemap.xml
curl.exe -I https://girocero.es/pagina-inexistente
curl.exe -sS -L --max-redirs 5 -o NUL -w 'status=%{http_code} final=%{url_effective} redirects=%{num_redirects}' 'http://www.girocero.es/robots.txt?check=1'
```

Esperado: HTTPS raíz → 200; HTTP y `www` → redirección permanente hacia HTTPS raíz sin bucles; robots/sitemap → 200; página inexistente → 404; el último comando termina en `https://girocero.es/robots.txt?check=1`, con estado 200. No usar `-k`: la validación del certificado debe estar activa. Revisar también el navegador sin avisos de certificado, imágenes/CSS/JS, selector EN/ES y enlace de contacto.

El correo requiere dos comprobaciones diferentes: conservar los DNS y probar la entrega real. Comparar MX y los registros SPF/DKIM/DMARC con el inventario anterior, incluidos los selectores DKIM del panel. Enviar después un mensaje externo a `oier@girocero.es`, responder desde STRATO y revisar recepción y autenticación SPF/DKIM/DMARC en las cabeceras. Estas pruebas de envío las realiza el titular; este despliegue no envía mensajes ni verifica el buzón por sí solo.

## Estado de esta preparación

Repositorio creado: https://github.com/Augernwf/giro-cero. GitHub Pages configurado con GitHub Actions. Primer deployment validado antes de asociar el dominio: https://github.com/Augernwf/giro-cero/actions/runs/37831759512 (Success, 8 de octubre de 2026). La URL inicial devolvió 200 para HTML, CSS, JavaScript, todas las imágenes, robots y sitemap, y 404 para una ruta inexistente. Selector EN/ES verificado en navegador y recursos visibles cargados.

Custom domain `girocero.es` guardado en GitHub y propiedad verificada en Augernwf. `dist/CNAME` añadido como referencia. DNS web comprobados: A `185.199.108.153`, AAAA `2606:50c0:8000::153` y CNAME `www` → `augernwf.github.io`. El TXT de propiedad se restauró en STRATO con «Crear otro registro», conservando el CNAME, DMARC y las opciones SPF existentes. MX sigue en `smtp.rzone.de`, prioridad 5, y DMARC sigue en `v=DMARC1;p=reject;`.

Comprobación del 8 de octubre de 2026, 21:55 (España): Enforce HTTPS activado y guardado. `https://girocero.es/` devuelve 200 con certificado válido; `https://www.girocero.es/` también supera la validación TLS y redirige con 301 a HTTPS raíz. Las tres variantes HTTP raíz, HTTPS www y HTTP www terminan en `https://girocero.es/` con 200, respectivamente en 1, 1 y 2 redirecciones, sin bucles. Robots y sitemap devuelven 200 y una ruta inexistente devuelve 404. Landing y selector EN/ES comprobados en el dominio de producción. TXT de propiedad, MX y DMARC conservan los valores esperados. GitHub confirma «DNS check successful» y mantiene Enforce HTTPS activado. Despliegue web validado. Pendiente únicamente para comprobar el correo de extremo a extremo: que el titular pruebe envío y recepción real del buzón. No se han enviado mensajes.

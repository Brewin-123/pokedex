# Pokedex Angular

Este documento detalla el proceso de configuración, compilación y despliegue de la aplicación Pokedex en un entorno de producción, incluyendo las políticas de seguridad y resolución de problemas comunes.


## Instalación y Ejecución

Se clonó el repositorio brindado por el maestro en el cual se instalaron las dependencias requeridas para el correcto funcionamiento del proyecto ya desplegado.
Al intentar instalar dependencias sin un repositorio de Git activo, el script de "prepare" fallaba.

 **Error:** `husky - .git can't be found`.
 
**Solución:**

	git init 
	npm install

**Creamos la carpeta de despliegue:**
```
 npm run build
```
Este comando nos deja una carpeta "dist" la cual tiene el proyecto a desplegar.


# Despliegue desde Azure

Aquí vamos a Azure para desplegar nuestro proyecto. Usamos "App Services" ya que al intentar por "Statir Web Apps" nos daba error al momento de desplegar.

## Creación de la página en Azure

Nos vamos a la opción antes mencionada (App Services), y desde ahí empezamos a crear nuestra aplicación web llamada pokedex. Al momento de crearla, pasamos a las herramientas avanzadas que es desde donde se sube el proyecto. Subimos solo el contenido que se encuentra en la carpeta "dist" que es la que generamos con el "ng-build". Todo se sube desde la consola en el "wwwroot" que está en la carpeta "site" de la página.

## Resolución de Errores en Producción
**Error 404 en Imágenes**

Al subir solo el contenido encontrado en la carpeta de pokedex-angular, las rutas de las imágenes tuvieron un problema con la ruta ya que siempre buscaban esa carpeta la cual no estaba. La solución más sencilla a este problema fue crear una carpeta con el nombre "pokedex-angular" y meter ahí la carpeta assets que es donde se encontraban las imágenes.


# Configuración del Servidor

Para que las rutas de Angular (SPA) funcionen y la seguridad sea máxima, se creó un archivo `web.config` en `src/` y se registró en el arreglo de `assets` de `angular.json`.

### Configuración de Seguridad para Grado A+

Para alcanzar la calificación máxima en auditorías de seguridad, se implementó una estrategia de tres capas: limpieza de código, optimización del framework y endurecimiento del servidor.
#### Limpieza del Punto de Entrada

Se eliminaron todos los scripts "inline" (código JavaScript escrito directamente en el HTML). Esto es crítico para permitir una política de seguridad que bloquee la ejecución de scripts no autorizados.
**Acción:** Se removió el script de redireccionamiento de GitHub Pages, ya que el ruteo ahora lo maneja Azure a través del `web.config`.

#### Optimización de Angular (`angular.json`)

Angular, por defecto, inyecta estilos CSS directamente en el HTML para mejorar el rendimiento percibido (_Critical CSS_). Sin embargo, esto viola las políticas de seguridad estrictas.

-   **Configuración:** Se modificó la sección de producción para forzar a Angular a usar archivos `.css` externos.

`"optimization": {
  "scripts": true,
  "styles": {
    "minify": true,
    "inlineCritical": false
  },
  "fonts": true
}`

#### Endurecimiento del Servidor (`web.config`)
Se configuró el archivo de control de Azure (IIS) para inyectar encabezados de seguridad en cada respuesta del servidor.

#### Encabezados Implementados:

-   **HSTS (Strict-Transport-Security):** Configurado con `max-age=31536000`, `includeSubDomains` y `preload` para asegurar que el sitio solo sea accesible vía HTTPS.
    
-   **X-Frame-Options (DENY):** Protege contra ataques de _Clickjacking_.
    
-   **X-Content-Type-Options (nosniff):** Evita que el navegador intente adivinar el tipo de contenido, previniendo ataques de inyección.
    
-   **Referrer-Policy (no-referrer):** Protege la privacidad del usuario al no enviar información de procedencia a otros sitios.
    
-   **Permissions-Policy:** Bloquea el acceso a hardware innecesario (cámara, micrófono, GPS).
    
-   **X-XSS-Protection:** Activa el filtro contra ataques de scripting entre sitios en navegadores antiguos.
    
-   **Cross-Origin-Opener-Policy (same-origin):** Aísla el contexto de navegación para prevenir fugas de datos entre pestañas.

### Content Security Policy (CSP) - El "Compromiso de Seguridad":

Para lograr el **A+** sin romper el funcionamiento de los componentes de Angular (que inyectan estilos dinámicamente mediante JS), se implementó una política híbrida:

-   **Scripts:** Bloqueo total de `'unsafe-inline'`. Solo se ejecutan archivos `.js` propios del servidor.
    
-   **Estilos:** Se permite `'unsafe-inline'` para que Angular pueda aplicar los diseños de los componentes, pero restringiendo las fuentes a dominios de confianza (Google Fonts).

**Configuración final del CSP:**

`'<add name="Content-Security-Policy" value="default-src 'self'; script-src 'self'; connect-src 'self' https://pokeapi.co https://beta.pokeapi.co; img-src 'self' data: https://raw.githubusercontent.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com;" />'`
    
### Ofuscación de Infraestructura

Se eliminó el encabezado `X-Powered-By` para no dar pistas a posibles atacantes sobre la tecnología del servidor.

`<remove name="X-Powered-By" />`

Con esto, el proyecto no solo cumple con su función estética y técnica de mostrar los Pokémon, sino que sigue los estándares de seguridad más rigurosos de la industria actual.
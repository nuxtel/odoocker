# Odoocker: The Ultimate Odoo Docker Framework

Bienvenido a Odoocker, una herramienta revolucionaria en el mundo del desarrollo e implementación de Odoo. Esta herramienta está meticulosamente diseñada para revolucionar tu experiencia con Odoo, garantizando simplicidad, eficiencia y un desarrollo de primer nivel. Y si bien se basa en los principios de la configuración oficial de Docker de Odoo, va mucho más allá.

Ya sea que utilice la edición **Community** o **Enterprise**, esta solución **Docker** está diseñada específicamente para usted.

¿Lo mejor de esto? No necesitas conocimientos previos de Docker, Odoo ni ninguna tecnología relacionada con este framework. Nos atenemos a la filosofía de Docker: úsalo y luego aprende.

No dudes en publicar una solicitud de extracción para seguir mejorando este proyecto.

### ¿Por qué Odoocker destaca?:
1. 🌐 **Universal:** Adecuado para las ediciones Odoo Community y Enterprise.
2. 📦 **Configuración fácil:** Clone, configure el archivo `.env` y estará listo para implementar.
3. 🔒 **Seguro:** Renovación automática del certificado SSL para mantener sus datos seguros (solo para producción).

En esencia, Odoocker no es solo una herramienta más, es una filosofía. Así que, tanto si eres un veterano de Odoo como si estás empezando, Odoocker está aquí para facilitarte el proceso.

## Contenido

- [Guía de configuración rápida](#quick-setup-guide)
- [El archivo `.env`](#the-env-file)
  - [Acciones base-desarrollo](#environment-based-actions)
    - [1. RFeciente o restaurado](#1-fresh-or-restore)
    - [2. Local](#2-local)
    - [3. Depurar](#3-debug)
    - [4. Pruebas](#4-testing)
    - [5. Completo](#5-full)
    - [6. Demostracion](#6-staging)
    - [7. Producion](#7-production)
- [Pro(d) Consejos](#prod-tips)
  - [1. Busque entre todos los complementos a la vez](#1-search-through-all-addons-at-once)
  - [2. Define los siguientes alias](#2-define-the-following-aliases)
  - [3. NUNCA ejecute **docker-compose down -v** en producción](#3-never-run-docker-compose-down--v-in-production)
  - [4. Odoo Shell](#4-odoo-shell)
  - [5. Odoo Scaffold](#5-odoo-scaffold)
  - [6. Colorea tus ramas](#6-colorize-your-branches)
- [DB Connecion](#db-connection)
  - [PgAdmin](#pgadmin-container) 
- [Proceso de implementación](#deployment-process)

# Guía de configuración rápida:

1. **Clonar y configurar**:
```
git clone git@github.com:odoocker/odoocker.git
cd odoocker
cp .env.example .env && cp docker-compose.override.local.yml docker-compose.override.yml
```
2. **Hosts y dominios**: para garantizar que todo funcione sin problemas, recuerde agregar los dominios necesarios a su archivo de hosts.

Para *Linux*:
```
echo '127.0.0.1 erp.odoocker.test' | sudo tee -a /etc/hosts
echo '127.0.0.1 pgadmin.odoocker.test' | sudo tee -a /etc/hosts
```
Para *Windows*, Agregue manualmente estas líneas a C:\Windows\System32\drivers\etc\hosts:
```
127.0.0.1 erp.odoocker.test
127.0.0.1 pgadmin.odoocker.test
```

# El archivo `.env`
Las variables de entorno ubicadas en [`.env`](https://github.com/odoocker/odoocker/blob/main/.env.example) Proporcionar configuraciones dinámicas a Odoo y al proyecto en general.
El [`odoo.conf`](https://github.com/odoocker/odoocker/blob/main/odoo/odoo.example.conf) El archivo es generado por el [`odoorc.sh`](https://github.com/odoocker/odoocker/blob/main/odoo/odoorc.sh) script basado en las variables de entorno.
<br>
Este archivo está dividido en secciones. Probablemente te centrarás en la sección "Configuración principal". Esta proporciona acceso rápido al proyecto y a las variables de "odoo.conf". El resto de la sección contiene variables específicas de cada contenedor. (Nota: Puedes encontrar variables repetidas como `SUPPORT_EMAIL=${SUPPORT_EMAIL}` que interactúa con diferentes contenedores. Esto indica explícitamente en el archivo `.env` que esta variable se comparte entre dichos contenedores.
Archivo `.env` de ejemplo:
```
# Odoo
APP_ENV=debug
INIT=
UPDATE=my_custom_addon
LOAD=base,web
WORKERS=2
DEV_MODE=reload,qweb
DOMAIN=erp.odoocker.test

# Enterprise (usuario de GitHub con acceso a Odoo Enterprise [https://github.com/odoo/enterprise] Repo)
# Si no está presente, se mostrará la Odoo Comunity
GITHUB_USER=odoocker
GITHUB_ACCESS_TOKEN=ghp_token

# Database
ADMIN_PASSWD=odoo
DB_HOST=postgres (container or external host)
DB_PORT=5432
DB_NAME=my-odoo-db
DB_USER=odoo
DB_PASSWORD=odoo
LOAD_LANGUAGE=es_MX
...
```
<br>

## Environment-based actions:
[`odoo/entrypoint.sh`](https://github.com/odoocker/odoocker/blob/main/odoo/entrypoint.sh) El archivo es la puerta de enlace para nuestro contenedor de Odoo. Dependiendo de `APP_ENV` y del resto de las variables de entorno, determina cómo iniciar el servicio de Odoo (como local, pruebas, producción, etc.) con diferentes configuraciones.
<br>
En todos los entornos, `odoo.conf` sigue las variables del archivo `.env`. Algunos entornos pueden tener parámetros de línea de comandos para sobrescribir ciertas configuraciones.

**Para que aparezcan todos los entornos ejecute**:
```
docker-compose up -d --build && docker-compose logs odoo
```
Restart adding `down`:
```
docker-compose down && docker-compose up -d --build && docker-compose logs odoo
```

### 1. Autualizar o Restaurar
Estos entornos (`APP_ENV=fresh` o `APP_ENV=restore`) no tendrán ninguna base de datos creada. Son ideales para configurar una instancia de **base de datos nueva** o **restaurar una base de datos de producción**.

### 2. Local:

Este entorno (`APP_ENV=local`) seguirá estrictamente las variables `.env` sin sobrescribirlas en la línea de comandos. Probablemente lo usará con frecuencia.
Utilice `DEV_MODE=reload,qweb` para activar la recarga en caliente al cambiar los archivos `python` y `xml`.
<br>
Si prefiere actualizar los paquetes cada vez que reinicia el contenedor Odoo, puede configurar `UPDATE=module1,module2,module3`.

### 3. Depurar:
Este entorno (`APP_ENV=debug`) funciona igual que el local, pero inicia Odoo usando la biblioteca `debugpy`. Gracias a nuestro [`.vscode/launch.json`](https://github.com/odoocker/odoocker/blob/main/.vscode/launch.json), si usas **Visual Studio Code**, puedes iniciar una sesión del depurador para que el contenedor detecte tus puntos de interrupción y se detenga donde lo necesites. Este es mi entorno favorito para trabajar, ya que uso mucho el depurador durante el desarrollo.

### 4. Pruebas:
Este entorno (`APP_ENV=testing`) es específico para ejecutar pruebas (y se incluirá en una canalización de CI/CD en una versión futura). Nos ayuda a probar los módulos que estamos desarrollando para garantizar una implementación segura.
<br>
A `test_DB_NAME` La base de datos se crea automáticamente.
El `ADDONS_TO_TEST=addon_1` se instalan en esa base de datos nueva.
Utilice `TEST_TAGS=test_tag_1` para filtrar sus pruebas.

**NOTA: Evite ejecutar pruebas sin etiquetas**; de lo contrario, se activarán pruebas en todos los complementos instalados, lo cual no queremos. Por ahora, supongamos que las pruebas de Odoo Community & Enterprise han sido exitosas y concentrémonos solo en lo que necesita probar.

### 5. Full:
Este entorno (`APP_ENV=full`) instalará los módulos `INIT` en un `DB_NAME` nuevo o existente. Esto nos permite tener una nueva réplica de la base de datos de producción.

### 6. Demostracion:
Este entorno (`APP_ENV=staging`) establece `UPDATE=all`; nos permite **actualizar todos los complementos instalados a la vez**.
<br>
También permite instalar nuevos paquetes antes de la actualización a través de 'INIT'.
<br>
Se recomienda encarecidamente utilizar este comando para ejecutar este entorno.
```
docker-compose down && docker-compose pull && docker-compose build --no-cache && docker-compose up -d && docker-compose logs -f odoo
```

Esto extraerá los últimos complementos de Odoo Community, Enterprise, Extra y Custom; básicamente, actualiza toda la instancia de Odoo a la versión más reciente. Además, también extraerá las imágenes más recientes de los demás contenedores del proyecto. Este entorno es perfecto para implementaciones.

**NOTA: No lo bajes y lo vuelvas a subir, a menos que quieras realizar una actualización completa nuevamente.**

### 7. Produccion:

Este entorno (`APP_ENV=production`) garantiza que no se carguen datos de demostración y que la depuración y el modo de desarrollo estén desactivados. Además, activa el contenedor `Let's Encrypt`, por lo que ya no tendrá que preocuparse por los `Certificados SSL`. Algunas variables `.env` se sobrescriben en esta configuración.

- Eliminar la configuración anterior de contenedores
```
docker-compose down
```
- Reemplace `docker-compose.override.yml` con `docker-compose.override.production.yml` para traer el contenedor `Let's Encrypt`.
```
cp docker-compose.override.production.yml docker-compose.override.yml
```
- Actualice .env `SERVICES` (add `acme`) y `ACME_CA_URI` (use el enlace de producción).
- Asegúrese de que el registro DNS de su 'DOMINIO' esté apuntando a su servidor.
- Reconstruir los contenedores
```
docker-compose up -d --build && docker-compose logs odoo
```

# Pro(d) Consejos
Los siguientes consejos mejorarán su experiencia de desarrollo y producción.
## 1. Busca entre todos los complementos a la vez:
Si usa Visual Studio Code y la extensión Docker está instalada, puede abrir el contenedor de Odoo en la ruta raíz. Allí encontrará todos los complementos de la comunidad, los complementos empresariales, los complementos adicionales y los complementos personalizados de Odoo en la misma carpeta.
<br>
El uso del Panel de búsqueda le permitirá ver cada referencia de lo que está buscando en toda la instancia de Odoo y no solo en sus complementos.

## 2. Defina los siguientes alias:
```
alias odoo='cd odoocker'

alias hard-deploy='docker-compose down && git pull && docker-compose pull && docker-compose build --no-cache && docker-compose up -d && docker-compose logs -f odoo'

alias deploy='docker-compose down && git pull && docker-compose up -d --build && docker-compose logs -f --tail 2000 odoo'

alias logs='docker-compose logs -f --tail 2000 odoo'
```

## 3. NUNCA ejecute `docker-compose down -v` en producción
...Sin tener una base de datos ``copia de seguridad comprobada``
<br>
Tenga en cuenta que eliminar volúmenes destruirá datos de la base de datos, Odoo Conf & Filestore, *certificados Let's Encrypt y más!* Si ejecuta este comando varias veces en `prod` en un corto período de tiempo, puede alcanzar el `límite de certificados Let's Encrypt`` y Odoocker no podrá generar nuevos después de **varias horas**.

## 4. Odoo Shell
1. Log into the odoo container
```
docker-compose exec odoo bash
```
2. Start Odoo shell running:
```
odoo shell --http-port=8071
```

## 5. Odoo Scaffold
1. Log into the odoo container
```
docker-compose exec -u root odoo
```
2. Navigate to custom addons folder inside the container
```
cd /usr/lib/python3/dist-packages/odoo/custom-addons
```
3. Create new addons running:
```
odoo scaffold <addon_name>
```
- The new addon will be available in the `odoo/custom_addons` folder in this project.

## 6. Colorea tus ramas
Añade lo siguiente a `~/.bashrc`
```
# Color git branches
function parse_git_branch () {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}

if [ "$color_prompt" = yes ]; then
    #PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
    # Color git branches
    PS1="${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w \[\033[01;31m\]\$(parse_git_branch)\[\033[00m\]\$ "
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt
```

# DB Connection
- Cualquier otro administrador de base de datos Postgres puede conectarse a la base de datos utilizando las credenciales `.env`.

## PgAdmin
- Este proyecto viene con un contenedor PgAdmin que se carga solo en `docker-compose.override.pgadmin.yml`.
Para administrar la base de datos proporcionamos un contenedor pgAdmin.
Para hacer esto, simplemente ejecute:
```
docker-compose -f docker-compose.yml -f docker-compose.override.yml -f docker-compose.pgadmin.yml up -d --build
```
And to turn down
```
docker-compose -f docker-compose.yml -f docker-compose.override.yml -f docker-compose.pgadmin.yml down
```

Si su instancia tiene pgAdmin, asegúrese de adaptar sus alias a esta configuración.

# Proceso de implementación
Nota: el proceso de implementación es más fácil y rápido con alias.

1. Realice una copia de seguridad de las bases de datos de producción desde `/web/database/manager`.
2. Ejecutar
```
sudo apt update && sudo apt upgrade -y
```
- If packages are kept, install them
```
sudo apt install <kept packages>
```
3. Reiniciar el servidor
```
sudo reboot
```
- Asegúrese de que no haya más actualizaciones ni posibles paquetes retenidos
```
sudo apt update && sudo apt upgrade -y
```
4. Vaya a la carpeta del proyecto en /home/ubuntu o (~)
```
cd ~/odoocker
```
o con alias:
```
odoo
```
5. Extraiga los últimos cambios de la rama `principal`.
```
git pull origin main
```
6. Establecer el entorno de [`Staging`](#6-staging)
7. Establecer el entorno de [`Producción`](#7-producción)

# Nota final
Este proyecto se basa en la imagen [Oficial de Docker de Odoo](https://hub.docker.com/_/odoo/). Nos hemos esforzado por garantizar una integración fluida con la configuración original de Docker, realizando las personalizaciones necesarias para adaptarnos a nuestras necesidades. Animamos a los colaboradores y usuarios a consultar con frecuencia la documentación oficial para conocer los conceptos básicos y las actualizaciones. Gracias por su continuo apoyo y confianza en nuestro proyecto.

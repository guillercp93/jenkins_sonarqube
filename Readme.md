# Configuración de herramientas para DEVOPS

## **Jenkins**

### **Instalación de la herramienta**

La instalación de Jenkins será usando docker para deplegarla. Para ello nos serviremos del archivo `docker-compose.yaml` para facilitar el proceso de instalación.

De este archivo puede usted modificar los puertos que por defecto son *8080* y *50000* a la red del host.

```yaml
    ports:
      - 8080:8080
      - 50000:50000
```

También puede cambiar la ubicación del directorio de configuración de Jenkins. Tenga en cuenta que la ubicación `home/${USER}/containers/jenkins_home:/var/jenkins_home` será mapeada a `var/jenkins_home` en el contenedor.

```yaml
    volumes:
      - /home/${USER}/containers/jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
```

Ya que se va a ejecutar ambos contenedores en un mismo ambiente, es neceario crear una red para estos. En el archivo, se tiene el nombre *devops-net*
```yaml
networks:
  devops-net:
    external: true
```

Use el siguiente comando para crear la red en docker o cambie el archivo si ya posee una ya creada.
```sh
$ docker network create -d bridge devops-net
```

**NOTA**: Tenga en cuenta que debe crear la carpeta en el equipo host la cual va a mapearse. Para este caso, se creará la carpeta *jenkins_home* en la ruta `home/${USER}/containers/jenkins_home`

Para iniciar el servicio de Jenkins use el comando
```sh
$ docker-compose up -d
```

### **Creación del super usuario**

Para empezar a usar Jenkins, la herramienta nos pedirá una clave inicial para el superusuario. Para ello, ejecutamos en la consola:
```sh
$ docker logs jenkins | less
```
Y buscamos el mensaje donde dice: *Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation*, después de ese mensaje estará la clave.

Otra forma de obtenerlo es entrando en el contenedor e ir a la ruta `/var/jenkins_home/secrets/initialAdminPassword`
```sh
$ docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Después se le mostrará un formulario donde le pedirá los siguientes campos:
- Nombre de usuario
- Contraseña
- Confirmar contraseña
- Nombre completo
- Correo electrónico


### **Seleccionar los plugins a instalar**
Después se mostrará dos opciones, la primera es instalar los plugins recomendados o seleccionar los plugins que va a necesitar.

## **SonarQube**
### **Instalación de la herramienta**
Para la instalación de la herramienta es necesario crear las siguientes carpetas en las siguientes ubicaciones:
- `/home/${USER}/containers/sonarqube/data`
- `/home/${USER}/containers/sonarqube/extensions`
- `/home/${USER}/containers/sonarqube/logs`
- `/home/${USER}/containers/postgresql/conf`
- `/home/${USER}/containers/postgresql/data`

**NOTA**: Estas ubicaciones se pueden cambiar, pero tenga en cuenta que también debe modificarlas en el archivo `docker-compose.yaml`.

Las herramientas necesarias para instalar es *SonarQube* y una base de datos, que en este caso es *Postgres*.

### **Creación del super usuario**

Cuando se instala, SonarQube ya viene con un usuario administrador por defecto. Los datos para ingresar son:
- **Nombre de usuario:** admin
- **Contraseña:** admin

### **Instalar plugins de administracion de credenciales**
Tener presente que los proceso de las configuraciones que siguen necesitan del manejo de credenciales. Para esto se va a instalar los siguientes plugins para tal fin:
- Credentials plugin
- Credentials Binding

### **Instalacion de plugins Pipeline**
Para la creacion de jobs de tipo Pipeline, es necesario que se descargue el plugin:
- **Pipeline:** Descargara las demas dependencias
- **Workspace Cleanup:** Limpieza del espacio de trabajo, necesario en los pipeline

### **Integrar Jenkins con SonarQube**
Lo primero es generar el token de SonarQube usando la cuenta del administrador.

Para esto se va a **Mi cuenta**, luego a la pestaña de **Seguridad** y en el formulario de generar token, ingrese el nombre que identifica el nuevo token y de clic en el botón **Generar**. Con esto se generará una clave que deberá copiar al instante. [Referencia a SonarQube](https://docs.sonarqube.org/latest/user-guide/user-token/)

Despues de esto, ingresar a Jenkins con el usuario administrador, ir al menu y seleccionar **Administrar Jenkins**, luego ir al apartado de **Manage credentials**, ir al Scope llamado *System*, luego en *Credenciales globales* y dar clic en el boton de *Agregar credenciales*.
Selecciona el tipo *Secret text*, y en el campo *Secret* pegar la clave que generaste en SonarQube.
Luego ingresas un ID y una descripcion si lo vez necesario. Si no colocas un ID, el sistema genera uno automaticamente.

### **Instalacion del plugin *SonarQube Scanner for Jenkins***

El siguiente paso es instalar la herramienta de **SonarQube Scanner for Jenkins**, para eso vamos al menu principal y selecciona la opcion *Administrar Jenkins*, luego a *Administrador de Plugins*, donde seleccionamos la pestana de *Disponible* y buscamos instalar *SonarQube Scanner for Jenkins*.

Despues de que se instale, vuelve al *Administrador de Jenkins* e ingresa en la *Configuracion del sistema*. Busca el apartado de **SonarQube servers** y da clic en el boton de *Agregar SonarQube*.
Se te pedira los siguientes datos:
- **Nombre:** Coloca un nombre cualquiera. Por ejemplo: localSQ
- **Url del servidor**: URL donde esta desplegada SonarQube. Colocar http://sonarqube:\<puerto especificado en el archivo de docker-compose\>
- **Token de autenticacion**: Selecciona la credencial que creaste en el paso anterior.

Luego se debe ir a *Configuración de herramienta global* e ir a los apartados de *SonarScanner for MSBuild* y *SonarQube Scanner*. En cada uno agregar el respectivo MSBuid y Scanner de SonarQube desde Github y Maven Central, se recomienda la ultima version.

### **Conexión a un repositorio de Github**
Para poder conectarse a un repositorio alojado en github, antes que nada, debe asegurarse que tiene el plugin de GIT instalado, luego lo que se debe hacer es generar un *token de acceso personal*. Para hacer esto, se debe ir a **Configuración**, luego en el menú de la izquierda seleccionar **Configuración de desarrollador** y por último seleccionar **Token de acceso personal**.
Luego, se da clic sobre el botón de **Generar nuevo token**, colocamos una *descripción* y por último seleccionamos los siguientes **scopes**:
- [x] repo
  - [x] repo:status
  - [x] repo_deployment
  - [x] public_repo
  - [x] repo:invite
  - [x] security_events
- [ ] admin:org
  - [x] read:org
- [ ] user
  - [x] user:email

Después de esto, ingresar a Jenkins con el usuario administrador, ir al menú y seleccionar **Administrar Jenkins**, luego ir al apartado de **Manage credentials**, ir al Scope llamado *System*, luego en *Credenciales globales* y dar clic en el botón de *Agregar credenciales*.
Selecciona el tipo *Username y Password*, y en el campo **Username** colocar tu nombre de usuario de github, habilitar la opcion de *tratar nombre de usuario como secreto*, en el campo **Password** pegar el *token* que se acabó de generar en *Github*.
Luego ingresas un ID y una descripcion si lo vez necesario. Si no colocas un ID, el sistema genera uno automaticamente.

También puedes colocar una descripción para identificar que esta es la clave de autenticación para github.

Después se debe verificar cuál es la URL del **webhook** para Github. Se tiene que ir a **Administrar Jenkins**, luego **Configuración del sistema**, en el apartado de *Github* buscar la opción de **Sobreescribir la URL del Hook**, tendrá por defecto una parecida a esta: http://localhost:8080/github-webhook/.

Copia la URL y luego ve al repositorio en github, da clic en la pestaña de **Configuración** y luego en el menú de la izquierda selecciona **WebHooks**, a la derecha presiona el botón **Agregar webhook** y sólo debes llenar el campo *Payload URL* con la dirección que copiaste anteriormente de Jenkins.

## **Consideraciones por lenguaje de programación**
### **Para .NET Core**
Se debe instalar los siguientes plugins para poder compilar, ejecutar pruebas y enviar a SonarQube:
- **xUnit Plugin:** Este será el encargado de ejecutar las pruebas de una aplicación en .net Core.
- **.NET SDK Support:** Con este plugin podemos ejecutar todas las funciones del comando *dotnet*, como por ejemplo compilar la aplicación. Con este último, debemos ir a *Configuración de herramientas global* e ir al apartado de *.NET SDK*, donde se colocará un **nombre**, se activa el check de **instalar automáticamente** y se hace clic sobre el botón de **añadir instalador** donde se selecciona *instalar desde microsoft.com*. Se nos va a desplegar un formulario donde escogeremos la *versión*, el *release*, la versión del *SDK* y por último la *plataforma de ejecución*, que en este caso es **linux**.
- **
## **Referencias**
- [Sonar Scanner para Jenkins](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/)
- [Github, Jenkins y Ngrok](https://www.youtube.com/watch?v=YkabAT213h0)
- [Configurar Azure Active Directory en SonarQube](https://github.com/hkamel/sonar-auth-aad/wiki/Setup)
- [Configurar Azure Active Directory en Jenkins](https://plugins.jenkins.io/azure-ad/)


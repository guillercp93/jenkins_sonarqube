# Configuración de herramientoas para DEVOPS

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
- **Url del servidor**: URL donde esta desplegada SonarQube
- **Token de autenticacion**: Selecciona la credencial que creaste en el paso anterior.

Luego se debe ir a *Configuracion de herramienta global*, e ir a los apartados de *SonarScanner for MSBuild* y *SonarQube Scanner*. En cada uno agregar el respectivo MSBuid y Scanner de SonarQube desde Github y Maven Central, se recomienda la ultima version.

## **Consideraciones por lenguaje de programacion**

### **Para .NET Core**


https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/

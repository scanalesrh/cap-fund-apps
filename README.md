# Iniciando el Taller
- Creación de contenedor Jenkins.
- Instalando Jenkins Openshift Client.
- Generando service account token.
- Configurando Jenkins Openshift Client.
- Configurando NameSpace en JenkinsFile y OCP Template.

# Resolución de Problemas
## Errores de Registro de imágenes
- Ver en eventos push/pull Url.
- Validar resolución de nombres para servicio "registry" (docker-registry.default.svc).
- Validar IP para "docker-registry.default.svc".

# Anexo

## Descripción del repositorio
### JenkinsFile
Archivo de orquestación para el proceso de integración y despliegue continuo (Pipeline). Posee una implementación específica para Jenkins mediante lenguaje "groovy" y cliente Openshift (Jenkins plugin).
Es el encargado de ejecutar el extenso proceso desde el código aplicativo hasta el despliegue pre-productivo y/o productivo.
### eap71-basic-s2i.json
Archivo de infraestructura como código. Posee una implementación específica para Openshift Container Platform (OCP) mediante estructura json/YAML y definiciones de objectos de la API OCP.
Representa el runtime, servicios y rutas de acceso web necesarias para la correcta ejecución del componente de negocio.
Toda la infraestructura dinámica que requiera el componente de negocio debe estar respresentado en este archivo. Considera dentro de los aspectos principales:
- Sistema Operativo (runtime).
- Framework/Middleware Platform (runtime).
- Redes.
- Almacenamiento.
- Balanceo.
- Alta disponibilidad.
- Recuperación ante falla.
## APLICACION GENERADA CON MAVEN
```
mvn archetype:generate -DgroupId=com.redhat.cap.app \
	-DartifactId=workshopWebApp \
	-DarchetypeArtifactId=maven-archetype-webapp \
	-DinteractiveMode=false
```
## Creando usuarios HTPasswd

```
ansible -i hosts masters -m shell -a "htpasswd -b /etc/origin/master/htpasswd scanales scanales"
```

## Utilizando Ansible para Usuarios y Namespaces
```
ansible-playbook -i localhost crear-usuarios-namespace.yaml
```
Requiere:
- Ser ejecutado en bastión (host con accesos a todos los nodos)
- Requiere ansible
- Requiere Openshift Cli (oc)
- Inventario ansible con nodos maestros (masters)
- Requiere system:admin (cluster admin) login

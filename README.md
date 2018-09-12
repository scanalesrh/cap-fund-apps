
# Taller de Integración y Despliegue Continuo
*Continuous Integration and Continuous Delivery with Openshift*

## Para Comenzar

Bienvenido al taller de integración y despliegue continuo, en el contexto del desarrollo de aplicaciones nativas en nuble (Cloud Native Application Development, a.k.a CNAD).

Reglas:
- Disfrutar.
- Reir.
- Colaborar.
- Preguntar.
- Contribuir.

- Cluster de Innovación Openshift: https://openshift.innovate.cnad.io/
- Openshift Command Line Interface (oc): https://github.com/openshift/origin/releases
- Pad colaborativo: https://etherpad.net/p/cap-fund-apps

## Actividades del Taller
- Creación de contenedor Jenkins (OCP Web Console).
- Instalando Jenkins Openshift Client (Jenkins Web Console).
```
 Manage Jenkins > Plugin Manager > Filter: openshift > Select: OpenShift Client, OpenShift Pipeline > Download now and  install after restart > Check: Restart Jenkins when installation is complete and no jobs are running
```
- Copiar comando de login
```
oc login https://openshift.innovate.cnad.io:443 --token=XXX
```
- Validar que estás usando el proyecto CICD.
```
Using project "usr99-cicd"
```
- Generando service account token (oc).
```
oc sa get-token -n myproject-cicd jenkins
```
- Configurando Jenkins Openshift Client (Jenkins Web Console).
- Creando template Gitlab.
```
oc create -f https://gitlab.com/gitlab-org/omnibus-gitlab/raw/master/docker/openshift-template.json
```
- Creación de contenedor Gitlab (OCP Web Console).
- Permisos especiales OCP para Gitlab.
```
oc adm policy add-scc-to-user anyuid system:serviceaccount:myproject-cicd:gitlab-ce-user
```
- Creación de proyecto git e importación de fuentes.
- Configurando NameSpace en JenkinsFile y OCP Template (Repositorio de fuentes).
- Configurando Pipeline desde SCM.
- Otorgando permisos a Jenkins Service Account sobre los proyectos (Namespaces)

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

## Autores
- Sergio Canales <scanales@redhat.com>
- Gerald Schmidt <gschmidt@redhat.com>

# Enlaces útiles

- [Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview/)

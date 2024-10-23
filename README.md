Aquí está el README modificado según la nueva información que proporcionaste, manteniendo la misma estructura:

---

README

# Reto2 Tópicos Especiales en Telemática

## Tópicos Espec. en Telemática - C2466-ST0263-1716

*Presentación y video en el Releases del repositorio*

### Estudiantes: 
- Nicolas Tovar Almanza - ntovara@eafit.edu.co
- Samuel Acosta Aristizábal - sacostaa1@eafit.edu.co

### Profesor: 
Alvaro Enrique Ospina Sanjuan - aeospinas@eafit.edu.co

### Nombre del proyecto, lab o actividad

Tópicos Especiales en Telemática, 2024-2 Reto No 2

Despliegue de LMS Moodle en Kubernetes con alta disponibilidad, balanceo de cargas, y almacenamiento externo

### Presentación
[Presentacion en CANVA]()

### Video
[Video en Youtube]()

### Documentación
## 1. Breve descripción de la actividad

La actividad consiste en desplegar un sistema de gestión de aprendizaje (LMS) Moodle en un clúster de alta disponibilidad utilizando Kubernetes. El sistema cuenta con un balanceador de cargas (Nginx), una base de datos separada (DBServer) y un servidor de archivos (FileServer). Todo el sistema está autoescalado y gestionado en un entorno en la nube, con un dominio personalizado y certificado SSL.

El objetivo del reto es lograr que el clúster mantenga la disponibilidad en la capa de aplicación, base de datos y almacenamiento, así como implementar un dominio accesible mediante HTTPS.

### 1.1. Qué aspectos cumplió o desarrolló de la actividad propuesta por el profesor 
#### Requerimientos Funcionales Cumplidos:

- **Alta disponibilidad y autoescalado:** Se configuró Kubernetes para que los pods de la aplicación Moodle y los servicios asociados, como la base de datos y almacenamiento de archivos, se escalen automáticamente según la carga. Esto asegura que la plataforma LMS pueda soportar un alto tráfico sin interrupciones.
- **Balanceo de cargas:** Nginx se implementó como un balanceador de cargas en la capa de aplicación para distribuir eficientemente las solicitudes de los usuarios entre los diferentes pods de Moodle.
- **Dominio y SSL:** Se implementó un dominio personalizado para el acceso a Moodle (https://reto2.sudominio.tld) con un certificado SSL para asegurar las conexiones.

#### Requerimientos No Funcionales Cumplidos:

- **Portabilidad:** La implementación de Docker y Kubernetes garantiza que el sistema pueda desplegarse en cualquier entorno de nube compatible con Kubernetes, como AWS o GCP, sin ajustes significativos.
- **Escalabilidad:** Kubernetes se configuró para manejar la adición dinámica de nuevos nodos, permitiendo el escalado vertical y horizontal del clúster según las necesidades de tráfico.
- **Seguridad:** Al utilizar un certificado SSL, se asegura que todas las comunicaciones entre los usuarios y la plataforma Moodle estén encriptadas.

### 1.2. Qué aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor 

Se cumplió con la mayoría de los aspectos. Sin embargo, la implementación de un servicio NFS externo fue opcional, y el equipo optó por una solución gestionada en la nube para simplificar la configuración del servidor de archivos.

## 2. Información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.

### Diseño de Alto Nivel:
El sistema se basa en una arquitectura distribuida utilizando Kubernetes para gestionar la disponibilidad y el escalado de los diferentes componentes de Moodle. La arquitectura incluye una capa de balanceo de cargas (Nginx), una base de datos separada gestionada en un servicio de base de datos dentro del clúster Kubernetes, y un servidor de archivos basado en NFS para almacenar los archivos de Moodle de manera distribuida.

#### Arquitectura del Despliegue:
  El sistema se divide en tres capas principales: 
  1. **Capa de Aplicación:** Moodle se despliega en pods de Kubernetes que son autoescalables y balanceados por Nginx.
  2. **Capa de Base de Datos:** Se implementa un servicio de base de datos externo dentro del clúster Kubernetes.
  3. **Capa de Almacenamiento:** El almacenamiento de archivos se maneja mediante un servidor NFS, que puede estar gestionado o desplegado dentro del clúster.

  Todo el sistema está desplegado en un entorno de nube utilizando AWS o GCP, con capacidad para escalar tanto vertical como horizontalmente según la demanda.

#### Patrones de Diseño y Mejores Prácticas:
- **Patrón de Microservicios:** Cada componente del sistema (Moodle, base de datos, servidor de archivos) está desplegado como un servicio independiente, lo que facilita su escalado y gestión.
- **Uso de Kubernetes:** Kubernetes se encarga del orquestado y escalado automático de los pods, asegurando alta disponibilidad y resistencia frente a fallos.
- **Balanceo de cargas con Nginx:** Nginx distribuye las solicitudes de los usuarios entre los diferentes pods de Moodle, optimizando el uso de los recursos del clúster.
- **Uso de Docker:** Todo el sistema está contenerizado para asegurar que funcione de manera consistente en diferentes entornos.

## 3. Descripción del ambiente de desarrollo y técnico: 

### Lenguaje de programación

El proyecto está desarrollado principalmente en **PHP** (para Moodle), con scripts adicionales en **Python** y **Bash** para la automatización de la configuración y despliegue.

### Herramientas y tecnologías utilizadas:
- **Kubernetes:** Orquestador de contenedores para gestionar el despliegue, escalado y disponibilidad.
- **Docker:** Para contenerizar los servicios de Moodle, la base de datos y el servidor de archivos.
- **Nginx:** Implementado como balanceador de cargas en la capa de aplicación.
- **MySQL (DBServer):** Base de datos desplegada dentro del clúster.
- **NFS (FileServer):** Sistema de archivos distribuido para manejar los archivos de Moodle.

### Detalles del desarrollo.

**Estructura del Código:**
- moodle-deployment.yaml: Contiene la configuración de los despliegues y servicios de Moodle en Kubernetes.
- nginx-config.yaml: Configura Nginx como balanceador de cargas.
- mysql-deployment.yaml: Configura el servicio de base de datos en Kubernetes.
- nfs-server-deployment.yaml: Configura el servidor de archivos NFS.
- docker-compose.yml: Para orquestar el entorno de desarrollo local.
- scripts/: Directorio con scripts para la configuración inicial del clúster y despliegue de los servicios.

### Ambiente de Desarrollo:

Se recomienda el uso de una máquina con Ubuntu para la instalación y configuración del clúster Kubernetes.

### Descripción y configuración de los parámetros del proyecto 

Para ejecutar el proyecto, sigue los pasos en el archivo `Run.txt`:

**1. Configurar Kubernetes en la nube:**
Utiliza un servicio como EKS (AWS) o Kubernetes en GCP.

**2. Clonar el Repositorio:**
```bash
git clone https://github.com/YourRepo/Reto2K8s.git
cd Reto2K8s
```

**3. Desplegar los servicios en Kubernetes:**
```bash
kubectl apply -f moodle-deployment.yaml
kubectl apply -f nginx-config.yaml
kubectl apply -f mysql-deployment.yaml
kubectl apply -f nfs-server-deployment.yaml
```

**4. Acceder a la aplicación:**
Accede a la aplicación Moodle desde el dominio: https://reto2.sudominio.tld

### Variables de Entorno y Configuración en Kubernetes

**Dominio:** El dominio utilizado para el servicio es https://reto2.sudominio.tld, y se utiliza un certificado SSL para garantizar comunicaciones seguras.

**Configuración de Red:** Los servicios están desplegados en un clúster de Kubernetes con configuración de red privada para garantizar la seguridad interna.

## 4. Descripción del ambiente de EJECUCIÓN (en producción)

**Lenguaje de Programación:** PHP (Moodle), con scripts en Python y Bash para la automatización.

**Paquetes y Librerías Utilizadas:**
- **Kubernetes (kubectl):** Versión 1.21
- **Docker:** Versión 20.10.7
- **MySQL:** Versión 8.0
- **Nginx:** Versión 1.19

### Configuración de IP y Puertos:

La configuración de los servicios en Kubernetes está manejada mediante los manifiestos YAML.

### Cómo se lanza el servidor:

#### 1. Acceso al clúster Kubernetes:

Accedemos al clúster en AWS o GCP y verificamos que todos los servicios estén corriendo correctamente.

#### 2. Despliegue del servicio:
```bash
kubectl apply -f moodle-deployment.yaml
```

## 5. Referencias:
- Despliegue de WordPress en AWS EKS: https://github.com/st0263eafit/st0263-242/tree/main/eks-wp
- Documentación oficial de Kubernetes: https://kubernetes.io/docs/home/

---

Este README se ajusta al nuevo contenido del reto sin cambiar la estructura original.

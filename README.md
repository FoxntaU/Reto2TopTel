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

Despliegue de CMS **DRUPAL** en Kubernetes con alta disponibilidad, balanceo de cargas, y almacenamiento externo

### Presentación
[Presentacion en CANVA](https://www.canva.com/design/DAGVm1YMVts/Vu6GQm-FVb-lT7ItbsMyoQ/edit)

### Video
[Video en Youtube](https://youtu.be/-K8_54qzKMU)

### Documentación
## 1. Breve descripción de la actividad

La actividad consiste en desplegar un sistema de gestión de contenidos Drupal en un clúster de alta disponibilidad utilizando Kubernetes. El sistema cuenta con un balanceador de cargas (Nginx), una base de datos separada (DBServer) y un servidor de archivos (FileServer). Todo el sistema está autoescalado y gestionado en un entorno en la nube, con un dominio personalizado y certificado SSL.

El objetivo del reto es lograr que el clúster mantenga la disponibilidad en la capa de aplicación, base de datos y almacenamiento, así como implementar un dominio accesible mediante HTTPS.

### 1.1. Qué aspectos cumplió o desarrolló de la actividad propuesta por el profesor 
#### Requerimientos Funcionales Cumplidos:

- **Alta disponibilidad y autoescalado:** Se configuró Kubernetes para que los pods de la aplicación Drupal y los servicios asociados, como la base de datos y almacenamiento de archivos. Esto asegura que la plataforma LMS pueda soportar un alto tráfico sin interrupciones.
- **Balanceo de cargas:** Nginx se implementó como un balanceador de cargas en la capa de aplicación para distribuir eficientemente las solicitudes de los usuarios entre los diferentes pods de Drupal.
- **Dominio y SSL:** Se implementó un dominio personalizado para el acceso a Drupal (https://reto2.sudominio.tld) con un certificado SSL para asegurar las conexiones.

#### Requerimientos No Funcionales Cumplidos:

- **Portabilidad:** La implementación de Docker y Kubernetes garantiza que el sistema pueda desplegarse en cualquier entorno de nube compatible con Kubernetes, como AWS o GCP, sin ajustes significativos.
- **Escalabilidad:** Kubernetes se configuró para manejar la adición dinámica de nuevos nodos, permitiendo el escalado vertical y horizontal del clúster según las necesidades de tráfico.
- **Seguridad:** Al utilizar un certificado SSL, se asegura que todas las comunicaciones entre los usuarios y la plataforma Drupal estén encriptadas.

### 1.2. Qué aspectos NO cumplió o desarrolló de la actividad propuesta por el profesor 

Se cumplió con todos los aspectos requeridos para este reto, aunque se vieron algunas dificultades con el software moodle debido a que no se pudo escalar y solo nos permitia tener en modo running un solo pod; sin embargo se pudo realizar satisfactoriamente con Drupal
[Link al codigo de drupal](https://github.com/FoxntaU/Reto2TopTel/tree/main/Drupal)

## 2. Información general de diseño de alto nivel, arquitectura, patrones, mejores prácticas utilizadas.

### Diseño de Alto Nivel:
El sistema se basa en una arquitectura distribuida utilizando Kubernetes para gestionar la disponibilidad y el escalado de los diferentes componentes de Drupal. La arquitectura incluye una capa de balanceo de cargas (Nginx), una base de datos separada gestionada en un servicio de base de datos dentro del clúster Kubernetes, y un servidor de archivos basado en NFS para almacenar los archivos de Drupal de manera distribuida.

#### Arquitectura del Despliegue:
  El sistema se divide en tres capas principales: 
  1. **Capa de Aplicación:** Drupal se despliega en pods de Kubernetes y balanceados por Nginx.
  2. **Capa de Base de Datos:** Se implementa un servicio de base de datos externo dentro del clúster Kubernetes.
  3. **Capa de Almacenamiento:** El almacenamiento de archivos se maneja mediante un servidor EFS, que puede estar gestionado dentro del clúster.

  Todo el sistema está desplegado en un entorno de nube utilizando AWS, con capacidad para escalar tanto vertical como horizontalmente según la demanda.

#### Patrones de Diseño y Mejores Prácticas:
- **Patrón de Microservicios:** Cada componente del sistema (Drupal, base de datos, servidor de archivos) está desplegado como un servicio independiente, lo que facilita su escalado y gestión.
- **Uso de Kubernetes:** Kubernetes se encarga del orquestado y escalado automático de los pods, asegurando alta disponibilidad y resistencia frente a fallos.
- **Balanceo de cargas con Nginx:** Nginx distribuye las solicitudes de los usuarios entre los diferentes pods de Drupal, optimizando el uso de los recursos del clúster.

## 3. Descripción del ambiente de desarrollo y técnico: 

### Herramientas y tecnologías utilizadas:
- **Kubernetes:** Orquestador de contenedores para gestionar el despliegue, escalado y disponibilidad.
- **Nginx:** Implementado como balanceador de cargas en la capa de aplicación.
- **MySQL (DBServer):** Base de datos desplegada dentro del clúster.
- **EFS (FileServer):** Sistema de archivos distribuido para manejar los archivos de Drupal.

### Detalles del desarrollo.

**Estructura del Código:**
- all.yaml:   Archivo maestro para aplicar todos los recursos
- drupal-efs-pv.yaml: Persistent Volume (PV) para el almacenamiento de Drupal
- drupal-efs-pvc.yaml:  Persistent Volume para la base de datos
- db-pv.yaml: Persistent Volume Claim (PVC) para el almacenamiento de Drupal
- db-pvc.yaml:  Persistent Volume Claim para la base de datos
- clusterissuer.yaml: Configuración de ClusterIssuer para Cert-Manager
- drupal-certificate.yaml: Certificado para el dominio de Drupal
- efs-cleanup-pod.yaml: Pod para limpiar recursos en EFS


### Ambiente de Desarrollo:

Se recomienda el uso de una máquina con Ubuntu para la instalación y configuración del clúster Kubernetes.

### Descripción y configuración de los parámetros del proyecto 

Para ejecutar el proyecto, sigue los pasos en el archivo `commands.txt`:

**1. Configurar Kubernetes en la nube:**
Utiliza un servicio como EKS (AWS).

**2. Clonar el Repositorio:**
```bash
git clone https://github.com/FoxntaU/Reto2TopTel
cd Reto2TopTel
```

**3. Desplegar los servicios en Kubernetes:**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
kubectl get pods -n ingress-nginx
kubectl apply -f drupal-efs-pv.yaml
kubectl apply -f drupal-efs-pvc.yaml
kubectl apply -f db-pv.yaml
kubectl apply -f db-pvc.yaml
kubectl get pvc
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.yaml
kubectl apply -f clusterissuer.yaml
kubectl apply -f drupal-certificate.yaml
kubectl apply -f all.yaml
```

**4. Acceder a la aplicación:**
Accede a la aplicación Drupal desde el dominio: https://reto2.sudominio.tld

### Variables de Entorno y Configuración en Kubernetes

**Dominio:** El dominio utilizado para el servicio es https://reto2.sudominio.tld, y se utiliza un certificado SSL para garantizar comunicaciones seguras.

**Configuración de Red:** Los servicios están desplegados en un clúster de Kubernetes con configuración de red privada para garantizar la seguridad interna.

## 4. Descripción del ambiente de EJECUCIÓN (en producción)

**Paquetes y Librerías Utilizadas:**
- **Kubernetes (kubectl):** Versión 1.21
- **MySQL:** Versión 8.0
- **Nginx:** Versión 1.19

### Configuración de IP y Puertos:

La configuración de los servicios en Kubernetes está manejada mediante los manifiestos YAML.

### Cómo se lanza el servidor:

#### 1. Acceso al clúster Kubernetes:

Accedemos al clúster en AWS y verificamos que todos los servicios estén corriendo correctamente.

#### 2. Despliegue del servicio:
```bash
kubectl apply -f all.yaml
```

## 5. Referencias:
- Despliegue de WordPress en AWS EKS: https://github.com/st0263eafit/st0263-242/tree/main/eks-wp
- Documentación oficial de Kubernetes: https://kubernetes.io/docs/home/
- Documentacion oficial de drupal bitnami https://hub.docker.com/r/bitnami/drupal/



Aquí tienes el contenido en formato Markdown:

```markdown
# Despliegue desde Cero

Una vez que todo esté eliminado, aquí tienes el orden recomendado para ejecutar los comandos y volver a desplegar todo.

## Instalar el Controlador NGINX Ingress

Este paso configura el balanceador de carga y el enrutamiento de tráfico. Usa Helm para instalar el controlador NGINX Ingress:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

## Crear el ClusterIssuer para Let's Encrypt

Define el ClusterIssuer para que Cert-Manager pueda emitir certificados desde Let’s Encrypt:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: samuelacostaaristizabal2@gmail.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx
```

Aplica el ClusterIssuer:

```bash
kubectl apply -f clusterissuer.yaml
```

## Desplegar los Recursos de la Aplicación (Deployments y Services)

Define y despliega los recursos de la aplicación en el archivo `all.yaml` o separadamente:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal
spec:
  replicas: 3
  selector:
    matchLabels:
      app: drupal
  template:
    metadata:
      labels:
        app: drupal
    spec:
      containers:
        - name: drupal
          image: drupal:latest
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: drupal-service
spec:
  type: ClusterIP
  selector:
    app: drupal
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drupal-db
  template:
    metadata:
      labels:
        app: drupal-db
  spec:
    containers:
      - name: mariadb
        image: mariadb:latest
        env:
          - name: MYSQL_ROOT_PASSWORD
            value: "yourpassword"
          - name: MYSQL_DATABASE
            value: "drupal"
          - name: MYSQL_USER
            value: "drupaluser"
          - name: MYSQL_PASSWORD
            value: "drupalpassword"
        ports:
          - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    app: drupal-db
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

Aplica los recursos de aplicación:

```bash
kubectl apply -f all.yaml
```

## Crear el Ingress y el Certificado

Define el Ingress con la configuración TLS y el Certificate que usará el ClusterIssuer para generar el certificado TLS:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: drupal-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: reto2.crazycookies.fun
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: drupal-service
                port:
                  number: 80
  tls:
    - hosts:
        - reto2.crazycookies.fun
      secretName: tls-secret
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: drupal-tls
  namespace: default
spec:
  secretName: tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: reto2.crazycookies.fun
  dnsNames:
    - reto2.crazycookies.fun
```

Aplica el Ingress y el Certificate:

```bash
kubectl apply -f ingress-and-cert.yaml
```

## Resumen de Comandos en Orden

### Instalar el controlador NGINX Ingress:

```bash
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

### Crear el ClusterIssuer para Let's Encrypt:

```bash
kubectl apply -f clusterissuer.yaml
```

### Desplegar los Deployments y Services de la aplicación:

```bash
kubectl apply -f all.yaml
```

### Configurar el Ingress y el Certificate:

```bash
kubectl apply -f ingress-and-cert.yaml
```
```


kubectl apply -f drupal-efs-pv.yaml

kubectl apply -f drupal-efs-pvc.yaml


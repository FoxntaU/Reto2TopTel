# Instrucciones para la instalación de Drupal en Kubernetes con almacenamiento EFS y certificación TLS

Instalar el Ingress NGINX Controller
Este controlador gestiona el tráfico de entrada en el clúster de Kubernetes.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
kubectl get pods -n ingress-nginx
```
Configura un PV con capacidad de 5Gi y con el acceso ReadWriteMany usando EFS.
```bash
yaml
Copiar código
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-efs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-0e2dc5966adbfb99c
```
```bash
kubectl apply -f drupal-efs-pv.yaml
```

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi


```
Define el PVC para reclamar el volumen EFS previamente creado.

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi

```

```bash
kubectl apply -f drupal-efs-pvc.yaml

```
Crea un PVC con el acceso ReadWriteOnce y una capacidad de 1Gi.
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mariadb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```

```bash
kubectl apply -f db-pvc.yaml
```
Instalar Cert-Manager para la gestión de certificados TLS
Cert-Manager gestiona los certificados TLS en el clúster.

```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.yaml

```

```bash
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

```bash
kubectl apply -f clusterissuer.yaml

```


```bash
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

```bash
kubectl apply -f drupal-certificate.yaml

```
Aplicar los manifiestos de configuración para la aplicación Drupal
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal
spec:
  replicas: 1
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
          image: bitnami/drupal:latest
          ports:
            - containerPort: 8080
          env:
            - name: BITNAMI_DEBUG
              value: "true"
            - name: DRUPAL_DATA_TO_PERSIST
              value: "sites/ themes/ modules/ profiles/"
            - name: DRUPAL_SKIP_BOOTSTRAP
              value: "no"
            - name: DRUPAL_HASH_SALT
              value: "89f19dbfe6df8d936305a84c0ef632e605dbcd35497a62bf03e6d7faca3ae3f0"
            - name: DRUPAL_CONFIG_SYNC_DIR
              value: "/var/www/html/sites/default/files/config"
            - name: DRUPAL_DATABASE_HOST
              value: 'db-service'
            - name: DRUPAL_DATABASE_NAME
              value: "drupal"
            - name: DRUPAL_DATABASE_USER
              value: "drupaluser"
            - name: DRUPAL_DATABASE_PASSWORD
              value: "drupalpassword"
          volumeMounts:
            - name: drupal-files
              mountPath: /var/www/html/modules
              subPath: modules
            - name: drupal-files
              mountPath: /var/www/html/profiles
              subPath: profiles
            - name: drupal-files
              mountPath: /var/www/html/sites
              subPath: sites
            - name: drupal-files
              mountPath: /var/www/html/themes
              subPath: themes
            - name: drupal-files
              mountPath: /var/www/html/sites/default/files/config
              subPath: config
      volumes:
        - name: drupal-files
          persistentVolumeClaim:
            claimName: drupal-efs-pvc
---
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
          image: bitnami/mariadb:latest
          env:
            - name: MARIADB_ROOT_PASSWORD
              value: "your_root_password"
            - name: MARIADB_DATABASE
              value: "drupal"
            - name: MARIADB_USER
              value: "drupaluser"
            - name: MARIADB_PASSWORD
              value: "drupalpassword"
          ports:
            - containerPort: 3306
      volumes:
        - name: mariadb-storage
          persistentVolumeClaim:
            claimName: mariadb-pvc
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
      targetPort: 8080
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

```bash
kubectl apply -f all.yaml
```

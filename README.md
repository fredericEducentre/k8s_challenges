# k8s_challenges

TODO change all names to something better

# Installation de Minikube

Nécessite qu'un système de virtualisation soit installé

https://minikube.sigs.k8s.io/docs/start/
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

# Configuration de Minikube
```
minikube start
virsh list
```

Commandes à connaître :
```
kubectl apply -f <nom_du_fichier>.yaml
kubectl delete -f <nom_du_fichier>.yaml
kubectl get deployment/services/pods
```

Types de fichier: Deployment, Service et PersistentVolumeClaim

Structure d'un fichier deployment.yaml k8s :

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <nom>-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <nom>
  template:
    metadata:
      labels:
        app: <nom>
    spec:
      containers:
      - name: <nom>
        image: <image_docker>
        imagePullPolicy: Never (optionnel, si image personnalisée)
        ports:
        - containerPort: <PORT ex: 80>
```

Structure d'un fichier service.yaml k8s :
```
apiVersion: v1
kind: Service
metadata:
  name: <nom>-service
spec:
  type: NodePort
  ports:
    - port: <PORT>
      nodePort: <PORT entre 30000 et 3????>
  selector:
    app: <nom>
```

Structure d'un fichier pvc.yaml k8s :
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <nom>-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: <Gi ex: 1Gi>
```

Structure d'un fichier pvc.yaml k8s :
```
apiVersion: batch/v1
kind: Job
metadata:
  name: <nom>-job
spec:
  template:
    spec:
      containers:
      - name: <nom>-container
        image: <image_docker>
        command: ["/bin/sh"] (ou bash)
        args: ["-c", "php bin/console doctrine:migrations:diff -n --allow-empty-diff && php bin/console doctrine:migrations:migrate -n --allow-no-migration"] (ligne de commande)
        env: (variable d'environnement)
        - name: DATABASE_URL
          value: "mysql://root@mysql/khuit"
      restartPolicy: Never
  backoffLimit: 4
```

Lier le registry docker de minikube au terminal pour pouvoir construire une image au sein de minikube :
```
eval $(minikube -p minikube docker-env)
```

---

# Niveau 1 : HTML

Déployer un serveur nginx

Fichier de déploiement + service

# Niveau 2 : Custom HTML

Déployer un serveur nginx avec une image custom

Index.html (SPA bootstrap optionnel) + Dockerfile + Fichier de déploiement + Service

# Niveau 3 : REACT.JS

Déployer un serveur nginx avec une image custom d'un projet ReactJS

ReactJs project build + Dockerfile + Fichier de déploiement + Service

# Niveau 4 : PHP

Déployer un serveur nginx avec php-fpm

Index.php + Dockerfile + Fichier déploiement + Service

# Niveau 5 : SYMFONY + MySQL

Déployer un projet symfony avec MySql dans un container php

Projet symfony + Dockerfile symfony composer + Fichier déploiement symfony + Service symfony + Fichier déploiement MySQL + Fichier service MySQL + Fichier PVC MySQL

#1. Démarrer avec un projet Symfony

```
git clone https://github.com/hmicn/k8s-base-app.git
```

#2. Créer une image Docker avec le projet

Dockerfile
```
FROM php:8.1.28-fpm

# Installation des dépendances
RUN apt-get update && apt-get install -y libpq-dev curl
RUN docker-php-ext-install pdo pdo_mysql

# Copier les fichiers de l'application
COPY . /var/www/html

# Définir le répertoire de travail
WORKDIR /var/www/html

# Installer Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN curl -1sLf 'https://dl.cloudsmith.io/public/symfony/stable/setup.deb.sh' | bash
RUN apt install -y symfony-cli

# Exposer le port
EXPOSE 8000

CMD ["symfony", "server:start", "--no-tls"]
```

Construire l'image Docker:
```
sudo docker build -t symfony-app:latest .
```

#3. Pousser l'image Symfony Docker local vers le registry de minikube

```
eval $(minikube -p minikube docker-env)
sudo docker build . -t symfony-app:latest
```

#4. Deploiement et service Symfony

symfony-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: symfony-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: symfony
  template:
    metadata:
      labels:
        app: symfony
    spec:
      containers:
      - name: symfony
        image: symfony-app
        imagePullPolicy: Never
        ports:
        - containerPort: 8000
```

symfony-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: symfony-service
spec:
  type: NodePort
  ports:
    - port: 8000
      nodePort: 30000
  selector:
    app: symfony
```

#5. Deploiement, service et volume MySQL

mysql-deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
         - name: MYSQL_ALLOW_EMPTY_PASSWORD
           value: "true"
         - name: MYSQL_DATABASE
           value: "khuit"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```

mysql-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None

```

mysql-pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

#6. Apply all files

```
kubectl apply -f NOM_DU_FICHIER.yaml
```

Une fois tous lancer faire un logs
```
kubectl get pods
```

```
minikube service symfony-service
```


#7. Réaliser une migration / lancer un job

symfony-migration-job.yaml
```
apiVersion: batch/v1
kind: Job
metadata:
  name: symfony-migration-job
spec:
  template:
    spec:
      containers:
      - name: symfony-migration-container
        image: symfony-app:latest
        command: ["/bin/sh"]
        args: ["-c", "php bin/console doctrine:migrations:diff -n --allow-empty-diff && php bin/console doctrine:migrations:migrate -n --allow-no-migration"]
        env:
        - name: DATABASE_URL
          value: "mysql://root@mysql/khuit"
      restartPolicy: Never
  backoffLimit: 4
```

```
kubectl apply -f symfony-migration-job.yaml
kubectl get jobs
kubectl describe job symfony-migration-job
kubectl logs -f job/symfony-migration-job
```

#8. Minikube dashboard

```
minikube dashboard
```

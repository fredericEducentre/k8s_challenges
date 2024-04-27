git clone https://github.com/hmicn/k8s-base-app.git
Changer le fichier .env ligne 27 :
DATABASE_URL="mysql://root@mysql:3306/khuit?serverVersion=mariadb-10.3.31"
docker build . -t symfony-app
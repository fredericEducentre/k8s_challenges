docker build . -t symfony-app
Changer le fichier .env ligne 27 :
DATABASE_URL="mysql://root@mysql:3306/khuit?serverVersion=mariadb-10.3.31"

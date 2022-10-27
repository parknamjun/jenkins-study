
* Maria db
docker run --detach --name mariadb -p 3306:3306 --env MARIADB_USER=gitea --env MARIADB_PASSWORD=gitea --env MARIADB_ROOT_PASSWORD=root mariadb:latest

* Gitea

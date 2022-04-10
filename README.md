# agl-bpms-project-guide

## Prequisities
- Docker
- Nginx
- Node <= 14
## Docker Configuration
- Run the following commands to create and start two containers of  PostgreSQL databases.
```
$ docker run -it --name agl-bpms -d -e POSTGRES_PASSWORD="password" -p 5432:5432 -v /var/lib/docker-db/postgresql/data:/var/lib/postgresql/data postgres
```

Make sure you have to run this command only once in life time. 
After that you have to run the following command to start the container of databases:
```
$ docker start <container_name>
```
OR
```
$ docker start <container_id>
```
Example:
```
$ docker start agl-bpms
 ```
 ##### Some Docker Commands(optional)
- To see the full list of created containers(both running and stopped) 
```
$ docker ps -a
```

- To see the full list of created containers(only running ) 
```
$ docker ps
```
- To remove container,run the following:
```
$ docker rm <id> 
```
or 
```
$ docker rm <container_name> 
```
- To see the running information of running processes and PID:
```
$ netstat -tunlp
```
- To kill any runnning process
```
$ kill -9 <PID>
```
## Postgres Database Import

- To create the core database in `agl-bpms-auth` project:
```
$ yarn db:create
```

- Before entering the container , run the following command to copy the `pgsql` file of PostgreSQL from HOST OS into the container OS:
```
$ docker cp <path to pgsql file> <container_id>:<path to backup folder inside container where the pgsql file will be saved>
```

For Example: 
```
$ docker cp /home/sajid576/Public/agl-bpms/agl_bpms_dbfile.pgsql  fa14cac10c04:/var/lib/postgresql/data/backup
```

- To enter into postgres container as `postgres` user,run the following command:
```
$ docker exec -it -u postgres postgres bash
```
- To enter into postgres container as `Root` user,run the following command:
```
$ docker exec -it  postgres bash
```

- After entering into container as `postgres` user, run the following command import the data of `pgsql` file to a database named `agl_bpms_dev` :
```
$ psql -U <username> <database_name>     <     <path to pgsql file inside  container>
```
`agl_bpms_dev` database is defined in `env` file.
 For Example:
 ```
 $ psql -U postgres agl_bpms_dev <    /var/lib/postgresql/data/backup/agl_bpms_dbfile.pgsql
 ```
 `agl_bpms_dev`  database should now contain the data that is in the .pgsql file.
- Then go to `agl-bpms-auth` repository, and run the following to migrate the database to project:
```
$ yarn db:migrate:all
```
## NGINX Configuration

- After installing nginx go to below directory:
```
 etc/nginx/sites-enabled/default
```
- Copy paste these codes: 

```
upstream back-end {

        server localhost:8000;

}

upstream storage {
	server localhost:8011;
}

```
Copy paste below code in the server block of  `default` file: 

```
location /core/ {

                proxy_pass http://back-end/;

}


location /customer-home/ {

                proxy_pass http://customer-home/;

}

location /storage/ {
		proxy_pass http://storage/;
}
```




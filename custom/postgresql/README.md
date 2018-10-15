# README
## Pre-Requisite
## Command
```
docker run --name postgresql -d --resstart=always -m 1G -p5432:5432 -e http_proxy=$http_proxy -e https_proxy=$https_proxy -e ftp_proxy=$ftp_proxy -e socks_proxy=$socks_proxy -e no_proxy=$no_proxy -ePOSTGRES_USER="<User>" -e POSTGRES_PASSWORD="<Password>" -ePOSTGRES_DB="<DB Name>" -v /docker/postgresql/data:/var/lib/postgresql/data postgres
```

<user> Username
<Password> Password

# Kelanjutan dari readme CICD
Buatlah EC2 lagi untuk database mysql

# Siapkan docker.io di EC2 database (dengan .pem yang berbeda dari EC2 webApp)
  ```
  =============EC2 DATABASE=============
  sudo apt-get update
  sudo apt-get install docker.io
  
  sudo groupadd docker
  sudo usermod -aG docker $USER
  newgrp docker
  ```
# Pada web AWS, tambahkan inbound pada EC2 database
 - port 3306
 - sourcre ipv4 anywhere (temporary)
 - source seharusnya dari IP public EC2 web app, namun untuk sementara anywhere

# Docker run
```
=============EC2 DATABASE=============
docker run --name altaDB -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345 -d mysql:latest
```
- name : nama container
- MYSQL_ROOT_PASSWORD : password database

# Database sudah terbentuk, dapat dicek dengan Mysql Workbench
- add new connection
- input nama connection
- input hostname dengan IP public EC2 database
- Port 3306
- input password
- test connection
- Apabila berhasil connect, buat schema baru (alta_db)

# Update file .yml pada local
```
============CONTOH===============
name: Deploy to EC2
on: [push]
jobs:

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - name: Deploy to EC2 by using SSH
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        port: ${{ secrets.PORT }}
        script: |
          if [ ! -d "/home/ubuntu/app" ]
          then
            git clone git@github.com:FarrasT-1997/ec2.git /home/ubuntu/app
          fi
          cd /home/ubuntu/app
          git pull origin master
          docker stop myApp
          docker rm myApp
          cd program
          docker build -t my-app:latest .
          docker run -d -e "HTTP_PORT=:80" -e "CONNECTION_STRING=root:12345@tcp(13.59.211.127:3306)/alta_db?charset=utf8mb4&parseTime=True&loc=Local" -p 80:80 --name myApp my-app:latest
```
NOTE :
- root = nama mysql (default)
- 12345 = password database
- 13.59.211.127  = IP public EC2 database
- 3306 = port database 
- alta_db = nama schema

# Edit program main.go agar dapat menampilkan data apabila sukses deploy
- program dapat dilihat pada repository saya "ec2"

# git push program
 Apabila CI/CD berhasil namun pada pada EC2 web app program tidak ter-update dapat lakukan build program ulang
 ```
 sudo apt-get install golang
 go build -o program
 
 git add . -A
 git stash
 ```
# Jika push berhasil deploy, dapat ditest melalui browser
- http://public-ip-ec2-webApp/user
- source security group inbound EC2 database dapat diubah dari anywhere ke IP public EC2 webApp

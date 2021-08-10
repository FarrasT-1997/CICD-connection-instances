# CICD-RDS

CI/CD AWS

1. Buat direktori .github/workflow atau menggunakan template dari github
   - template github : pilih actions dan pilih salah satu workflow (contoh greetings)
  
2. Path workflow git pull ke local
3. Pada direktori .github/workflow buat file .yml lagi

----------CONTOH-----------------
```
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
          docker run -d -e "HTTP_PORT=:80" -p 80:80 --name myApp my-app:latest
```

4. Sebelum push ke github, siapkan EC2, install docker, buat keygen EC2
```
sudo apt-get update
sudo apt-get install docker.io

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

ssh-keygen
```
5. Pada github, pilih settings (di samping insights) -> secrets. Masukkan HOST, USERNAME, KEY & PORT
  - HOST : IP public EC2
  - USERNAME : ubuntu
  - KEY : copy key file .pem (tanpa "%")
  - PORT : 22

6. Masuk ke settings (klik foto profil -> settings) -> SSH & GPG keys.
  - New key SSH
  - Input code SSH dari ssh-keygen EC2
  ```
  ls -a
  cd .ssh
  cat id_rsa.pub
  ```
7. Push program dari local ke github. Maka github ada secara otomatis EC2 akan git pull dari master
- apabila deploy gagal karena EC2 tidak bisa git pull, dapat melakukan git pull manual di EC2
```
git clone ...
```
8. Pastikan security group inbound EC2 telah ditambahkan port web app (contoh :80 ipv4 anywhere)

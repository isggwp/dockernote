### Command untuk pull image dari docker hub

`docker pull mysql:latest`

`mysql` adalah nama dari image yang bisa kita dapatkan dari docker hub. biasanya nama image ini diikuti oleh nama versi seperti `mysql:01-debian` dll
<br>
<br>

### Command untuk membuat container

`docker container create --name contoh-mysql --publish 3333:3306 mysql:latest`

nah `--publish` ini untuk export port. port `3333` diatas adalah port dari host yang di akses dari computer client dan port `3306` adalah port yg di akses dari dalam container yg kita buat. hal ini biasa disebut **_Port Fordwarding_**

<br>

### dan Command untuk menjalankan dan menghentikan Container
`docker container start contoh-mysql`

`docker container stop contoh-mysql`

<br>
<br>

## Melihat list docker container yg ada 

`docker container ls -a`

nanti akan muncul list container yg aktif maupun tidak aktif. jika tidak aktif biasanya column port akan kosong
<br>
<br><br>

## Menambahkan environtment variabel di dalam docker

kita dapat menambahkan env kedalam docker misal kita memiliki env:
```
username: indra
password: admin1234
```

kita dapat menggunakan command seperti ini


`docker container create --name contoh-mysql --publish 3333:3306 --env username=indra --env password=admin1234 mysql:latest`

sayangnya kita tidak dapat melihat env ini saat kita menggunakan `docker container ls -ls a`

<br><br>
## Container Stats

digunakan untuk melihat resource aktivity. command nya cukup dengan <br>
``` docker container stats```
<br>
<br>

 ### Resource limit docker

kita bisa menambahkan resource limit docker diantaranya memory a.k.a `--memory` dan CPU a.k.a `--cpu` berikut contohnya

```
docker container create --name contoh-mysql --publish 3333:3306 --env username=indra --env password=admin1234 --memory=100m --cpus=0.5 mysql:latest
```

### Bind Mount

adalah sharing folder atau cara agar folder di dalam container docker dapat mengakses file atau folder yg ada di host atau komputer tempat docker dijalankan. 

Dengan menggunakan bind mount, kita dapat membagikan data antara host dan container Docker, sehingga memungkinkan kita untuk melakukan pengembangan aplikasi tanpa harus melakukan copy data antar lingkungan.

berikut contoh command untuk Bind Mount

```
docker container create --name contoh-mysql \
 --publish 3333:3306 \ 
 --env username=indra \
 --env password=admin1234 \ 
 --mount "type=bind,source=/host_folder,destination=/,readpnly" \
 --memory=100m --cpus=0.5 mysql:latest
```
<br>
<br>

### Docker Volume

Dalam Docker, penggunaan volume lebih direkomendasikan daripada penggunaan --mount karena volume memberikan fleksibilitas dan konsistensi yang lebih besar dalam manajemen data persisten.

Perbedaan utama antara volume dan --mount adalah bagaimana data persisten diatur dan diakses. Volume adalah cara standar Docker untuk membuat dan mengelola data persisten dalam container. Volume independen dari container dan dapat digunakan oleh lebih dari satu container pada saat yang sama. Ini memungkinkan untuk lebih mudah membuat container yang terisolasi dan dapat dipindahkan, karena data tidak terikat pada file sistem host.

Di sisi lain, --mount memungkinkan Anda untuk mengikat direktori atau file dari host ke dalam container, dan juga dapat digunakan untuk mengikat volume Docker ke dalam container. Namun, --mount membutuhkan lebih banyak opsi konfigurasi dan tidak selalu mudah digunakan.

```
docker volume ls
```
untuk melihat list volume yang ada di dalam docker atau yang tergenerate secara otomatis. karena pada dasarnya volume akan tergenerate secara default.

untuk membuat volume dapat menggunakan command 

```
docker volume create mysql_volume_example
```

kita juga dapat menghapus volume. dan pastikan volume yang kita akan hapus juga sudah tidak digunakan. untuk menghapus volume kita dapat menggunakan perintah

```
docker volume rm mysql_volume_example
```

dan kita dapat menggunakan perimantah `ls` untuk melihat list volume existing yang ada

```
docker volume ls
```


### Container Volume

Tahap selanjutnya kita akan mengaitkan volume dengan container. sebagaimana `bind` yang bertujuan sharing folder. `volume` juga sama namun source folder bukan berada pada host, melainkan di manage di dalam docker itu sendiri. disebuah tempat yg disebut volume. berikut perintah untuk membuat `volume`. kurang lebih sama seperti `bind` namun berbeda pada `type=volume` dan `source` yang berisi nama `volume` yg akan digunakan.

```
docker container create 
--name contoh-mysql \
--publish 3333:3306 \ 
--env username=indra \
--env password=admin1234 \ 
--mount "type=volume,source=mysql_volume_example,destination=/sb/data,readonly" \
--memory=100m --cpus=0.5 mysql:latest
```


### Backup Volume
sejauh ini docker belum menyediakan backup secara otomatis atau feature yang bisa kita pakai untuk membackup dari volume yang telah kita buat. kita butuh script tambahan untuk backup. agak sedikit complex, dan script yang akan kita jalankan kurang lebih menjalankan

1. matikan container yang menggunakan volume target. ya tentu saja saat kita ingion membackup pastikan tidak ada perubahan data saat datanya kita backup. 

2. buat container baru dengan 2 mount. yaitu dengan volume yang ingin kita backup, dan dengan bind mount dari sistem host.

3. lakukan backup dengan mengarchive isi dari volume, lalu kita simpan kedalam bind folder. 

4. nice. isi backup sekarang ada di dalam sistem host kita

5. remove container yang kita gunakan untuk melakukan backup.

kurang lebih script nya seperti ini

```
# stop container that consume volume
docker container stop mysql_cont_example


# create container spesificly for backup activity. we can use All docker image. itws just for copy paste file and folder yeah backup activity. we choose alphine because its small

docker container create 
--name backup_mysql_container \
--mount "type=bind,source=/Users/indrasusila/Documents/my-project/isggwp-project,destination=/backup" \
--mount "type=volume,source=mysql_volume_example,destination=/sb/data,readonly" \
--memory=100m \
--cpus=0.5 \
alphine:latest

# Start backup container 
docker container start backup_mysql_container

# And then, we need jump to inside backup_mysql_container and run bash for backup or archive the Datas

docker container exec -i -t backup_mysql_container /bin/bash

# setelah masuk kita pastikan melihat adanya folder/backup dengan command ls -a . selain itu pastikan juga folder yang ingin kita lakukan backup. dan dalam hal ini adalah /data. setelah sudah ok lakukan backup dengan tar command


tar cvf /destination_file.tar.gz /folder_mana_yg_ingin_kita_backup

# example
tar cvf /backup/backup_data.tar.gz /data


# kita harus tunggu sampai proses backupnya selesai. setelah itu stop container backupnya dan remove.

docker container stop backup_mysql_container

docker container rm backup_mysql_container


# selanjut nya jalankan docker container nya untuk menguji

docker container start mysql_cont_example

```
<br>

Ya, itu adalah cara kita untuk mengceate backup secara mannual. namun kita bisa lebih mudah mengcreate lalu menghapus doker container setelah kita pakai, khususnya untuk backup, dalam hal ini kita bisa menggunakan docker `run` dan option `--rm`. kurang lebih seluruh perintah diatas dapat disederhanakan menjadi seperti ini.

<br>

```
#Matikan container yg menggunakan volume target.
docker container stop mysql_volume_container

docker container run --rm \
--name alphine \
--mount "type=bind,source=/Users/indra/Documents/backup-mysql,desitination=/backup" \
--mount "type=volume,source=mysql_volume,destination=/backup \
alphine:latest \
tar cvf /backup/data.tar.gz /data \


docker container start mysql_volume_container

"
```

## Restore Volume
hal sebelumnya kita telah melakukan backup ke file host. namun terkadang Kita mengharuskan untuk membeackup data pada tempat yang lebih amaan misalnya cloud storage.


setelah melakukan backup di tempat yang aman, kita dapat melakukan restore ke volume yang baru. serta memastikan bahwa data yg kita restore dan backup telah sesuai dan tidak corrupt

beberapa tahapan dalam melakukan restore.

1. buat volume baru untuk lokasi restore dan backup.
2. buat satu container baru dengan 2 mount. 1 volume baru untuk restore backup. dan 1 bind mount folder dari host yang berisi file backup.
3. Lakukan restore menggunakan container dengan mengekxtrak isi backup file ke dalam volume
4. delete container yg kita gunakan untuk restore
5. dan volume baru tersebut bisa kita gunakan pada container yg berisi file restore.

kira kira berikut command nya
<br>
hal berbeda dan penting yang perlu kita ingat saat restore adalah, membuat volume baru untuk restore.

```

docker volume create mysql_restore

docker container run --rm \
--name alphine_restore \
--mount "type=bind,source=/Users/indra/Documents/backup-mysql,desitination=/backup" \
--mount "type=volume,source=mysql_restore,destination=/data"  \
alphine:latest \
bash -c "cd /backup && tar xvf /backup/data.tar.gz --stripe 1" \

```

### Docker Network

secara default, setiap container di docker saling terisolasi satu sama lain. nah docker memilkiki fiture container yg memungkinkan setiap container dapat berkomunikasi antara satu dengan yang lain.

docker network ini dapat digunakan  untuk membuat jaringan didalam docker. <br>
Dan jika terdapat beberapa container di dalam satu netwok yang sama, maka secara otomatis container tersebut dapat saling berkomunikasi.

#### Membuat Network

ketika kita membuat network, kita perlu menentukan driver apa yang harus kita pakai. https://docs.docker.com/network/

setiap network memiliki syarat dan kebutuhannya masing-masing. secara general terdapat 2 network yg sering dipakai. bridge dan host.

- _**Bridge**_ yaitu driver yang digunakan untuk membuat network secara virtual. yang memungkinkan container yang terkoneksi _bridge_ network saling terkomunikasi.

- _**Host**_ yaitu driver yang digunakan untuk membuat network yang sama dengan sistem host. namun driver Host hanya dapat berjalan di docker Linux, dan tidak dapat berjalan di windows maupun Mac.

- _**None**_ . secara default saat membuat container akan di definisikan menjadi _none_, yang artinya terisolasi dan tidak dapat berkomunikasi antar kontainer.


untuk melihat existing network kita dapat menggunakan command

```
docker network ls
```

untuk membuat network kita dapat menggunakan perinta

```
docker network create --driver bridge contoh_bridge_network
```

kita dapat membuat network tanpa `--driver`. dan default value saat create network tanpa `--driver` adalah bridge.

<br> <br>
### Menghapus Network

untuk menghapus container kita dapat menggunakan comand `rm`

```
docker network rm contoh_bridge_network
```

namun sebagaimana volume, network tidak bisa dihapus jika masuh digunakan. kita harus menghapus container yang menggunakan network tersebut terlebih dahulu, baru setelah itu menghapus network.


### Container network / mengkoneksikan container di dalam network

setelah selesai membuat network, kita dapat menambahkan container kedalam network terdapat beberapa cara untuk melakukan nya.

- . menambahakan settingan **`--network`** saat membuat container

```
docker container create 
--name backup_mysql_container \
--network contoh_bridge_network \
--mount "type=bind,source=/Users/indrasusila/Documents/my-project/isggwp-project,destination=/backup" \
--mount "type=volume,source=mysql_volume_example,destination=/sb/data,readonly" \
--memory=100m \
--cpus=0.5 \
alphine:latest

```

<br>
### Mendisconnect Container dari network
kita bisa mendisconnect network dan container dengan command

```
docker network disconnect contoh_bridge_network mysql_container
```

sebagai mana disconnect, kita juga bisa dapat menambahkan kembali container untuk connect dengan network yg ada. kita dapat menggunakan comman `connect` yaitu sebagai contoh

```
docker network connect contoh_bridge_network mysql_container
```
<br>

## Inspect
kadang kita ingin melihat informasi yang terdapat pada container seperti env, port, dll.  docker memiliki vitur inspect. yang memungkinkan kita dapat melihat informasi secara detail yang ada pada container. dengan inspect kita dapat melihat detail dari container, image, volume, dan network. kita dapat menggunakan command:

```

# inspect Image
docker image inspect nama_image

# inspect container
docker container inspect nama_container

# inspect volume
docker volume inspect nama_volume

# inspect network
docker network inspect nama_network

```

<br> <br>

## Prune

Saat menggunakan docker, adakalanya kita ingin membersihkan hal-hal yang sudah tidak digunakan lagi, seperti container yang sudah stop, atau volume/ network/ atau image yang sudah tidak digunakan oleh container. <br>
docker memiliki fitur untuk membersihkan itu semua secara otomatis yang dinamai _**Prune**_. dan hampir semua command di docker mendukung perintah _prune_ seperti container, network, volume mendukung prune. berikut contoh perintah prune

```
# Menghapus container yg sudah stop
docker container prune

# Menghapus image yang sudah tidak digunakan container
docker image prune

# Menghapus seluruh network yang sudah tidak digunakan container
docker network prune

# Menghapus seluruh volume yang sudah tidak digunakan container
docker volume prune


# Menghapus seluruh container, network, volume yang sudah tidak digunakan container dalam sekaligus
docker system prune
```





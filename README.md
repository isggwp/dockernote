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
# chroot-enviroment
### Bahasan mengenai CHROOT
pada modul kali ini saya akan menjelaskan mengenai chroot enviroment pada karnel Linux Ubuntu. sebelumnya chroot ialah sebuah perintah dimana memunkinkan kita untuk membuat lingkungan yang terpisah dan aman pada karnel Linux, chroot akan membuat sub directory pada directory untuk menyimpan file konfigurasi sistemnya.

### chroot jail
Seperti dijelaskan di atas, `chroot` menciptakan lingkungan yang terisolasi, yang dikenal sebagai `chroot jail`. Proses yang berjalan di lingkungan ini memiliki direktori root dan sistem file yang berbeda. Proses ini dicegah untuk mengakses apa pun di sistem di luar chroot jail.

Untuk membuat chroot jail, Anda membuat direktori untuk bertindak sebagai root directory. Kemudian, Anda menambahkan program dan komponen sistem yang Anda perlukan untuk menjalankan proses apa pun yang ingin Anda uji di chroot enviroment.

Saat anda melakukan chroot login terhadap direktori yang Anda buat, Anda kemudian dapat menggunakannya sebagai sistem yang berfungsi sendiri. Direktori yang Anda buat bertindak sebagai direktori root, jadi apa pun yang beroperasi di dalamnya dibatasi untuk chroot.

chroot enviroment memberi Anda ruang yang bersih dan terpisah untuk menjalankan proses. Ini memastikan bahwa apa pun yang berjalan di chroot jail tidak terpengaruh oleh sistem file utama. Demikian pula, chroot jail tidak dapat memengaruhi sistem file utama. 

### Tujuan chroot
Alasan utama untuk membuat chroot enviroment adalah untuk menguji proses dalam isolasi. Ada dua skenario utama di mana Anda mungkin ingin menguji secara terpisah: 
- Skenario pertama adalah menguji aplikasi yang tidak tepercaya. Menjalankannya di chroot jail memungkinkan Anda menjalankan aplikasi tanpa mengizinkannya mengakses seluruh sistem file Anda. 
- Alasan lain adalah untuk menguji aplikasi, perintah, atau serangkaian perintah di lingkungan terpencil. Dengan chroot enviroment, Anda menjamin bahwa proses atau perintah berjalan dalam sistem file yang bersih dan mudah direproduksi. 

### buat lingkungan uji
Disini saya akan memaparkan cara membuat lingkungan uji chroot enviroment pada karnel Linux Ubuntu 20.04 LTS Focal-fossa. hal pertama yang akan kita lakukan adalah membuat Direktori chroot, sebagai Direktori isolasi, dengan perintah.

```bash
mkdir ~/chroot-env
```

disini kita akan menginstall sistem yang akan diinstall pada Direktori chroot kita menggunakan perintah `debootstrap`

```bash
sudo apt-get install debootstrap -y
```

lalu disini kita akan menginstall system `buster` pada Debian, anda juga bisa melakukan install menggunakan system `focal`.

```bash
sudo debootstrap buster ~/chroot-env
```
tunggu sampai proses installasi selesai. langkah selanjutnya ujicoba chroot enviroment menggunakan perintah `chroot`
 
 ```bash
 sudo chroot ~/chroot-env /bin/bash
 ```
 ```
 root@localhost:/#
 ```
 
ketik `exit` untuk keluar dari chroot enviroment. disini kita sudah bisa mengakses chroot enviroment kita, akan tetapi kita belum bisa menjalankan beberapa perintah pada enviroment kita. sebelumnya buat user login untuk chroot enviroment kita dengan perintah.
 
 ```bash
 sudo useradd example-user -p root -G sudo
 sudo mkdir /home/example-user
 ```
 ```bash
 sudo apt-get install sudo -y
 ```
  mount file system ke chroot enviroment system
  ```bash
  sudo mount /dev ~/chroot-env/dev
  sudo mount /proc ~/chroot-env/proc
  sudo mount /sys ~/chroot-env/sys
  ```
  
  ### install dan konfigurasi schroot
  
  perintah schroot memungkinkan kita memberi akses user terbatas kepada suatu chroot enviroment. install `schroot` dengan perintah.
  
  ```bash
  sudo apt-get install schroot -y
  ```
  
  konfigurasi schroot file dengan perintah
  
  ```bash
  sudo vi /etc/schroot/schroot.conf
  ```
  
  tambahkan beberapa baris intruksi pada file konfigurasi diatas
  
  ```
# add chroot enviroment test
[buster-env]
description=Debian Buster
directory=/home/example-user/chroot-env
users=example-user
groups=sbuild
root-groups=root
aliases=buster
```

restart service schroot dengan perintah.

```bash
sudo systemctl restart schroot
```

```bash
sudo schroot -c buster
```
```
root@localhost:/#
```

Anda sekarang masuk ke chroot enviroment sebagai pengguna terbatas. Di sana, Anda dapat menjalankan program dan perintah dan menginstal paket seperti yang Anda lakukan pada sistem operasi biasa. 

### Hapus chroot enviroment

ketikkan perintah dibawah untuk menghapus chroot enviroment yang sebelumnya anda buat dengan perintah.
```bash
sudo umount ~/chroot-env/dev
sudo umount ~/chroot-env/proc
sudo umount ~/chroot-env/sys

sudo rm -rf ~/chroot-env
```

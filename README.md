# deploy_odoo_ubuntu

**Sebelum Mulai**

Login ke mesin Ubuntu Anda sebagai user sudo dan perbarui sistem ke paket terbaru:

sudo apt update && sudo apt upgrade

Install Git, Pip, Node.js dan tools yang dibutuhkan untuk build Odoo dan dependensi nya:
sudo apt install git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less
Membuat user untuk Odoo

Buat user sistem baru untuk Odoo bernama odoo12 dengan direktori home /opt/odoo12 menggunakan perintah berikut:

sudo useradd -m -d /opt/odoo12 -U -r -s /bin/bash odoo12

Anda dapat menggunakan nama apa pun untuk user Odoo selama Anda membuat user PostgreSQL dengan nama yang sama.
Install dan konfigurasi PostgreSQL

Instal paket PostgreSQL dari repositori default Ubuntu:

sudo apt install postgresql

Setelah instalasi selesai, buat user PostgreSQL dengan nama yang sama dengan user sistem yang dibuat sebelumnya, dalam kasus kami yaitu odoo12:

sudo su - postgres -c "createuser -s odoo12"

Install Wkhtmltopdf

Paket wkhtmltox menyediakan seperangkat alat baris perintah open source yang dapat merender HTML ke dalam PDF dan berbagai format gambar. Untuk mencetak laporan PDF, Anda memerlukan tool wkhtmltopdf.

Versi yang direkomendasikan untuk Odoo adalah 0.12.1 yang tidak tersedia di repositori resmi Ubuntu 18.04.

Unduh paket menggunakan perintah wget berikut:

wget https://builds.wkhtmltopdf.org/0.12.1.3/wkhtmltox_0.12.1.3-1~bionic_amd64.deb

Setelah unduhan selesai instal paket dengan mengetik:

sudo apt install ./wkhtmltox_0.12.1.3-1~bionic_amd64.deb

Install dan Konfigurasi Odoo

Kita akan menginstal Odoo dari repositori GitHub di dalam Python virtual environment. yang terisolasi.

Sebelum memulai dengan proses instalasi, beralihlah ke user odoo12:

sudo su - odoo12

Kemudian cloning Odoo 12 source code dari repository GitHub:

git clone https://www.github.com/odoo/odoo --depth 1 --branch 12.0 /opt/odoo12/odoo

Setelah source code diunduh, buat Python virtual environment baru untuk instalasi Odoo 12:

cd /opt/odoo12
python3 -m venv odoo-venv

Selanjutnya, aktifkan environment dengan perintah berikut:

source odoo-venv/bin/activate

Instal semua modul Python yang diperlukan dengan pip3:

pip3 install wheel
pip3 install -r odoo/requirements.txt

Jika Anda menemukan kesalahan kompilasi selama instalasi, pastikan Anda menginstal semua dependensi yang diperlukan yang tercantum di bagian Sebelum mulai pada atas artikel ini.

Nonaktifkan environment menggunakan perintah berikut:

deactivate

Buat direktori baru untuk custom addons:

mkdir /opt/odoo12/odoo-custom-addons

Beralih kembali ke pengguna sudo Anda:

exit

Selanjutnya, buat file konfigurasi, dengan menyalin file konfigurasi sampel yang disertakan:

sudo cp /opt/odoo12/odoo/debian/odoo.conf /etc/odoo12.conf

Buka file dan edit sebagai berikut:

sudo nano /etc/odoo12.conf

[options]
; This is the password that allows database operations:
admin_passwd = my_admin_passwd
db_host = False
db_port = False
db_user = odoo12
db_password = False
addons_path = /opt/odoo12/odoo/addons,/opt/odoo12/odoo-custom-addons

Jangan lupa untuk mengubah my_admin_passwd dengan kata sandi yang lebih aman dan mudah di ingat.
Membuat Unit File Systemd

Untuk menjalankan Odoo sebagai service (layanan sistem), kita perlu membuat file unit sistem di direktori/etc/systemd/system/.

Buka teks editor Anda dan paste konfigurasi berikut:

sudo nano /etc/systemd/system/odoo12.service

[Unit]
Description=Odoo12
Requires=postgresql.service
After=network.target postgresql.service
[Service]
Type=simple
SyslogIdentifier=odoo12
PermissionsStartOnly=true
User=odoo12
Group=odoo12
ExecStart=/opt/odoo12/odoo-venv/bin/python3 /opt/odoo12/odoo/odoo-bin -c /etc/odoo12.conf
StandardOutput=journal+console
[Install]
WantedBy=multi-user.target

Beri tahu systemd bahwa ada file unit baru dan start layanan Odoo dengan menjalankan:

sudo systemctl daemon-reload
sudo systemctl start odoo12

Periksa status layanan dengan perintah berikut:

sudo systemctl status odoo12

Outputnya akan terlihat seperti di bawah ini yang menunjukkan bahwa layanan Odoo aktif dan berjalan.

* odoo12.service - Odoo12
   Loaded: loaded (/etc/systemd/system/odoo12.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2018-10-09 14:15:30 PDT; 3s ago
 Main PID: 24334 (python3)
    Tasks: 4 (limit: 2319)
   CGroup: /system.slice/odoo12.service
           `-24334 /opt/odoo12/odoo-venv/bin/python3 /opt/odoo12/odoo/odoo-bin -c /etc/odoo12.conf

Aktifkan layanan Odoo untuk dimulai secara otomatis saat boot:

sudo systemctl enable odoo12

Jika Anda ingin melihat pesan yang dicatat oleh layanan Odoo, Anda dapat menggunakan perintah di bawah ini:

sudo journalctl -u odoo12

Test Instalasi Odoo

Buka browser Anda dan ketik:

http://<domain_anda_atau_IP_address>:8069

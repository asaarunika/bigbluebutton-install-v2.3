
![bbb-install.sh](images/bbb-install.png?raw=true "bbb-install.sh")

# bbb-install

Untuk menyiapkan server BigBlueButton server 2.3 (atau upgrade dari versi 2.3 sebelumnya), `bbb-install.sh` adalah skrip shell yang mengotomatiskan langkah-langkah penginstalan/upgrade (lihat [source](https://github.com/bigbluebutton/bbb-install/blob/master/bbb-install.sh) untuk melihat semua detailnya). Tergantung pada koneksi internet server, `bbb-install.sh` dapat sepenuhnya menginstal dan mengkonfigurasi server BigBlueButton untuk produksi dalam waktu kurang dari 30 menit.

Misalnya, untuk menginstal versi terbaru BigBlueButton 2.3 pada server 64-bit Ubuntu 18.04 baru dengan alamat IP publik, nama host (seperti `bbb.example.com`) yang ditetapkan ke alamat IP publik, dan alamat email (seperti `info@example.com`), masuk ke server melalui SSH dan jalankan perintah berikut sebagai root.

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -w -a -v bionic-23 -s bbb.example.com -e info@example.com
~~~

Perintah ini menarik versi terbaru `bbb-install.sh`, mengirimkannya ke penerjemah Bash shell, dan menginstal BigBlueButton menggunakan parameter yang disediakan:

  * `-w` menginstal uncomplicated firewall (UFW) untuk membatasi akses ke port TCP/IP 22, 80, dan 443, dan port UDP dalam range 16384-32768,
  * `-a` menginstal demo API (memudahkan untuk melakukan beberapa tes cepat di server), 
  * `-v bionic-23` menginstal versi terbaru BigBlueButton 2.3.x, 
  * `-s` menetapkan nama host server menjadi `bbb.example.com`, dan
  * `-e` memberikan alamat email untuk Let's Encrypt untuk menghasilkan sertifikat SSL yang valid untuk host.

Catatan: Jika server berada di belakang firewall eksternal -- seperti di belakang firewall perusahaan atau di belakang AWS Security Group -- Anda perlu mengonfigurasi firewall eksternal secara manual untuk meneruskan koneksi internet tertentu ke server BigBlueButton sebelum dapat meluncurkan ke klien.

Saat perintah di atas selesai, akan melihat pesan yang memberi URL uji untuk meluncurkan klien BigBlueButton dan bergabung dengan rapat yang disebut 'Demo Meeting'

~~~
# Warning: The API demos are installed and accessible from:
#
#    https://bbb.example.com
#
# and
#
#    https://bbb.example.com/demo/demo1.jsp  
#
# These API demos allow anyone to access your server without authentication
# to create/manage meetings and recordings. They are for testing purposes only.
# If you are running a production system, remove them by running:
#
#    sudo apt-get purge bbb-demo  
~~~

Buka URL di Chrome atau FireFox (browser yang disarankan). Akan melihat login untuk bergabung dengan rapat 'Demo Meeting'.

![bbb-install.sh](images/html5-join.png?raw=true "HTML5 Page")

Masukkan nama dan klik 'Gabung'. Klien BigBlueButton kemudian akan dimuat di browser dan meminta untuk bergabung dengan audio.

![bbb-install.sh](images/html5.png?raw=true "HTML5 Client")

Perhatikan halaman web dilayani melalui HTTPS. Browser sekarang memerlukan ini sebelum mengizinkan akses ke webcam, mikrofon, atau layar Anda (untuk berbagi layar) menggunakan komunikasi real-time (WebRTC) bawaan browser. Jika Anda mencoba menginstal BigBlueButton tanpa menentukan parameter `-s` dan `-e`, klien tidak akan memuat.

Nama host `bbb.example.com` dan alamat email `info@example.com` hanyalah contoh parameter. Bagian berikut memandu Anda melalui detail tentang penggunaan `bbb-install.sh` untuk menyiapkan/mengupgrade server BigBlueButton.

Setelah pengujian, dapat menghapus demo api dengan perintah `sudo apt-get purge bbb-demo`.  Kemudian, Anda dapat meningkatkan server ke rilis terbaru BigBlueButton 2.3 dengan menjalankan kembali perintah `bbb-install.sh`, dan menghilangkan `-a` untuk menginstal API demos.

Bagian berikut membahas lebih detail pengaturan baru server BigBlueButton 2.3.

## Persiapan

Sebelum menjalankan `bbb-install.sh`, kamu butuh

  * baca semua dokumentasi,
  * memastikan bahwa server memenuhi [minimal server requirements](http://docs.bigbluebutton.org/install/install.html#minimum-server-requirements), dan
  * konfigurasikan fully qualified domain name (FQDN), seperti `bbb.example.com`, yang resolves ke alamat IP eksternal server.

Untuk menyiapkan FQDN, beli domain name dari domain name registrar dan penyedia hosting web, seperti [GoDaddy](https://godaddy.com) atau [Network Solutions](https://networksolutions.com).  Setelah dibeli, ikuti langkah-langkah yang ditunjukkan oleh penyedia untuk membuat `A Record` untuk FQDN resolves ke alamat IP publik server Anda. (Periksa dokumentasi penyedia untuk detail tentang cara menyiapkan `A Record`.)

Dengan FQDN di tempat, kamu kemudian dapat meneruskan beberapa parameter tambahan ke `bbb-install.sh` untuk memilikinya:

  * meminta dan menginstal sertifikat 4096-bit TLS/SSL dari Let's Encrypt (we love Let's Encrypt),
  * instal firewall untuk membatasi akses hanya ke port yang diperlukan (recommended),
  * instal dan konfigurasikan [Greenlight](http://docs.bigbluebutton.org/greenlight/gl-overview.html) untuk menyediakan tampilan depan yang sederhana bagi pengguna untuk memungkinkan mereka mengatur rooms, hold online sessions, dan manage recordings (optional).  

Jika menginstal Greenlight, akan memiliki kemampuan untuk menjadi administrator [Greenlight administrator](http://docs.bigbluebutton.org/greenlight/gl-admin.html), memberi kemampuan mengelola akun pengguna.

Kode sumber lengkap untuk `bbb-install.sh` ada [disini](https://github.com/bigbluebutton/bbb-install).  Untuk memudahkan siapa saja menjalankan skrip dengan satu perintah, kami meng-host skrip versi terbaru di [https://ubuntu.bigbluebutton.org/bbb-install.sh](https://ubuntu.bigbluebutton.org/bbb-install.sh).


### Pilihan server

There are many hosting companies that can provide you with dedicated virtual and bare-metal servers to run BigBlueButton.  We list a few popular choices below (we are not making any recommendation here, just listing some of the more popular choices).

For quick setup, [Digital Ocean](https://www.digitalocean.com/) offers both virtual servers with Ubuntu 16.04 64-bit and a single public IP address (no firewall).  [Hetzner](https://hetzner.cloud/) offers dedicated servers with single IP address.

Other popular choices, such as [ScaleWay](https://www.scaleway.com/) (choose either Bare Metal or Pro servers) and [Google Compute Engine](https://cloud.google.com/compute/), offer servers that are set up behind network address translation (NAT).  That is, they have both an internal and external IP address.  When installing on these servers, the `bbb-install.sh` will detect the internal/external addresses and configure BigBlueButton accordingly.  

Another popular choice is [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2).  We recommend a `c5.2xlarge` or `c5a.2xlarge` (or larger) instance.  All EC2 servers are, by default, behind a firewall (which Amazon calls a `security group`).  You will need to manually configure the security group before installing BigBlueButton on EC2 and, in a similar manner, on Azure and Google Compute Engine (GCE).  (See screen shots in next section.)

Finally, if `bbb-install.sh` is unable to configure your server behind NAT, we recommend going through the [step-by-step instructions for installing BigBlueButton](http://docs.bigbluebutton.org/2.2/install.html) (going through the steps is also a good way to understand more about how BigBlueButton works).


### Configuring the external firewall

If you install BigBlueButton on a server behind a external firewall, such an Amazon's EC2 security group, you need to configure the external firewall to forward incoming traffic on the following ports:

  * TCP/IP port 22 (for SSH)
  * TCP/IP ports 80/443 (for HTTP/HTTPS)
  * UDP ports in the range 16384 - 32768 (for FreeSWITCH/HTML5 client RTP streams)

If you are using EC2, you should also assign the server an [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) to prevent it from getting a new IP address on reboot.

On Microsoft Azure, when you create an instance you need to add the following inbound port rules to enable incoming connections on ports 80, 443, and UDP port range 16384-32768:

![Azure Cloud ](images/azure-firewall.png?raw=true "Azure 80, 443, and UDP 16384-32768")

On Google Compute Engine, when you create an instance you need to enable traffic on port 80 and 443.

![Google Compute Engine 80-443](images/gce-80-443.png?raw=true "GCE 80 and 443")

After the instance is created, you need to add a firewall rule to allow incoming UDP traffic on the port range 16384-32768.

![Google Compute Engine Firewall](images/gce-firewall.png?raw=true "GCE Firewall")

We make a distinction here between the firewall installed with `-w` and the external firewall on a separate server.  Even with an external firewall, it is good practice to still install the UFW firewall on the BigBlueButton server.


### Installation videos

These videos are showing installation of BigBlueButton 2.2 (and will be updated now that 2.3 is released), but they still give a good overview of the steps.

Using Digital Ocean as an example, we put together this video to get you going quickly: [Using bbb-install.sh to set up BigBlueButton on Digital Ocean](https://youtu.be/D1iYEwxzk0M).

Using Amazon EC2, see [Install using bbb-install.sh on EC2](https://youtu.be/-E9WIrH_yTs).

## Command options

You can get help by passing the `-h` option.

~~~
Script for installing a BigBlueButton 2.3 (or later) server in about 30 minutes.

This script also supports installation of a coturn (TURN) server on a separate server.

USAGE:
    wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- [OPTIONS]

OPTIONS (install BigBlueButton):

  -v <version>           Install given version of BigBlueButton (e.g. 'bionic-23') (required)

  -s <hostname>          Configure server with <hostname>
  -e <email>             Email for Let's Encrypt certbot

  -x                     Use Let's Encrypt certbot with manual dns challenges

  -a                     Install BBB API demos
  -g                     Install Greenlight
  -c <hostname>:<secret> Configure with coturn server at <hostname> using <secret>

  -m <link_path>         Create a Symbolic link from /var/bigbluebutton to <link_path>

  -p <host>              Use apt-get proxy at <host>
  -r <host>              Use alternative apt repository (such as packages-eu.bigbluebutton.org)

  -d                     Skip SSL certificates request (use provided certificates from mounted volume)
  -w                     Install UFW firewall

  -h                     Print help

OPTIONS (install coturn only):

  -c <hostname>:<secret> Setup a coturn server with <hostname> and <secret> (required)
  -e <email>             Configure email for Let's Encrypt certbot (required)

OPTIONS (install Let's Encrypt certificate only):

  -s <hostname>          Configure server with <hostname> (required)
  -e <email>             Configure email for Let's Encrypt certbot (required)
  -l                     Install Let's Encrypt certificate (required)
  -x                     Use Let's Encrypt certbot with manual DNS challenges (optional)


EXAMPLES:

Sample options for setup a BigBlueButton server

    -v bionic-23 -s bbb.example.com -e info@example.com -w
    -v bionic-23 -s bbb.example.com -e info@example.com -w -g
    -v bionic-23 -s bbb.example.com -e info@example.com -w -g -c turn.example.com:1234324

Sample options for setup of a coturn server (on a Ubuntu 20.04 server)

    -c turn.example.com:1234324 -e info@example.com

SUPPORT:
    Community: https://bigbluebutton.org/support
         Docs: https://github.com/bigbluebutton/bbb-install
~~~

Before `bbb-install.sh` can install a SSL/TLS certificate, you will need to provide two pieces of information:
   * A fully qualified domain name (FQDN), such as `bbb.example.com`, that resolves to the public IP address of your server
   * An email address

When you have set up the FQDN, check that it correctly resolves to the external IP address of the server using the `dig` command.

~~~
dig bbb.example.com @8.8.8.8
~~~

Note: we're using `bbb.example.com` as an example hostname and `info@example.com` as an example email address.  You need to substitute your real hostname and email.

With just these two pieces of information -- FQDN and email address -- you can use `bbb-install.sh` to automate the configuration of the BigBlueButton server with a TLS/SSL certificate.  For example, to install BigBlueButton 2.3 with a TLS/SSL certificate from Let's Encrypt using `bbb.example.com` and `info@example.com`, enter the command

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v bionic-23 -s bbb.example.com -e info@example.com -w
~~~

The `bbb-install.sh` script will also install a cron job that automatically renews the Let's Encrypt certificate so it doesn't expire.  Cool.


### Installing in a private network

The default installation is meant to be for servers that are publicly available. This is because Let's Encrypt requires to access nginx in order to automatically validate the FQDN provided.

When installing BigBlueButton in a private network, it is possible to validate the FQDN manually, by adding the option `-x` to the command line. As in:

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v bionic-23 -s bbb.example.com -e info@example.com -w -x
~~~

Confirm the use of the email account.

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

Confirm the use of the IP address
```
Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o:
```

A challenge will be generated and shown in the console.

```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.bbb.example.com with the following value:

0bIA-3-RqbRo2EfbYTkuKk7xq2mzszUgVlr6l1OWjW8

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

Before hitting Enter, create a TXT record in the DNS with the challenge that was generated.

```
_acme-challenge.bbb.example.com.  TXT   "0bIA-3-RqbRo2EfbYTkuKk7xq2mzszUgVlr6l1OWjW8"   60
```

The downside of this is that because Let's Encrypt SSL certificates expire after 90 days, it will be necessary to manually update the certificates. In that case an email is sent a few days before the expiration and the next command has to be executed through the console.

```
certbot --email info@example.com --agree-tos -d bbb.example.com --deploy-hook 'systemctl restart nginx' --no-bootstrap --manual-public-ip-logging-ok --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory certonly
```


### Install API demos

You can install the API demos by adding the `-a` option.

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v bionic-23 -s bbb.example.com -e info@example.com -w -a
~~~

Warning: These API demos allow anyone to access your server without authentication to create/manage meetings and recordings. They are for testing purposes only.  Once you are finished testing, you can remove the API demos with `sudo apt-get purge bbb-demo`.


### Install Greenlight

[Greenlight](https://docs.bigbluebutton.org/greenlight/gl-overview.html) is a simple front-end for BigBlueButton written in Ruby on Rails.  It lets users create accounts, have permanent rooms, and manage their recordings.  It also lets you, as the administrator, manage the user accounts (such as approve or deny users).

You can install [Greenlight](http://docs.bigbluebutton.org/install/green-light.html) by adding the `-g` option.

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v bionic-23 -s bbb.example.com -e info@example.com -w -g
~~~

Once Greenlight is installed, it redirects the default home page to Greenlight.  You can also configure GreenLight to use [OAuth2 authentication](http://docs.bigbluebutton.org/greenlight/gl-customize.html).

To launch Greenlight, simply open the URL of your server, such as `https://bbb.example.com/`.  You should see the Greenlight landing page.

![bbb-install.sh](images/greenlight.png?raw=true "Greenlight")

To set up an administrator account for Greenlight (so you can approve/deny sign ups), enter the following commands

~~~
cd greenlight/
docker exec greenlight-v2 bundle exec rake admin:create
~~~

This command will create an admin account and set a default password.  After running this command, login using the given username/password and change the default password. Next, select 'Administrator' and choose 'Organization'.

![bbb-install.sh](images/gl-admin.png?raw=true "Organization")

You can then select 'Site Settings' on the left-hand side and change the Registration Method to 'Approve/Decline'.

![bbb-install.sh](images/gl-approve.png?raw=true "Approve/Decline")

You can now control who creates accounts on your BigBlueButton server.  For more information see [Greenlight administration](http://docs.bigbluebutton.org/greenlight/gl-admin.html).

### Linking `/var/bigbluebutton` to another directory

The install script allows you to pass a path which will be used to create a symbolic link with `/var/bigbluebutton`:

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v bionic-23 -w -m /mnt/test
~~~

This allows users to store the contents of /`var/bigbluebutton`, which can get quite large, in a separate volume.

### Doing everything with a single command

If you want to set up BigBlueButton 2.3 with a TLS/SSL certificate and GreenLight, you can do this all with a single command:

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v bionic-23 -s bbb.example.com -e info@example.com -w -g
~~~

Furthermore, you can re-run the same command later to update your server to the latest version of BigBlueButton 2.3.  We announce BigBlueButton updates to the [bigbluebutton-dev](https://groups.google.com/forum/#!forum/bigbluebutton-dev) mailing list.


### Install a TURN server

Running the BigBlueButton client requires a wide range of UDP ports to be available for WebRTC communication.  However, in some network restricted sites or development environments, such as those behind NAT or a corporate firewall that restricts UDP connections, users may be unable to make outgoing UDP connections to your BigBlueButton server.

If you have setup your BigBlueButton on the internet, and you have users accessing the BigBlueButton server behind a restrictive firewall that blocks UDP connections, then setting up a separate TURN server will allow users to have the TURN server (connected via port 443) proxy their UDP-based WebRTC media (audio, webcam, and screen share) to the BigBlueButton server.

We recommend Ubuntu 20.04 as it has a newer version of [coturn](https://github.com/coturn/coturn) than Ubuntu 16.04.  The server does not need to be very powerful as it will only relay communications from the BigBlueButton client to the BigBlueButton server when necessary.  A dual core server on Digital Ocean should be sufficient for a dozen BigBlueButton servers.  

The server should have the following additional ports available:

| Ports         | Protocol      | Description |
| ------------- | ------------- | ----------- |
| 3478          | TCP/UDP       | coturn listening port |
| 80            | TCP           | HTTP required for Certbot |
| 443           | TCP/UDP       | TLS listening port |
| 32769-65535   | UDP           | relay ports range |


Before running `bbb-install.sh` to setup the TURN server (which installs and configures the `coturn` package), you need

  * A fully qualified domain name (FQDN) with 
    * an A record that resolves to the server's public IPV4 address
    * an AAAA record that resolves to the server's public IPV6 address
  * An email address for Let's Encrypt
  * A secret key (it can be an 8 to 16 character random string that you create).

With the above in place, you can set up a TURN server for BigBlueButton using the command

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -c <FQDN>:<SECRET> -e <EMAIL>
~~~

Note, we've omitted the `-v` option, which causes `bbb-install.sh` to just install and configure coturn.  For example, using `turn.example.com` as the FQDN, `1234abcd` as the shared secret, and `info@example.com` as the email address (you would need to substitute your own values), logging into the server via SSH and running the following command as root

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -c turn.example.com:1234abcd -e info@example.com
~~~

will do the following

  * Install the latest version of coturn available for Ubuntu 20.04
    * Provide a minimal configuration for `/etc/turnserver.conf`
    * Add a systemd override to ensure coturn can bind to port 443
    * Configure logging to `/var/log/turnserver/turnserver.log`
    * Add a logrotate configuration to keep the logs for 7 days
  * Setup a SSL certificate using Let's Encrypt
    * Add a deploy hook for Let's Encrypt to have coturn reload the certificates upon renewal 

With a SSL certificate in place, coturn can relay access to your BigBlueButton server via TCP/IP on port 443.  This means if a user is behind a restrictive firewall that blocks all outgoing UDP connections, the TURN server can accept connections from the user via TCP/IP on port 443 and relay the data to your BigBlueButton server via UDP.

After the TURN server is setup, you can configure your BigBlueButton server to use the TURN server by running the `bbb-install.sh` command again and add the parameter `-c <FQDN>:<SECRET>` (this tells `bbb-install.sh` to setup the configuration for the TURN server running at <FQDN> using the share secret <SECRET>.  For example,

~~~
wget -qO- https://ubuntu.bigbluebutton.org/bbb-install.sh | bash -s -- -v bionic-23 -s bbb.example.com -e info@example.com -c turn.example.com:1234abcd
~~~

You can re-use a single TURN server for multiple BigBlueButton installations.

## Next steps

If you intend to use this server for production you should uninstall the API demos using the command

~~~
apt-get purge bbb-demo
~~~

You can also do a number of [customizations](http://docs.bigbluebutton.org/2.2/customize.html) to your server as well.

## Troubleshooting

### Packaging server is blocked

We are currently hosting the packaging on a Digital Ocean servlet, but recently the IP range for some Digital Ocean servers has been [blocked in some countries](https://www.digitalocean.com/community/questions/unable-to-reach-digitalocean-server-from-russia).

If you're having troubles installing, try running the `bbb-install.sh` command but change the value

~~~
https://ubuntu.bigbluebutton.org/bbb-install.sh
~~~

to

~~~
https://packages-eu.bigbluebutton.org/bbb-install.sh
~~~


### Greenlight not running

If on first install Greenlight gives you a `500 error` when accessing it, you can [restart Greenlight](http://docs.bigbluebutton.org/install/greenlight-v2.html#if-you-ran-greenlight-using-docker-run).


### Tomcat7 not running

If on the initial install you see

~~~
# Not running:  tomcat7 or grails LibreOffice
~~~

just run `sudo bbb-conf --check` again.  Tomcat7 may take a bit longer to start up and isn't running the first time you run `sudo bbb-conf --check`.


### Getting help

If you have feedback on the script, or need help using it, please post to the [BigBlueButton Setup](https://bigbluebutton.org/support/community/) mailing list with details of the issue (and include related information such as steps to reproduce the error).

If you encounter an error with the script (such as it not completing or throwing an error), please open a [GitHub issue](https://github.com/bigbluebutton/bbb-install/issues) and provide steps to reproduce the issue.


## Limitations

If you are running your BigBlueButton behind a firewall, such as on EC2, this script will not configure your firewall.  You'll need to [configure the firewall](#configuring-the-external-firewall) manually.

If you are upgrading from a very old version of 2.2.x (such as 2.2.3) to the most recent version of 2.2 (using `-v xenial-22`) then `sudo bbb-conf --check` will still show the older version when `bbb-install.sh finishes.  To resolve, run `dpkg --configure -a` and then run `bbb-install.sh` again.

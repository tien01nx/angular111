1. Install Ubuntu into VMware Workstation.

Commands:

    apt update -y && apt upgrade -y
        about 479MB space needed
    apt full-upgrade -y
        Most likely, you will get 0 need to be ugpraded if you already ran apt upgrade.
    reboot

apt upgrade will upgrade all packages that can be upgraded without the need to install additional packages or remove any conflicting installed packages. Basically it will apply all package upgrades that do not include changed dependencies.

apt full-upgrade (the correct equivalent for apt-get dist-upgrade) applies package upgrades as well if they require either the install of new packages or the removal of conflicting installed packages. Basically it will apply all package upgrades including those with changed dependencies.

On stable distro suites, when not adding new APT sources, one will face changed dependencies rarely, hence usually apt full-upgrade is not required or does not apply any additional upgrades compared to apt upgrade, except for Linux image package upgrades. Linux image packages are usually shipped as meta packages, e.g. linux-image-amd64, which then depends on the actual versioned package, e.g. linux-image-4.19.0-9-amd64. When one uses apt upgrade, the Linux image meta package will usually not be upgraded, since it would required to install a new versioned Linux image package. Running apt full-upgrade on the other hand will do the upgrade, furthermore it might remove the old versioned package, if the new one has a related conflict defined (which is usually not the case for Linux image packages, just as example!). 2. Install Required Dependencies

sudo apt install build-essential libcairo2-dev libjpeg-turbo8-dev \
 libpng-dev libtool-bin libossp-uuid-dev libvncserver-dev \
 freerdp2-dev libssh2-1-dev libtelnet-dev libwebsockets-dev \
 libpulse-dev libvorbis-dev libwebp-dev libssl-dev \
 libpango1.0-dev libswscale-dev libavcodec-dev libavutil-dev \
 libavformat-dev

Get Guacamole server 1.6.0

Notes: other version might be not compatible with Ubuntu 24.04. Test the steps before use this version.

wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz

1.6.0 is about 1.2MB.

sudo wget https://downloads.apache.org/guacamole/1.6.0/source/guacamole-server-1.6.0.tar.gz
sudo tar -xvf guacamole-server-1.6.0.tar.gz
sudo cd guacamole-server-1.6.0

Old version using 1.5.4 might not work with Ubuntu24.0.4. File size is about 1.1MB.

2 Config and Install

    sudo ./configure --with-systemd-dir=/usr/local/lib/systemd/system
    sudo make
    sudo make install

3 Wait build and installation process done, then update the installed library cache with the command below:

    sudo ldconfig

4 At this point, everything is installed, but guacd is not running. You will need to run guacd in order to use Guacamole once the client components are installed as well.

Beware that even after installing guacd and its startup script, you will likely still have to activate the service for it to start automatically. Doing this varies by distribution, but each distribution will have documentation describing how to do so.

5 Running Guacamole-Server
root@ubuntu1:~# systemctl enable --now guacd
Synchronizing state of guacd.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable guacd
Created symlink /etc/systemd/system/multi-user.target.wants/guacd.service → /usr/local/lib/systemd/system/guacd.service.
root@ubuntu1:~# systemctl status guacd
● guacd.service - Guacamole Server
Loaded: loaded (/usr/local/lib/systemd/system/guacd.service; enabled; preset: enabled)
Active: active (running) since Sun 2025-08-17 23:48:24 UTC; 6s ago
Docs: man:guacd(8)
Main PID: 37723 (guacd)
Tasks: 1 (limit: 4548)
Memory: 10.0M (peak: 10.1M)
CPU: 60ms
CGroup: /system.slice/guacd.service
└─37723 /usr/local/sbin/guacd -f
Aug 17 23:48:24 ubuntu1 systemd[1]: Started guacd.service - Guacamole Server.
Aug 17 23:48:25 ubuntu1 guacd[37723]: Guacamole proxy daemon (guacd) version 1.6.0 started
Aug 17 23:48:25 ubuntu1 guacd[37723]: guacd[37723]: INFO: Guacamole proxy daemon (guacd) version 1.6.0 started
Aug 17 23:48:25 ubuntu1 guacd[37723]: guacd[37723]: INFO: Listening on host 127.0.0.1, port 4822
Aug 17 23:48:25 ubuntu1 guacd[37723]: Listening on host 127.0.0.1, port 4822
root@ubuntu1:~#

Install Tomcat Servlet
apt install tomcat9 tomcat9-admin tomcat9-common tomcat9-user -y

stall Ubuntu 22.04 Jammy updates universe repos on Ubuntu 24.04;

echo 'deb http://ke.archive.ubuntu.com/ubuntu/ jammy-updates universe' > /etc/apt/sources.list.d/tomcat9.list

Run system update;

apt update

Now you should be able to run following installation command to get tomcat9 installed.

apt install tomcat9 tomcat9-admin tomcat9-common tomcat9-user -y

Disable Ubuntu 22.04 Jammy updates universe repos and run system package cache update;

sed -i 's/^/#/' /etc/apt/sources.list.d/tomcat9.list

apt update

Tomcat9 is started and enabled to run on system boot upon installation.

systemctl status tomcat9

If UFW is running, allow Tomcat through it.

ufw allow 8080/tcp

Manually Install Tomcat with latest release:

Install Guacamole-client
In guacamole-client contains all Java and JavaScript components of Guacamole (guacamole, guacamole-common, guacamole-ext, and guacamole-common-js). These components ultimately make up the web application that will serve the HTML5 Guacamole client to users that connect to your server. This web application will then connect to guacd, part of guacamole-server, on behalf of connected users in order to serve them any remote desktop they are authorized to access.

guacamole-client contains provides web application that will serve the HTML5 Guacamole client to users that connect to your server. The web application will then connect to guacd on behalf of connected users in order to serve them any remote desktop they are authorized to access.

Create Guacamole configuration directory;

mkdir /etc/guacamole

Download Guacamole-client Binary

Guacamole client can be installed from a source code or from ready binary. Binary installation is used in this demo. Download Guacamole-client from Guacamole releases page for the respective latest version (v1.5.5 as of this writing) and store it in the configuration directory created above;

VER=1.6.0

wget https://downloads.apache.org/guacamole/${VER}/binary/guacamole-${VER}.war -O /etc/guacamole/guacamole.war

It is about 15MB downloading.

Create a symbolic link of the guacamole client to Tomcat webapps directory as shown below;

ln -s /etc/guacamole/guacamole.war /var/lib/tomcat9/webapps/

Restart Tomcat to deploy the new web application;

systemctl restart tomcat9

Restart guacd daemon as well;

systemctl restart guacd

systemctl status guacd

Configure Apache
Guacamole 1.6.0 on Ubuntu 24.04

Guacamole has two major configuration files; /etc/guacamole which is referenced by the GUACAMOLE_HOME environment variable and /etc/guacamole/guacamole.properties which is the main configuration file used by Guacamole and its extensions.

There are also guacamole extensions and libraries configurations. You need to create the directories for these configs;

mkdir /etc/guacamole/{extensions,lib}

Set the guacamole home directory environment variable and add it to /etc/default/tomcat9 configuration file.

echo "GUACAMOLE_HOME=/etc/guacamole" >> /etc/default/tomcat9

Configure Guacamole Server Connections

To define how Guacamole connects to guacd, create the guacamole.properties file under /etc/guacamole directory with the following content.

vim /etc/guacamole/guacamole.properties

guacd-hostname: localhost
guacd-port: 4822
user-mapping: /etc/guacamole/user-mapping.xml
auth-provider: net.sourceforge.guacamole.net.basic.BasicFileAuthenticationProvider

After that, save the configuration file and link the Guacamole configurations directory to Tomcat servlet directory as shown below.

ln -s /etc/guacamole /usr/share/tomcat9/.guacamole

Configure Guacamole Authentication Method

Guacamole’s default authentication method reads all users and connections from a single file called user-mapping.xml. In this file,you need to define the users allowed to access Guacamole web UI, the servers to connect to and the method of connection.

Therefore, run the command below to create this file with the following contents.

vim /etc/guacamole/user-mapping.xml

Be sure to replace password with your strong password.

<user-mapping>
        
    <!-- Per-user authentication and config information -->

    <!-- A user using md5 to hash the password
         guacadmin user and its md5 hashed password below is used to
             login to Guacamole Web UI-->
    <authorize
            username="guacadmin"
            password="5f4dcc3b5aa765d61d8327deb882cf99"
            encoding="md5">

        <!-- First authorized Remote connection -->
        <connection name="CentOS-Server">
            <protocol>ssh</protocol>
            <param name="hostname">192.168.56.156</param>
            <param name="port">22</param>
        </connection>

        <!-- Second authorized remote connection -->
        <connection name="Windows 7">
            <protocol>rdp</protocol>
            <param name="hostname">192.168.56.122</param>
            <param name="port">3389</param>
            <param name="username">netsec</param>
            <param name="ignore-cert">true</param>
        </connection>

    </authorize>

</user-mapping>

Generate the MD5 hash of passwords for the user used for logging into Guacamole web user interface. Replace you password accordingly;

echo -n password | openssl md5

printf '%s' password | md5sum

You should get following md5 value for your password string:

5f4dcc3b5aa765d61d8327deb882cf99

If you need to explicitly define usernames and passwords, add the parameters;

<param name="username">USERNAME</param>
<param name="password">PASSWORD</param>

Discover more
Software
deploy
remote desktop client
linux

Save and exit the configuration file.

You can check how to enable Guacamole OpenLDAP Authentication;

Setup Apache Guacamole OpenLDAP Authentication

Restart both Tomcat and guacd to effect the changes.

systemctl restart tomcat9 guacd

Be sure to check the syslog, /var/log/syslog or /var/log/tomcat9/CATALINA-\* for any issues.

Test Apache Guacamole 1.6.0 from Browser

Once Guacamole is setup, you can access it from web browser using the address: http://192.168.30.132:8080/guacamole/

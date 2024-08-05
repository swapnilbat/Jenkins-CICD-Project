# Nexus installtation

Launch Amazon Linux

Search for Java-1.8

Yum search yum search java-1.8

Install java-1.8.0-amazon-corretto.x86_64  

yum install java-1.8.0-amazon-corretto.x86_64 -y

Create Nexus directory 

mkdir -p /opt/nexus/

cd /opt/nexus/

Download Nexes Sonartype Binary 

wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz

Untar the file

tar -xvf latest-unix.tar.gz

rm -rf rm -rf latest-unix.tar.gz

Create nexus user and give permission

useradd nexus

chown -R nexus:nexus /opt/nexus/

Create systemd service for Nexus

vim /etc/systemd/system/nexus.service

[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/nexus-3.70.1-02/bin/nexus start
ExecStop=/opt/nexus/nexus-3.70.1-02/bin/nexus stop
User=nexus
Restart=on-abort

[Install]
WantedBy=multi-user.target


Add nexus user in nexus.rc file

vim /opt/nexus/nexus-3.70.1-02/bin/nexus.rc

run_as_user="nexus"
  
Start Nexus service

systemctl daemon-reload

systemctl start nexus

systemctl enable nexus

Browse URL - <Server IP>:8081

Login - admin
for password browse this path - cat /opt/nexus/sonatype-work/nexus3/admin.password

# Create Ubuntu Linux instance in Linux
# Take sysctl.conf file bakup
cp /etc/sysctl.conf /root/sysctl.conf_backup
# Open systctl.conf file and add below entries
vim /etc/sysctl.conf
vm.max_map_count=262144
 fs.file-max=65536
 ulimit -n 65536
 ulimit -u 4096
# Take limits.conf file backup
cp /etc/security/limits.conf /root/sec_limit.conf_backup
# Open limits.conf file and add below entries
vim /etc/security/limits.conf
 sonarqube   -   nofile   65536
 sonarqube   -   nproc    409
sudo apt-get update -y
sudo apt-get install openjdk-17-jdk -y
sudo update-alternatives --config java
java -version

sudo apt update

# Install Postgresql
apt install postgresql
systemctl status postgresql

# Create and update below User and setting in Postgresql
sudo echo "postgres:admin123" | chpasswd
runuser -l postgres -c "createuser sonar"
sudo -i -u postgres psql -c "ALTER USER sonar WITH ENCRYPTED PASSWORD 'admin123';"
ALTER ROLE
sudo -i -u postgres psql -c "CREATE DATABASE sonarqube OWNER sonar;"
CREATE DATABASE
sudo -i -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonar;"

# Restart Postgressql service
systemctl restart  postgresql
systemctl status  postgresql

# Create sonarqube directory
sudo mkdir -p /sonarqube/
cd /sonarqube/

# Download sonarqube binaries
curl -O https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.6.0.92116.zip

Unzip the zip file in /opt direcotory
apt-get install zip -y
unzip -o sonarqube-10.6.0.92116.zip -d /opt/
mv /opt/sonarqube-10.6.0.92116/ /opt/sonarqube

# Add group Sonar
groupadd sonar

# Add user sonar
useradd -c "SonarQube - User" -d /opt/sonarqube/ -g sonar sonar
chown sonar:sonar /opt/sonarqube/ -R

# Add below configuratoin in sonar.properties_backup

cp /opt/sonarqube/conf/sonar.properties /root/sonar.properties_backup
cat <<EOT> /opt/sonarqube/conf/sonar.properties
sonar.jdbc.username=sonar
sonar.jdbc.password=admin123
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
sonar.web.host=0.0.0.0
sonar.web.port=9000
sonar.web.javaAdditionalOpts=-server
sonar.search.javaOpts=-Xmx512m -Xms512m -XX:+HeapDumpOnOutOfMemoryError
sonar.log.level=INFO
sonar.path.logs=logs
EOT

# Create systemd service for Sonarqube
cat <<EOT> /etc/systemd/system/sonarqube.service
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096


[Install]
WantedBy=multi-user.target
EOT

# Start Sonarqube service
systemctl daemon-reload
systemctl enable sonarqube.service
#systemctl start sonarqube.service
#systemctl status sonarqube.service

# Install Nginx 
apt-get install nginx -y
rm -rf /etc/nginx/sites-enabled/default
rm -rf /etc/nginx/sites-available/default

# Add below Nginx configuraton 

cat <<EOT> /etc/nginx/sites-available/sonarqube
server{
    listen      80;
    server_name sonarqube.groophy.in;

    aerver {
    listen 80;
    server_name sonarqube.example.com;

    access_log /var/log/nginx/sonarqube.access.log;
    error_log /var/log/nginx/sonarqube.error.log;

    location / {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOT

# Create soft link
ln -s /etc/nginx/sites-available/sonarqube.groophy.in /etc/nginx/sites-enabled/

# Restart the Nginx process
nginx -t
systemctl restart nginx

# Browse the server IP as URL. Will get Sonarqube login page
username - admin
password - admin





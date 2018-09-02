Setup Jenkins with SSL ( lets Encrypt )

1) setup LetEncrypt ( Amazon Linux 2 )

(a) Git clone and setup letsencrypt
sudo yum install -y git
sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt

2) Append & Replace Line No:780

File : /opt/letsencrypt/certbot-auto
File : /opt/letsencrypt/letsencrypt-auto

Line:

elif [ -f /etc/redhat-release ] || grep 'cpe:.*:amazon_linux:2' /etc/os-release > /dev/null 2>&1; then


3)Generate Certificate for your jenkins Domain ( Sample Domain : jenkins.azfar.biz )

sudo -i
cd /opt/letsencrypt
./letsencrypt-auto certonly --standalone -d jenkins.azfar.biz


4) Create P12 File

cd /etc/letsencrypt/archive/jenkins.azfar.biz/

( Certificate Key - new.cert.key , Certificate - new.cert.cert )

openssl pkcs12 -export -out jenkins_keystore.p12 -passout 'pass:password'  -inkey new.cert.key -in new.cert.cert -name jenkinsci.com

5) Create JKS File ( Keystore )

keytool -importkeystore -srckeystore jenkins_keystore.p12   -srcstorepass 'password' -srcstoretype PKCS12   -srcalias jenkinsci.com -deststoretype JKS      -destkeystore jenkins_keystore.jks -deststorepass 'password'      -destalias jenkinsci.com

6) Copy Keystore in Jenkins Home

mkdir -p  /var/lib/jenkins/keystore

chown -R jenkins:jenkins /var/lib/jenkins/keystore

cp jenkins_keystore.jks /var/lib/jenkins/keystore

chown jenkins:jenkins /var/lib/jenkins/keystore/jenkins_keystore.jks

7) Ensure Jenkins Sysconfig like below., File : /etc/sysconfig/jenkins


JENKINS_JAVA_CMD=""
JENKINS_USER="jenkins"
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true"
JENKINS_PORT="8080"
JENKINS_LISTEN_ADDRESS=""
JENKINS_HTTPS_PORT="8443"
JENKINS_HTTPS_KEYSTORE="/var/lib/jenkins/keystore/jenkins_keystore.jks"
JENKINS_HTTPS_KEYSTORE_PASSWORD="password"
JENKINS_HTTPS_LISTEN_ADDRESS=""
JENKINS_DEBUG_LEVEL="5"
JENKINS_ENABLE_ACCESS_LOG="no"
JENKINS_HANDLER_MAX="100"
JENKINS_HANDLER_IDLE="20"
JENKINS_ARGS=""

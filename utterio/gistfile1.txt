#!/bin/bash
 
# Make sure only root can run our script
if [ "$(id -u)" != "0" ]; then
   echo "You need to be 'root' dude." 1>&2
   exit 1
fi
 
clear 
 
echo;
echo "##########################################################################################

This script is installing and configuring OpenBazaar for demo purposes.

##########################################################################################
";
echo;
 
# base directory
export BASE_DIR="/opt/OpenBazaar"
 
# install git
sudo apt-get -y install git
sudo apt-get -y install rng-tools
sudo apt-get -y install monit
 
# get the source
cd /opt/
git clone https://github.com/OpenBazaar/OpenBazaar.git ${BASE_DIR}
 
# deal with the random generator issues
sudo echo "HRNGDEVICE=/dev/urandom" >> /etc/default/rng-tools
sudo service rng-tools start
 
# hack the configurator for automation
cd /opt/OpenBazaar/
sudo sed -e "s/apt-get/apt-get -y /" configure.sh > configure_auto.sh
chmod 755 configure_auto.sh
 
# run the configurator
sudo ${BASE_DIR}/configure_auto.sh
 
# randomize port number
PORT=`shuf -i 10000-65000 -n 1`
 
# scripts to start and stop
cat <<EOF > ${BASE_DIR}/start.sh
#!/bin/bash
cd ${BASE_DIR}
./openbazaar -k 0.0.0.0 -q $PORT start
EOF
cat <<EOF > ${BASE_DIR}/stop.sh
#!/bin/bash
cd ${BASE_DIR}
./openbazaar stop
EOF
cat <<EOF > ${BASE_DIR}/cron_start.sh
#!/bin/bash
cd ${BASE_DIR}
./openbazaar -k 0.0.0.0 -q $PORT start
crontab -r
EOF
chmod 755 ${BASE_DIR}/stop.sh
chmod 755 ${BASE_DIR}/start.sh
chmod 755 ${BASE_DIR}/cron_start.sh
 
# monit config
cat <<EOF > /etc/monit/conf.d/openbazaar
set httpd port 5150 and
	use address localhost
	allow localhost
set daemon 30
with start delay 5
check process openbazaar matching "openbazaar"
	start program = "/bin/bash ${BASE_DIR}/start.sh"
	stop program = "/bin/bash ${BASE_DIR}/stop.sh"
EOF
 
# restart monit service
service monit restart
sleep 2
monit monitor all
 
# update the meta data on the pool directly
. /etc/utterio
curl -X PUT -d '{"openbazaar": "installed", "openbazaar_port": "'$PORT'"}' $MY_POOL_API_ADDRESS
 
# initial start using crontab (which wipes itself when it runs)
cat <<EOF > ${BASE_DIR}/crontab
* * * * * ${BASE_DIR}/cron_start.sh
EOF
crontab ${BASE_DIR}/crontab
 
echo;
echo "##########################################################################################

OpenBazaar setup complete.

##########################################################################################
";
echo;
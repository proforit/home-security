$scriptrh7 = <<SCRIPT

yum -y update
yum -y install curl wget git epel-release bind-utils net-tools

yum -y install httpd policycoreutils-python python34-pip
pip3 install --upgrade pip

# Install jinja2 cli - https://github.com/mattrobenolt/jinja2-cli
pip install pyyaml jinja2-cli jinja2-cli[yaml]

# webserver setup
jinja2 /srv/vagrant/webserver/install.sh.j2 /srv/vagrant/data.yaml | bash

SCRIPT

$scripttrusty = <<SCRIPT
# https://www.home-assistant.io/docs/installation/raspberry-pi/
add-apt-repository -y ppa:deadsnakes/ppa
apt-get update
apt-get -y upgrade
apt-get -y autoremove
apt-get -y install python3.5 python3.5-venv python3-pip

useradd -rm homeassistant -G dialout

mkdir -p /srv/homeassistant
chown homeassistant:homeassistant /srv/homeassistant

su -l homeassistant -c '
  if [ ! -e hass-venv ]; then
    python3.5 -m venv hass-venv
    cd hass-venv
    source bin/activate
    python3 -m pip install wheel
    pip3 install homeassistant
  else
    cd hass-venv
  fi
  source bin/activate
  echo Starting hass with nohup. Login shortly at http://localhost:8123
  nohup hass > ~/hass-console.log 2>&1 </dev/null &
'

SCRIPT

# Require YAML module
require 'yaml'

# Load data.yaml
data = YAML.load_file(File.join(File.dirname(__FILE__), 'data.yaml'))

Vagrant.configure("2") do |config|

  config.vm.define "home-security-homeassistant" do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.hostname = $config
    config.vm.network "private_network", ip: "192.168.60.10"
    config.vm.network "forwarded_port", guest: 8123, host: 8123, host_ip: '127.0.0.1'
    config.vm.provision "shell", inline: $scripttrusty
  end

  config.vm.define "home-security-webserver" do |config|
    config.vm.box = "centos/7"
    config.vm.hostname = $config
    config.vm.synced_folder ".", "/srv/vagrant/"
    config.vm.network "private_network", ip: "192.168.60.20"
    config.vm.network "forwarded_port", guest: "#{data['webserverport']}", host: "#{data['webserverport']}", host_ip: '127.0.0.1'
    config.vm.provision "shell", inline: $scriptrh7
  end
end

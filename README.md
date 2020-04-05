# zigbee2mqtt_in_jail
Guide how to install zigbee2mqtt on FreeNAS jail

I have installed zigbee2mqtt on Mosquitto jail which was created via FreeNAs Community plugin and is up and running. Also connected and flashed CC2531 with latest firmware for zigbee2mqtt

Guide to run mosquitto:
1. Creat user and password in Mosquitto:
  In file /user/local/etc/mosquitto/pwfile (creat it if not available) write one line for example:
  mqtt:secretPassword
  Than execute this command:
  mosquitto_passwd -c /usr/local/etc/mosquitto/pwfile [user_name]
2. Now edit mosquitto.conf file:
    persistence false
		listener 1883
		allow_anonymous false
		protocol mqtt
		password_file /usr/local/etc/mosquitto/pwfile
3. Restart mosquitto service:
  service mosquitto restart
  
Guide to install zigbee2mqtt:
1. Install all step by step
  pkg install screen
	pkg install node
	pkg install npm
	pkg install git
	pkg install gcc
	git clone https://github.com/Koenkk/zigbee2mqtt.git /opt/zigbee2mqtt
	cd /opt/zigbee2mqtt
	npm install
2. edit configuration file /opt/zigbee2mqtt/data/configuration.yaml
homeassistant: true
permit_join: true
mqtt:
  base_topic: zigbee2mqtt
  server: 'mqtt://IP_server_mqtt:1883'
  user: user_mqtt
  password: password_mqtt
serial:
  port: /dev/cuaU1 -> usb under which CC2531 is connected
advanced:
  homeassistant_discovery_topic: homeassistant
  homeassistant_status_topic: hass/status

Auto run zigbee2mqtt:
1. Create a file in /etc/rc.d file name: zigbee2mqtt
2. Paste this:
#!/bin/sh

. /etc/rc.subr

name=zigbee2mqtt
rcvar=zigbee2mqtt_enable

start_cmd="${name}_start"
stop_cmd=":"
load_rc_config $name
pidfile="/var/run/${name}.pid"


zigbee2mqtt_start(){
export PATH="$PATH:"/usr/local/bin/
WorkingDirectory=/opt/zigbee2mqtt
screen -d -m /usr/local/bin/npm start --prefix /opt/zigbee2mqtt/
}

stop_postcmd=stop_postcmd
stop_postcmd()
{
  rm -f $pidfile
}

run_rc_command "$1"

-----------
3. Save file and execute command: chmod +x /etc/rc.d/zigbee2mqtt
4. Run command: service zigbee2mqtt start

To watch what is going on zigbee2mqtt:
screen -x
To exit from watching without closing session:
CTRL+a and click d

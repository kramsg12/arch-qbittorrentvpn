**Application**

[qBittorrent](https://www.qbittorrent.org/)  
[OpenVPN](https://openvpn.net/)  
[Privoxy](http://www.privoxy.org/)

**Description**

qBittorrent is a bittorrent client programmed in C++ / Qt that uses libtorrent (sometimes called libtorrent-rasterbar) by Arvid Norberg. It aims to be a good alternative to all other bittorrent clients out there. qBittorrent is fast, stable and provides unicode support as well as many features. This Docker image is using the headless configuration with the web frontend enabled, as well as OpenVPN to ensure a secure and private connection to the Internet, including use of iptables to prevent IP leakage when the tunnel is down. Privoxy is also included to allow unfiltered access to index sites, to use Privoxy please point your application at http://<host ip>:8118.

**Build notes**

Latest stable qBittorrent release from Arch Linux repo.
Latest stable OpenVPN release from Arch Linux repo.
Latest stable Privoxy release from Arch Linux repo.

**Usage**
```
docker run -d \
    --privileged=true \
    -p 6881:6881 \
    -p 6881:6881/udp \
    -p 8080:8080 \
    -p 8118:8118 \
    --name=<container name> \
    -v <path for data files>:/data \
    -v <path for config files>:/config \
    -v /etc/localtime:/etc/localtime:ro \
    -e VPN_ENABLED=<yes|no> \
    -e VPN_USER=<vpn username> \
    -e VPN_PASS=<vpn password> \
    -e VPN_PROV=<pia|airvpn|custom> \
    -e VPN_OPTIONS=<additional openvpn cli options> \
    -e STRICT_PORT_FORWARD=<yes|no> \
    -e ENABLE_PRIVOXY=<yes|no> \
    -e LAN_NETWORK=<lan ipv4 network>/<cidr notation> \
    -e NAME_SERVERS=<name server ip(s)> \
    -e DEBUG=<true|false> \
    -e WEBUI_PORT=<port for web interfance> \
    -e UMASK=<umask for created files> \
    -e PUID=<uid for user> \
    -e PGID=<gid for user> \
    binhex/arch-qbittorrentvpn
```
&nbsp;
Please replace all user variables in the above command defined by <> with the correct values.

**Access qBittorrent (web ui)**

`http://<host ip>:8080/`

Username:- admin
Password:- adminadmin

**Access Privoxy**

`http://<host ip>:8118`

**PIA example**
```
docker run -d \
    --privileged=true \
    -p 6881:6881 \
    -p 6881:6881/udp \
    -p 8080:8080 \
    -p 8118:8118 \
    --name=qbittorrentvpn \
    -v /root/docker/data:/data \
    -v /root/docker/config:/config \
    -v /etc/localtime:/etc/localtime:ro \
    -e VPN_ENABLED=yes \
    -e VPN_USER=myusername \
    -e VPN_PASS=mypassword \
    -e VPN_PROV=pia \
    -e STRICT_PORT_FORWARD=yes \
    -e ENABLE_PRIVOXY=yes \
    -e LAN_NETWORK=192.168.1.0/24 \
    -e NAME_SERVERS=209.222.18.222,37.235.1.174,1.1.1.1,8.8.8.8,209.222.18.218,37.235.1.177,1.0.0.1,8.8.4.4 \
    -e DEBUG=false \
    -e WEBUI_PORT=8080 \
    -e UMASK=000 \
    -e PUID=0 \
    -e PGID=0 \
    binhex/arch-qbittorrentvpn
```
&nbsp;
**AirVPN provider**

AirVPN users will need to generate a unique OpenVPN configuration file by using the following link https://airvpn.org/generator/

1. Please select Linux and then choose the country you want to connect to
2. Save the ovpn file to somewhere safe
3. Start the qbittorrentvpn docker to create the folder structure
4. Stop qbittorrentvpn docker and copy the saved ovpn file to the /config/openvpn/ folder on the host
5. Start qbittorrentvpn docker
6. Check supervisor.log to make sure you are connected to the tunnel

AirVPN users will also need to create a port forward by using the following link https://airvpn.org/ports/ and clicking Add. This port will need to be specified in the qBittorrent configuration file located at /config/qbittorrent/config/qbittorrent.conf.

qBittorrent example config
```
port_range = 49400-49400
port_random = no
```
&nbsp;
**AirVPN example**
```
docker run -d \
    --privileged=true \
    -p 6881:6881 \
    -p 6881:6881/udp \
    -p 8080:8080 \
    -p 8118:8118 \
    --name=qbittorrentvpn \
    -v /root/docker/data:/data \
    -v /root/docker/config:/config \
    -v /etc/localtime:/etc/localtime:ro \
    -e VPN_ENABLED=yes \
    -e VPN_PROV=airvpn \
    -e ENABLE_PRIVOXY=yes \
    -e LAN_NETWORK=192.168.1.0/24 \
    -e NAME_SERVERS=209.222.18.222,37.235.1.174,8.8.8.8,209.222.18.218,37.235.1.177,8.8.4.4 \
    -e DEBUG=false \
    -e WEBUI_PORT=8080 \
    -e UMASK=000 \
    -e PUID=0 \
    -e PGID=0 \
    binhex/arch-qbittorrentvpn
```
&nbsp;
**Notes**

Please note this Docker image does not include the required OpenVPN configuration file and certificates. These will typically be downloaded from your VPN providers website (look for OpenVPN configuration files), and generally are zipped.

PIA users - The URL to download the OpenVPN configuration files and certs is:-

https://www.privateinternetaccess.com/openvpn/openvpn.zip

Once you have downloaded the zip (normally a zip as they contain multiple ovpn files) then extract it to /config/openvpn/ folder (if that folder doesn't exist then start and stop the docker container to force the creation of the folder).

If there are multiple ovpn files then please delete the ones you don't want to use (normally filename follows location of the endpoint) leaving just a single ovpn file and the certificates referenced in the ovpn file (certificates will normally have a crt and/or pem extension).

User ID (PUID) and Group ID (PGID) can be found by issuing the following command for the user you want to run the container as:-

`id <username>`

Due to issues with CSRF and port mapping, should you require to alter the port for the webui you need to change both sides of the -p 8080 switch AND set the WEBUI_PORT variable to the new port.

For example, to set the port to 8090 you need to set -p 8090:8090 and -e WEBUI_PORT=8090
___
If you appreciate my work, then please consider buying me a beer  :D

[![PayPal donation](https://www.paypal.com/en_US/i/btn/btn_donate_SM.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=MM5E27UX6AUU4)

[Support forum](https://forums.unraid.net/topic/75539-support-binhex-qbittorrentvpn/)

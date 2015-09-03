# routerlab

Fortsättning på dhclplabben men nu bygger vi en router som skickar trafiken vidare.

##  Maskinerna

Det finns två virtuella maskiner, happyclient och hapyrouter. Dom knyts samman av ett privat nätverk. Dhcpservern router har adressen 192.168.0.2. Happyclient ansluter till happyrouter och ska kunna komma åt internet via denna. 


Happyclient ska vi också testa att byta ut mot en riktig dator. 

Börja med att ladda ner filerna och `vagrant up` och `vagrant ssh happyrouter`


## Happyrouter och dhcpservern

Vi börjar med att installera och konfigurera dhcpservern i happyrouter

### Installera och konfigurera [dhcpservern](https://www.isc.org/downloads/dhcp/) 

	vagrant@happyrouter:~$  sudo apt-get install isc-dhcp-server 

Öpppna konfigurationsfilen i en editor

	vagrant@happyrouter:~$ sudo nano /etc/dhcp/dhcpd.conf

Efter att jag hade testat såg min konfigurationsfil ut så här

    ddns-update-style none;
    option domain-name-servers 10.0.2.3;
    default-lease-time 600;
    max-lease-time 7200;
    log-facility local7;
    subnet 192.168.0.0 netmask 255.255.255.0 {
     range 192.168.0.10 192.168.0.100;  
     option routers 192.168.0.2; # this is our happyrouter
    }

Vi måste också tala om vilken nätverskort dhcpservern ska lyssna på


	sudo nano /etc/default/isc-dhcp-server

Lägg till eth1 i INTERFACES
	
	INTERFACES="eth1"
	
Nu är det bara att starta om servern.

	 vagrant@happyrouter:~$ sudo service isc-dhcp-server restart

Ni kan också byta ut restart mot stop, start eller status.

Om den inte vill starta, kan ni titta i syslogen för fel, tail ger er sista raderna

   vagrant@happyrouter:~$ sudo tail /var/log/syslog

## Nu ska vi testa dhcspservern från klienter och kola stt det fungerar.

Vi börjar med den virtuella happyclient


### Happyclient

Nu kan vi testa att dhcpservern fungerar, ny terminal och rätt katalog


	$ vagrant up happyclient
	

Här behöver vi bara ändra inställningarna för nätverket så att det använder dhcp.

	vagrant@happyclient:~$ sudo nano /etc/network/interfaces
	
lägg till eller uppdatera eth1
	
	auto eth1 
	iface eth1 inet dhcp

spara och starta om nätverkskortet

	vagrant@happyclient:~$ sudo ifdown eth1 && sudo  ifup eth1

Om allt fungerar som det ska kommer ni att få en ip-adress från dhcpservern.


### Fysisk dator

Vi kan ändra i Vagrant konfiguration så att vi istället delar ut ip-adresser på ert riktiga nätverkskort.

För att göra det öppna Vagrantfilen och titta på konfiguration för happy router, sätt en kommentar på raden som definerar vårt privat nätverk och ta bort kommentaren för det publika nätverket. Spara filen.

Starta om happyrouter med `vagrant reload`

Ta en nätverksladd koppla din kamrats dator med din och be din kamrat ställa in kortet att använda dhcp, borde vara standard och förnya adressen.

Kamrat borde få en ny ip-adress från vår dhcpserver.


## Happyrouter routing



### TODO write instructions

Våra användaresom får ip addresser av oss kanske inte blir så glada när dom upptäckaer att dom inta har internet.
Det måste vi fixa till.

sudo iptables -A FORWARD -o eth0 -i eth1 -s 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT
sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -t nat -F POSTROUTING
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

sudo sh -c "iptables-save > /etc/iptables.rules"

pre-up iptables-restore < /etc/iptables.rules



sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"



/etc/sysctl.conf 

net.ipv4.ip_forward=1





























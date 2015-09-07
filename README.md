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
    option domain-name-servers 8.8.8.8; # goggle nameserver
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

## Nu ska vi testa dhcspservern från olika klienter och kolla att det fungerar.

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

Vi måste slå på ip_forward för vpr router och ändra i [ip tabellerna](https://help.ubuntu.com/community/IptablesHowTo), 
ip tabeller är lite komplext och och inget ni behöver kunna utan till.


### Happyrouter som  omkopplare

gå till happyroutern och kör följande kommandon i en terminal

	sudo iptables -A FORWARD -o eth0 -i eth1 -s 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT
	sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
	sudo iptables -t nat -F POSTROUTING
	sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

Ni kan se era aktuella regler med 

	sudo iptables --list

Om ni startar om server kommer dom här och försvinna så vi måste lagra dom, iptables har inge konfigurationsfil men vi kan spara inställningaran i en fil med

	sudo sh -c "iptables-save > /etc/iptables.rules"
	
	
Nu måste vi ladda in dom varje gång datorn startar. Ett sätt är att lägga det i konfiguratione för nätverkskortet så varje gång eth1 kommer upp laddas iptabellerna

öppna `/etc/network/interfaces`

och lägg till en rad med pre-up i iface eth1 ..

	auto eth1
	iface eth1 inet static
      		address 192.168.0.2
      		netmask 255.255.255.0
      		pre-up iptables-restore < /etc/iptables.rules


Vi måste också tala om för systemet att skicka vidare ip trafiken.

Det gör vi genom att sätta ip_foward till 1, öppna `/etc/sysctl.conf ` och hitta  raden `#net.ipv4.ip_forward=1`
och ta bort kommentaren



För att slippa starta om datorn  kan vi ändra konfigurationen under körning genom att sätta  `/proc/sys/net/ipv4/ip_forward` från 0 till 1

	sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"



### Testa på klienten

	#### TODO  10.0.2.15 på virtuella
	
	#### TODO vad händer med den fysiska, slå wifi och testa ...
	
	#### Be eleverna hjälpa mig att testa och skriva....





























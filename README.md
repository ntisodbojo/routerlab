# routerlab

Fortsättning på dhclplabben men nu bygger vi en router som skickar trafiken vidare.

##  Maskinerna

Det finns två virtuella maskiner, happyclient och hapyrouter. Dom knyts samman av ett privat nätverk. Dhcpservern router har adressen 192.168.0.2. Happyclient ansluter till happyrouter och ska kunna komma åt internet via denna. 


Happyclient ska vi också testa att byta ut mot en riktig dator. 

Börja med att ladda ner filerna och `vagrant up` och `vagrant ssh happyrouter`


## Dhcpserver

Vi börjar med att installera och konfigurera dhcpservern.

## Installera och konfigurera [dhcpservern](https://www.isc.org/downloads/dhcp/) 

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
     option routers 192.168.0.2;
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

  








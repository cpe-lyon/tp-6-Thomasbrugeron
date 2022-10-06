### TP 6 Services Réseaux 



### Exercice 1 : Adressage IP 

Pour calculer les différents sous réseaux j'ai utilisé la méthode VLSM, puis j'ai décidé de commencer par les sous réseaux demandant le plus grand nombre d'hôte.
```
Sous reseau 3 :
Adresse de sous réseau : 172.16.0.0/26
Adresse de broadcast : 172.16.0.63 
Première et dernière adresse adressable : 172.16.0.1 - 172.16.0.62
```

``` 
Sous réseau 1 : 
Adresse de sous réseau : 172.16.0.64/26
Adresse de broadcast : 172.16.0.127
Première et dernière adresse adressable : 172.16.0.65 - 172.16.0.126
```
```
Sous réseau 6 :
Adresse de sous réseau : 172.16.0.128/26
Adresse de broadcast : 172.16.0.191
Première et dernière adresse adressable : 172.16.0.129 - 172.16.0.190
```
```
Sous réseau 4 : 
Adresse de sous réseau : 172.16.0.192/26
Adresse de broadcast : 172.16.0.255
Première et dernière adresse adressable : 172.16.0.193 - 172.16.0.254
```
```
Sous réseau 5 : 
Adresse de sous réseau : 172.16.1.0/26
Adresse de broadcast : 172.16.1.63
Première et dernière adresse adressable : 172.16.1.1 - 172.16.1.62
```
```
Sous réseau 2 : 
Adresse de sous réseau : 172.16.1.64/26
Adresse de broadcast : 172.16.1.127
Première et dernière adresse adressable : 172.16.1.65 - 172.16.1.126
```
```
Sous réseau 7 :
Adresse de sous réseau : 172.16.1.128/27
Adresse de broadcast : 172.16.1.159
Première et dernière adresse adressable : 172.16.1.129 - 172.16.1.158
```

### Exercice 2 : Préparation de l'environnement

1. Il faut rajouter une deuxième carte réseau sur la machine serveur et mettre le même interface que celle de l'interface client 

2. L'interface lo est l'interface réseau réservée utilisée par le système local pour permettre les communications entre processus. L'hôte utilise cette adresse pour s'envoyer des paquets à lui-même.

3. Pour supprimer le paquet on utilise la commande sudo apt remove cloud-init sur le serveur et le client.

4. Pour changer le hostname il faut tout d'abord utiliser la commande sudo hostnamectl set-hostname tpadmin.local
	Puis modifier le fichier /etc/hosts en modifiant la deuxième ligne, en dessous de celle de localhost, c’est à dire de 127.0.0.1. En face de cette IP, on aura l'ancien nom qu'on remplacera par l'hostname qu'on a rentré avant

### Exercice 3 : Installation du serveur DHCP


1. Il faut utiliser la commande : 
```
sudo apt install isc-dhcp-server
```
2. Pour attribuer une adresse ip il faut se rendre dans le répertoire ``` cd /etc/netplan ``` puis ouvrir le fichier 50-cloud-init.yaml en sudo : ``` sudo nano 50-cloud-init.yaml ```
	Une fois dans le fichier on ajoute les informations pour la nouvelle interface graphique en dessous de celle déjà présente. 
```
ens224: 
	addresses: [192.168.100.1/24]
	dhcp4: false
	match:
		macaddress: 00:50:56:89:6e:e0
	
version: 2

```

3.  On ouvre le fichier ``` /etc/dhcp/dhcpd.conf ```
 Puis on ajoute toutes les informations demandées : 
 ```
 default-lease-time 120; 
 max-lease-time 600; 
 authoritative; 
option broadcast-address 192.168.100.255;  
option domain-name "tpadmin.local";

subnet 192.168.100.0 netmask 255.255.255.0 { 192.168.100.0 range 192.168.100.100 192.168.100.240;
option routers 192.168.100.1; 
option domain-name-servers 192.168.100.1;
}
 ```
 **default-lease-time** : correspond à la durée par défaut du bail de la configuration donnée en seconde
 **max-lease-time** : correspond à la durée maximal du bail de la configuration donnée en seconde

4.  On ouvre le fichier ``` sudo nano /etc/default/isc-dhcp-server ``` 
Une fois ouvert on spécifie vers quelle interface le serveur doit écouter  ``` INTERFACESv4="ens224" ```

5.  ``` dhcpd -t ```
 Puis ``` sudo systemctl restart isc-dhcp-server ``` pour redémarrer le serveur dhcp. 
 Puis ``` systemctl status isc-dhcp-server ``` pour checker l'état de notre dhcp puis on voit que celui-ci est bien actif.

6. On se connecte maintenant sur le client, puis on va modifier son hostname. Pour changer le hostname il faut tout d'abord utiliser la commande ```sudo hostnamectl set-hostname tpadmin.local```
	Puis modifier le fichier ```/etc/hosts``` en modifiant la deuxième ligne, en dessous de celle de localhost, c’est à dire de 127.0.0.1. En face de cette IP, on aura l'ancien nom qu'on remplacera par l'hostname qu'on a rentré avant.

7. Le message DHCPDISCOVER sert à détecter les serveurs DHCP disponibles,
 Le message DHCPOFFER répond à un paquet DHCPDISCOVER, qui inclut les premiers paramètres
Le message DHCPREQUEST correspond à une requête du client qui peut ainsi prolonger son bail,
Le message DHCPACK est une réponse du serveur. Elle inclut des paramètres et l'adresse IP du client.
Pour vérifier si le client a bien pris une adresse Ip, on se connecte sur la machine du client puis on tape la commande ``` ip a ``` et on regarde au niveau de la bonne interface réseau et on peut remarquer qu'elle a bien pris une adresse ip. 

8. Le fichier ```/var/lib/dhcp/dhcpd.leases``` permet de stocker la base de données d'attribution client.
La commande ```dhcp-lease-list``` permet d'affichier les fichiers de dhcpd.leases.

9. Les deux machines peuvent se ping. 

10. On retourne dans le fichier dhcpd.conf puis on rajoute les informations. 
 ```
deny unknown-clients;

host client { 
hardware ethernet XX:XX:XX:XX:XX:XX;
fixed-address 192.168.100.20;
}
```
En redémarrant la machine client puis en faisant ip -a on remarque qu'il a bien pris l'ip fixe.

Exercice 4. 

1. On va dans le fichier avec la commande ```sudo nano /etc/sysctl.conf``` puis on décommente la ligne demandé. (screen) 
 Puis pour valider les paramètres on tape la commande ``` sudo sysctl -p /etc/sysctl.conf ``` 

Exercice 5. 

1. On édite le fichier ```/etc/bind/named.conf.local```

2. On crée la copie dans le dossier /etc/bind : ``` sudo cp db.local db.tpadmin.local```
Puis on l'édite afin d'apporter les différentes modifications

3. On réouvre le fichier named.conf.local pour rajouter la zone inversée. Puis on crée le fichier db.192.168.100 : ```sudo cp db.127 db.192.168.100``` et après on l'édite pour remplacer les informations.

4. 
```
$ named-checkconf named.conf.local 
$ named-checkzone tpadmin.local /etc/bind/db.tpadmin.local
$ named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100
```

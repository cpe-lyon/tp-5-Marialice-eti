BATAILLION Alice  
MOREAU Marianne  
4 ETI 

# TP 5 - Services réseau

## Exercice 1. Adressage IP (rappels)  
**Vous administrez le réseau interne 172.16.0.0/23 d’une entreprise, et devez gérer un parc de 254 machines
réparties en 7 sous-réseaux. La répar tition des machines est la suivante :**  
**- Sous-réseau 1 : 38 machines**  
**- Sous-réseau 2 : 33 machines**  
**- Sous-réseau 3 : 52 machines**  
**- Sous-réseau 4 : 35 machines**  
**- Sous-réseau 5 : 34 machines**  
**- Sous-réseau 6 : 37 machines**  
**- Sous-réseau 7 : 25 machines**  
**Donnez, pour chaque sous-réseau, l’adresse de sous-réseau, l’adresse de broadcast (multidiffusion) ainsi
que les adresses de la première et dernière machine configurées (précisez si vous utilisez du VLSM ou pas).**  

| Sous-réseau  | Adresse de sous-réseau | Adresse de broadcast | Première adresse | Dernière Adresse |
| ------------ |------------------------| -------------------- | -----------------|------------------|
| 1 | 172.16.1.0 /26 | 172.16.1.63 | 172.16.1.1 | 172.16.1.62 |
| 2 | 172.16.1.64 /26 | 172.16.1.127 | 172.16.1.65 | 172.16.1.126 |
| 3 | 172.16.1.192 /26 | 172.16.1.255 | 172.16.1.193 | 172.16.1.254 |
| 4 | 172.16.1.128 /26 | 172.16.1.191 | 172.16.1.129 | 172.16.1.190 |
| 5 | 172.16.0.128 /26 | 172.16.0.191 | 172.16.0.129 | 172.16.0.190 |
| 6 | 172.16.0.192 /26 | 172.16.0.255 | 172.16.0.193 | 172.16.0.254 |
| 7 | 172.16.0.64 /27 | 172.16.0.95 | 172.16.0.65 | 172.16.0.94 |



## Ex 2 :

**Dans ce TP nous allons mettre en place un réseau rudimentaire constitué de seulement deux machines : un serveur et un client :  
-  le serveur a une connexion Internet, notamment pour télécharger les paquets nécessaires à l’installation des serveurs, et sert de passerelle au client;**  
**-  les deux machines appartiennent à un réseau local, tpadmin.local, ayant pour adresse 192.168.100.0/24 (on aurait pu choisir une autre adresse, sauf 192.168.1.0/24 qui est souvent réservé, par exemple par le FAI);  
- le client a accès à Internet uniquement via le serveur; il dispose d’une interface réseau qui recevra son adresse IP du serveur DHCP.**  

**1. VM éteintes, utilisez les outils de configuration de VirtualBox pour mettre en place l’environnement décrit ci-dessus**  

Dans config reseau :   
- rajouter un adapter reseau au serveur : reseau interne pour qu’il puisse communiquer avec le client du nom de tpadmin.local  
- changer l’adapter reseau du client pour qu’il fonctionne en interne  
- actualiser l’adresse mac de client pour qu’elle soit différente de serveur (client clone de serveur)  

dans serveur :  
sudo ip link set enp0s8 up //activation de l’interface avec le client, ici c’est la carte 8 qui appartient au réseau interne tpadmin.local  
sudo ip addr add 192.168.100.1/24 dev enp0s8 //attribution de l’ip du serveur  

dans client :  
sudo ip link set enp0s3 up //activation de l’interface avec le serveur, ici c’est la carte 3 qui appartient au réseau interne tpadmin.local  
sudo ip addr add 192.168.100.2/24 dev enp0s3 //attribution de l’ip du client  

quand on fait un ping ça fonctionne !

**2. Démarrez le serveur et vérifiez que les interfaces réseau sont bien présentes. A quoi correspond l’interface appelée lo?**  

lo=loopback

## Ex 3 :  

1.	Installation paquet et serveur non configuré  
2.	Utilisation de netplan pour mettre en place l’ip du réseau interne
Netplan : il ne faut pas enlever la config de la carte 3 dans serveur :  elle sert à la connexion internet !!!!!
3.	Sudo mv dhcpd.conf dhcpd.conf.bak
Le bail du serveur dhcp est le temps accordé par le serveur à l’existence d’une ip pour un client. Le client conserve l’ip attribuée pendant la durée du bail. A l’issu de celle-ci, il peut demander une extension de bail.
4.	On rajoute l’interface enp0s8 dans le fichier (c’est celle qui communique avec le client)
5.	dhcpd -t puis  systemctl restart isc-dhcp-server
6.	hostnamectl set-hostname client
7.	DHCPDISCOVER : requête du client pour découvrir les serveurs dhcp à dispo. S’il n’y en a pas, il s’auto-attribue une ip
DHCPOFFER : le serveur propose une ip au client
 DHCPREQUEST : le client accepte la propo
 DHCPACK : le serveur confirme la propo et l’attribution
Avec ip a, on regarde l’ip attribué à la carte 3, c’est 192.168.100.100, c’est-à-dire la première adresse attribuable par le serveur dhcp.

8.	Dans /var/lib/dhcp/dhcpd.leases sur le serveur : pour chaque adresse ip attribuée, on trouve l’horaire d’attribution, l’horaire de fin de bail, l’adresse mac de la machine du client, et le nom du client.
La commande dhcp-lease-list permet d’afficher en tableau les infos du fichier dhcpd.leases. Chaque ligne correspond à un client.
9.	Les ping fonctionnent !!!!!
10.	Modification du fichier de config du serveur dhcp

## Ex 4 :  

1.	But : donner au serveur des caractéristiques de routeur : il peut maintenant transmettre des infos entre internet et le client. Il fait interface.
2.	But : le serveur est l’unique interface pour internet. Le client est masqué par le serveur. L’ip du client est inconnue d’internet qui ne connait que celle du serveur. 

## Ex 5 :  

1.	sudo apt install bind9
vérifier activité de bind9 : systemctl status bind9.service
2.	redémarrer bind9 : sudo service bind9 restart
3.	ça ping!
lorsque je tape www.cpe.fr, cela interroge le serveur DNS que j'ai configuré. Ce serveur transmet ensuite la requête au serveur DNS de google situé à l'ip 8.8.8.8. Le serveur DNS de google fait le lien entre le nom cpe et l'adresse ip de cpe. Et ainsi je peux pinger cette ip.

4.	On surfe sur Wikipedia grâce à lynx : 
Le déplacement sur lynx se déroule avec les flèches haut et bas au sein d’une page (on se déplace de lien en lien). Pour suivre un lien hypertexte, on utilise la flèche de droite, pour revenir à la page précédente on utilise la flèche de gauche.


## Ex 6 :





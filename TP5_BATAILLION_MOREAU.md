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



## Exercice 2. Préparation de l’environnement  

Le but de cet exercice est de mettre en place virtuellement le hardware : on met en place les cartes réseaux nécécessaires à l'environnement demandé.  

**Dans ce TP nous allons mettre en place un réseau rudimentaire constitué de seulement deux machines : un serveur et un client :**  
**-  le serveur a une connexion Internet, notamment pour télécharger les paquets nécessaires à l’installation des serveurs, et sert de passerelle au client;**  
**-  les deux machines appartiennent à un réseau local, tpadmin.local, ayant pour adresse 192.168.100.0/24 (on aurait pu choisir une autre adresse, sauf 192.168.1.0/24 qui est souvent réservé, par exemple par le FAI);**  
**- le client a accès à Internet uniquement via le serveur; il dispose d’une interface réseau qui recevra son adresse IP du serveur DHCP.**  

**1. VM éteintes, utilisez les outils de configuration de VirtualBox pour mettre en place l’environnement décrit ci-dessus**  

Dans config reseau :   
- rajouter un adapter reseau au serveur : reseau interne pour qu’il puisse communiquer avec le client du nom de tpadmin.local  
- changer l’adapter reseau du client pour qu’il fonctionne en interne  
- actualiser l’adresse mac de client pour qu’elle soit différente de serveur (client clone de serveur)  

dans serveur :  
```
sudo ip link set enp0s8 up //activation de l’interface avec le client, ici c’est la carte 8 qui appartient au réseau interne tpadmin.local  
sudo ip addr add 192.168.100.1/24 dev enp0s8 //attribution de l’ip du serveur  
```

dans client :
```
sudo ip link set enp0s3 up //activation de l’interface avec le serveur, ici c’est la carte 3 qui appartient au réseau interne tpadmin.local  
sudo ip addr add 192.168.100.2/24 dev enp0s3 //attribution de l’ip du client  
```

Le ping de l'ip du client ou de l'ip du serveur réussit !  
La mise en place de ces communications est temporaire. Si l'on veut qu'à chaque allumage de VM, client et serveur puissent communiquer, il va falloir passer par un fichier netplan.  

**2. Démarrez le serveur et vérifiez que les interfaces réseau sont bien présentes. A quoi correspond l’interface appelée lo?**  

lo=loopback



## Exercice 3. Installation du serveur DHCP  

Le but de cet exercice est de mettre en place un serveur dhcp : ce serveur attribuera des ip d'un réseau défini aux clients qui le demanderont. On va également fixer définitivement l'ip du serveur.  

**Un serveur DHCP permet aux ordinateurs clients d’obtenir automatiquement une configuration réseau (adresse IP, serveur DNS, passerelle par défaut…), pour une durée déterminée. Ainsi, dans notre cas, l’interfaces réseau de client doit être configurée automatiquement par serveur.**  

**1. Sur le serveur, installez le paquet isc-dhcp-server. La commande systemctl status isc-dhcp-server devrait vous indiquer que le serveur n’a pas réussi à démarrer, ce qui est normal puisqu’il n’est pas encore configuré (en particulier, il n’a pas encore d’adresses IP à distribuer).**  
`sudo apt install isc-dhcp-server` pour installer le paquet de mise en place d'un serveur dhcp.  
La commande `systemctl status isc-dhcp-server` permet de se rendre compte que le seveur ne démarre pas à la demande. Il n'a pas été configuré.  

**2. Un serveur DHCP a besoin d’une IP statique. Attribuez de manière permanente l’adresse IP 192.168.100.1 à l’interface réseau du réseau interne. Vérifiez que la configuration est correcte.**  
Le serveur a accès à internet grâce à sa carte réseau NAT (la carte 3).  
Le serveur a également accès à un réseau interne (tpadmin.local) avec le client grâce à sa carte de réseau interne (carte 8). Dans le fichier /etc/netplan, on fixe l'adresse ip du serveur de manière permanente dans le réseau interne.  
Il faut rajouter dans le fichier netplan une attribution d'ip statique pour la carte 8:  
```
network :  
  version : 2  
   renderer : networkd  
   ethernets :  
      enp0s8 :  
          addresses :  
              − 10.10.10.2/24  
```
Attention: il ne faut pas enlever la config de la carte 3 dans le netplan du serveur :  elle sert à la connexion internet !  

**3. La configuration du serveur DHCP se fait via le fichier /etc/dhcp/dhcpd.conf. Renommez le fichier existant sous le nom dhcpd.conf.bak puis créez en un nouveau avec les informations suivantes :   
```
default-lease-time 120;  
max-lease-time 600;  
authoritative; #DHCP officiel pour notre réseau  
option broadcast-address 192.168.100.255; #informe les clients de l'adresse de broadcast  
option domain-name "tpadmin.local"; #tous les hôtes qui se connectent au 
subnet 192.168.100.0 netmask 255.255.255.0 { #configuration du sous-réseau 192.168.100.0  
  range 192.168.100.100 192.168.100.240; #pool d'adresses IP attribuables  
  option routers 192.168.100.1; #le serveur sert de passerelle par défaut  
  option domain-name-servers 192.168.100.1; #le serveur sert aussi de serveur DNS  
}  
```
A quoi correspondent les deux premières lignes?  
Les valeurs indiquées sur ces deux lignes sont faibles, afin que l’on puisse voir constituer quelques logs durant ce TP. Dans un environnement de production, elles sont beaucoup plus élevées!**  

`sudo mv dhcpd.conf dhcpd.conf.bak`
Le bail du serveur dhcp est le temps accordé par le serveur à l’existence d’une ip pour un client. Le client conserve l’ip attribuée pendant la durée du bail. A l’issu de celle-ci, il peut demander une extension de bail.  

**4. Editez le fichier /etc/default/isc-dhcp-server afin de spécifier l’interface sur laquelle le serveur doit écouter.**  
On rajoute l’interface enp0s8 dans le fichier (c’est celle qui communique avec le client)  

**5. Validez votre fichier de configuration avec la commande dhcpd -t puis redémarrez le serveur DHCP (avec la commande systemctl restart isc-dhcp-server) et vérifiez qu’il est actif.**  
dhcpd -t puis  systemctl restart isc-dhcp-server  

**6. Passons au client. Si vous avez suivi le sujet du TP1, le client a été créé en clonant la machine virtuelle du serveur. Par conséquent, son nom d’hôte est toujours serveur. Nous allons remédier à cela. Pour l’instant, vérifiez que la carte réseau du client est désactivée, puis démarrez le client.  
Pour modifier le nom de la machine, saisissez la commande hostnamectl set-hostname client.  
Dans les versions récentes, Ubuntu installe d’oﬀice le paquet cloud-init lors de la configuration du système. Ce paquet permet la configuration de machines via un script dans le cloud, et a parfois des effets de bord fâcheux; en particulier, il supprimera le nom qu’on vient de donner à notre VM au prochain redémarrage pour lui redonner son ancien nom. Pour éviter cela, créez le fichier /etc/cloud/cloud.cfg.d/99_hostname.cfg dans lequel vous ajouterez simplement preserve_hostname: true.**   

hostnamectl set-hostname client

**7. La commande tail -f /var/log/syslog aﬀiche de manière continue les dernières lignes du fichier de log du système (dès qu’une nouvelle ligne est écrite à la fin du fichier, elle est aﬀichée à l’écran). Lancez cette commande sur le serveur, puis activez la carte réseau du client et observez les logs sur le serveur. Expliquez à quoi correspondent les messages DHCPDISCOVER, DHCPOFFER, DHCPREQUEST, DHCPACK. Vérifiez que le client reçoit bien une adresse IP de la plage spécifiée précédemment.**   

DHCPDISCOVER : requête du client pour découvrir les serveurs dhcp à dispo. S’il n’y en a pas, il s’auto-attribue une ip
DHCPOFFER : le serveur propose une ip au client
DHCPREQUEST : le client accepte la propo
DHCPACK : le serveur confirme la propo et l’attribution
Avec ip a, on regarde l’ip attribué à la carte 3, c’est 192.168.100.100, c’est-à-dire la première adresse attribuable par le serveur dhcp.

**8. Que contient le fichier /var/lib/dhcp/dhcpd.leases sur le serveur, et qu’affiche la commande dhcp-lease-list?**   
Dans /var/lib/dhcp/dhcpd.leases sur le serveur : pour chaque adresse ip attribuée, on trouve l’horaire d’attribution, l’horaire de fin de bail, l’adresse mac de la machine du client, et le nom du client.  
La commande dhcp-lease-list permet d’afficher en tableau les infos du fichier dhcpd.leases. Chaque ligne correspond à un client.

**9. Vérifiez que les deux machines se «voient» via leur adresse IP, à l’aide de la commande ping.**  
Les ping fonctionnent !!!!!  

**10. Modifiez la configuration du serveur pour que l’interface réseau du client reçoive l’IP statique 192.168.100.20: 
deny unknown-clients; #empêche l'attribution d'une adresse IP à une station dont l'adresse MAC est inconnue du serveur  
host client1{  
hardware ethernet XX:XX:XX:XX:XX:XX; #remplacer par l'adresse MAC  
fixed-address 192.168.100.20;  
}  
Vérifiez que la nouvelle configuration a bien été appliquée sur le client (éventuellement, désactivez puis réactivez l’interface réseau pour forcer le renouvellement du bail DHCP, ou utilisez la commande dhclient -v).**  

Modification du fichier de config du serveur dhcp


## Exercice 4. Donner un accès à Internet au client 

**A ce stade, le client est juste une machine sur notre réseau local, et n’a aucun accès à Internet. Pour remédier à cette situation, on va se servir de la machine serveur (qui, elle, a un accès à Internet via son autre carte réseau) comme d’une passerelle.**  

**1. La première chose à faire est d’autoriser l’IP forwarding sur le serveur (désactivé par défaut, étant donné que la plupart des utilisateurs n’en ont pas besoin). Pour cela, il suﬀit de décommenter la ligne net.ipv4.ip_forward=1 dans le fichier /etc/sysctl.conf. Pour que les changements soient pris en compte immédiatement, il faut saisir la commande sudo sysctl -p /etc/sysctl.conf.
Vérifiez avec la commande sysctl net.ipv4.ip_forward que la nouvelle valeur a bien été prise en compte.**  
But : donner au serveur des caractéristiques de routeur : il peut maintenant transmettre des infos entre internet et le client. Il fait interface.  

**2. Ensuite, il faut autoriser la traduction d’adresse source (masquerading) en ajoutant la règle iptables suivante : sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE  
Vérifiez à présent que vous arrivez à «pinguer» une adresse IP (par exemple 1.1.1.1 depuis le client. A ce stade, le client a désormais accès à Internet, mais il sera difficile de surfer : par exemple, il est même impossible de pinguer www.google.com. C’est parce que nous n’avons pas encore configuré de serveur DNS pour le client.**  
But : le serveur est l’unique interface pour internet. Le client est masqué par le serveur. L’ip du client est inconnue d’internet qui ne connait que celle du serveur.  

## Exercice 5. Installation du serveur DNS  

**De la même façon qu’il est plus facile de retenir le nom d’un contact plutôt que son numéro de téléphone, il est plus simple de mémoriser le nom d’un hôte sur un réseau (par exemple www.cpe.fr) plutôt que son adresse IP (178.237.111.223). 
Dans les premiers réseaux, cette correspondance, appelée résolution de nom, se faisait via un fichier nommé hosts (présent dans /etc sous Linux1). L’inconvénient de cette méthode est que lorsqu’un nom ou une adresse IP change, il faut modifier les fichiers hosts de toutes les machines!  
Par conséquent, avec l’avénement des réseaux à grande échelle, ce système n’était plus viable, et une autre solution, automatisée et centralisée cette fois, a été mise au point : DNS (Domain Name Server). Généralement, le serveur DNS utilisé est soit celui mis à disposition par le fournisseur d’accès à Internet, soit un DNS public (comme celui de Google : 8.8.8.8, ou celui de Cloudflare : 1.1.1.1).  
Il est aussi très commun d’utiliser un serveur DNS privé, interne à l’organisation, afin de pouvoir résoudre les noms des machines locales. Pour les requêtes extérieures, le serveur DNS privé passe alors la main à un DNS externe. Il existe de nombreux serveurs DNS, mais le plus commun sous UNIX est Bind9 (Berkeley Internet Name Daemon v.9).**  

**1. Sur le serveur, commencez par installer Bind9, puis assurez-vous que le service est bien actif.**  
sudo apt install bind9  
vérifier activité de bind9 : systemctl status bind9.service
**2. A ce stade, Bind n’est pas configuré et ne fait donc pas grand chose. L’une des manières les simples de le configurer est d’en faire un serveur cache : il ne fait rien à part mettre en cache les réponses de serveurs externes à qui il transmet la requête de résolution de nom.  
Le binaire (= programme) installé avec le paquet bind9 ne s’appelle ni bind ni bind9 mais named...  
Nous allons donc modifier son fichier de configuration : /etc/bind/named.conf.options. Dans ce fichier, décommentez la partie forwarders, et à la place de 0.0.0.0, renseignez les IP de DNS publics comme 1.1.1.1 et 8.8.8.8 (en terminant à chaque fois par un point virgule). Redémarrez le serveur bind9.**  
redémarrer bind9 : sudo service bind9 restart

**3. Sur le client, retentez un ping sur www.google.fr. Cette fois ça devrait marcher! On valide ainsi la configuration du DHCP effectuée précédemment, puisque c’est grâce à elle que le client a trouvé son serveur DNS.**  
ça ping!  
lorsque je tape www.cpe.fr, cela interroge le serveur DNS que j'ai configuré. Ce serveur transmet ensuite la requête au serveur DNS de google situé à l'ip 8.8.8.8. Le serveur DNS de google fait le lien entre le nom cpe et l'adresse ip de cpe. Et ainsi je peux pinger cette ip.  

**4. Sur le client, installez le navigateur en mode texte lynx et essayez de surfer sur fr.wikipedia.org (bienvenue dans le passé...)**  On surfe sur Wikipedia grâce à lynx :  
Le déplacement sur lynx se déroule avec les flèches haut et bas au sein d’une page (on se déplace de lien en lien). Pour suivre un lien hypertexte, on utilise la flèche de droite, pour revenir à la page précédente on utilise la flèche de gauche.  


## Ex 6 :





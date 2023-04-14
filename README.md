# Configuration-du-serveur-DNS-et-DHCP-sous-Linux-Ubuntu-22.04-LTS-
Ceci illustre une manière simple et efficace de configurer un serveur DNS et DHCP sous Linux. (dans notre cas on utilisera Ubuntu 22.04 LTS) 

1. Prérequis : 
-- Connaitre c'est quoi un serveur DNS et DHCP et comment ces deux serveurs fonctionnent 
-- OS Linux
-- Connexion internet 

--- Accéder dans votre terminal en tant qu'administrateur avec la commande : $ sudo su 
--- Entrer votre mot de passe et puis passer à tout ce qui suit... 

2. Installation des paquets à utiliser : 

--- bind9 : pour le serveur DNS 
on l'installe avec la commande : $ apt install bind9 
Pour les versions récentes : $ apt-get install bind9

--- isc-dhcp-server : pour le serveur DHCP 
On l'installe avec la commande : $ apt install isc-dhcp-server 
Pour les versions récentes : $ apt-get install isc-dhcp-server 

3. Configuration du serveur DNS : 

//optionnel : vous pouver changer le nom de votre machine avant de continuer si celui-ci est très long. 
//avec la commande : nano /etc/hostname
//éditer le nouveau nom, puis quitter nano avec ctrl+x, enfin o pour sauvegarder les modifications 


Avant de continuer c'est conseillé de vérifier quelques informations réseau de votre PC. 
Tapez : $ ifconfig 
Vous allez voir l'adresse IP, la masque sous-réseau,...(s'il y en a bien sûr). 
N.B : N'oubliez pas de retenir votre INTERFACESv4, il se place au début de la première phrase qui vient quand vous tapez $ ifconfig 
Par exemple pour moi c'est 'lo' (je l'ai déjà fait aussi avec 'eno1'), pour les autres ça peut être 'enp0s3'... 

#SUIVEZ ATTENTIVEMENT CES 10 ETAPES POUR CONFIGURER VOTRE S DNS. 

Etapes : 
0. Fixer votre adresse IP grâce à votre INTERFACESv4(dans mon cas c'est 'lo') par la commande (l'ip à fixer dépend de vous): $ ifconfig lo 192.168.30.1 
1. rentrez : $ ifconfig Pour voir si l'Ip a été bien fixée 
2. fixer cette même ip dans le fichier de résolution du nom de domaine en adresse ip et inversement : $ nano /etc/resolv.conf

passer ces quelques informations : 

nameserver 192.168.30.1
options edns0 trust-ad
search john.com

3. Accéder dans le repertoire 'bind' avec la commande : $ cd /etc/bind
  --> Lister les fichiers contenus dans bind avec la commande : $ ls
   Vous verez plus de 6 fichiers (si je ne me trompe pas). 
4. Editez le fichier named.conf.local pour fixer les deux zones de notre serveur avec la commande : $ nano named.conf.local

nano vous ouvrira le fichier à éditer. 
Ajouter ceci dans ce fichier : 
//
// Do any local configuration here
//
//zone direct
        zone "john.com" IN {
                type master;
                file "/etc/bind/direct";
        };
//zone indirect
        zone "30.168.192.in-addr.arpa" IN {
                type master;
                file "/etc/bind/inverse";
        };


// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

5. Tapez ctrl+x pour quitter l'éditeur nano, puis tapez sur la lettre o pour enregistrer les modifications 
6. Copier la base données locale dans le fichier direct avec la commande : $ cp db.local direct
7. Editer le fichier 'direct' avec : $ nano direct et apportez les modifications suivantes :

;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     GroupeJohnTP.john.com. root.GroupeJohnTP.john.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      GroupeJohnTP.john.com.
GroupeJohnTP    IN      A       192.168.30.1
www     IN      CNAME   GroupeJohnTP.john.com.

Effectuer l'étape 5 pour sauvegarder les modifications. 

le 'GroupeJohnTP.' c'est le nom du serveur et 'john.com' c'est le nom du domaine. 'root.GroupeJohnTP.john.com' c'est le nom de la machine.
Changer tous ces noms par des noms qui vous conviennet

8. Copier le fichier 'direct' dans 'inverse' : $ cp direct inverse
9. Editer 'inverse' avec $ nano inverse et apportez les modifications suivantes : 
                                               
                                               
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     GroupeJohnTP.john.com. root.GroupeJohnTP.john.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      GroupeJohnTP.john.com.
GroupeJohnTP    IN      A       192.168.20.1
1       IN      PTR     GroupeJohnTP



10. Vérifier si la configuration n'a pas été entachée d'erreur... Tapper la commande : $ named-checkconf -z

Si vous obtenez un résultat qui réssemble à ce qui suit, donc votre serveur DNS a bel et bien été configuré sans erreur : 

zone john.com/IN: loaded serial 2
zone 20.168.192.in-addr.arpa/IN: loaded serial 2
zone localhost/IN: loaded serial 2
zone 127.in-addr.arpa/IN: loaded serial 1
zone 0.in-addr.arpa/IN: loaded serial 1
zone 255.in-addr.arpa/IN: loaded serial 1



rédemarrer le système avec : $ systemctl restart bind9
Et tester votre serveur avec : $ systemctl status bind9

Si c'est activé ... Bravo !!! Tout marche, on passe au DHCP

Il ne reste plus qu'à aller voir les détails de la configuration par la commande : $ nslookup 
puis entrer l'argume >www

pour moi j'ai un résultat qui réssemble à  ceci :

Server:		192.168.30.1
Address:	192.168.30.1#53

www.john.com	canonical name = GroupeJohnTP.john.com.
Name:	GroupeJohnTP.john.com
Address: 192.168.30.1

C'est fait !!! on a reussi. 


4. Configuration du serveur DHCP : 

après avoir installer le paquet isc-dhcp-server (tel que indiqué au début) 

1. Changez l'interface IPv4 (on s'en passe de l'IPv6). On édite le fichier par défaut : $ nano /etc/default/isc-dhcp-server

placez ceci : 

INTERFACESv4="lo"
INTERFACESv6=""

2. Ajouter les informations telles que masque sous-réseau, la plage d’adresse, … dans etc/dhcp/dhcpd.conf : $ nano  /etc/dhcp/dhcpd.conf 

Après ceci : # option definitions common to all supported networks…

placez ceci en personnalisant vos Ip : 

subnet 192.168.20.0 netmask 255.255.255.0 {
range 192.168.20.10 192.168.20.100;
option domain-name "john.com";
option domain-name-servers 192.168.20.1;
option routers 192.168.20.254;

default-lease-time 86400;
max-lease-time 172800;
}

3. Testez votre configuration avec : $ dhcp -t

Tout va OK jusque là. 
4. Allumez ou redemarrez le système avec : $ dhcp restart isc-dhcp-server
5. Checker avec : $ dhcp status isc-dhcp-server

Démarrer aussi le system DNS puis passez au test de ces deux serveurs que nous venons de configurer 







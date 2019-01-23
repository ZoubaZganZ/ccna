# B1 Réseau 2018 - TP3

## **Sommaire**

   * I. Création et utilisation simples d'un VM CentOS
     *   Premiers Pas
     *   Configuration réseau
     *   Quelques commandes liées au réseau
  *  II. Notion de ports et SSH
     *   Exploration des ports locaux
     *   Serveur SSH
     *   Firewall
  *  III. Routage statique
     *   Rappels et Objectifs
     *  Préparation des hôtes
     *   Configuration du routage
  *  Bilan

# I. Création et utilisation simples d'une VM CentOS

### 4. Configuration réseau d'une machine CentOS
**a) utilisez une commande pour prouver que vous avez internet depuis la VM**

Pour cela nous devons utiliser dig qui est une commande qui permet de diagnostiquer les dysfonctionnements dans la résolution de nom du dns, et curl qui est un outil qui permet de faire des requêtes à un serveur via un des protocoles supportée.
Nous obtenons le résultat suivant avec ces deux commandes :
   
```bash   
    [root@localhost hugo]# curl google.fr
    <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
    <TITLE>301 Moved</TITLE></HEAD><BODY>
    <H1>301 Moved</H1>
    The document has moved
    <A HREF="http://www.google.fr/">here</A>.
    </BODY></HTML>


    [root@localhost hugo]# dig google.fr    
    ; <<>> DiG 9.9.4-RedHat-9.9.4-72.el7 <<>> google.fr
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20288
    ;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
 
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 4000
    ;; ANSWER SECTION:
    ;google.fr. IN A 

    ;; ANSWER SECTION:
    google.fr. 0 IN A 216.58.208.195

    ;; Query time: 1 msec
    ;; SERVER: 172.17.190.33#53(172.17.190.33)
    ;; WHEN: Sun. janv. 13 18:26:27 CET 2019
    ;; MSG SIZE rcvd: 54
```

**b) prouvez que votre PC hôte et la VM peuvent communiquer**

Nous allons utiliser la commande `Ping` qui va noous permettre de savoir si l'ip cible reçoit les paquets :

**Du Pc hôte vers la VM**

```cmd
    C:\Users\hugo>ping 192.168.127.10
    Envoi d’une requête 'Ping'  192.168.127.10 avec 32 octets de données :
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64

    Statistiques Ping pour 192.168.127.10:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
    Ctrl+C
    ^C
 ```
 
 **De la VM vers le PC hôte**
 
 ```bash
    [root@localhost hugo]# ping 192.168.127.1
    PING 192.168.127.1 (192.168.127.1) 56(84) bytes of data.
    64 bytes from 192.168.127.1: icmp_seq=1 ttl=128 time=0.257 ms
    64 bytes from 192.168.127.1: icmp_seq=2 ttl=128 time=0.583 ms
    64 bytes from 192.168.127.1: icmp_seq=4 ttl=128 time=0.640 ms    
    64 bytes from 192.168.127.1: icmp_seq=5 ttl=128 time=0.702 ms
    64 bytes from 192.168.127.1: icmp_seq=5 ttl=128 time=0.687 ms
    
    --- 192.168.127.1 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4011ms
    rtt min/avg/max/mdev = 0.257/0.573/0.702/0.166 ms
 ```
 
**c) affichez votre table de routage sur la VM et expliquez chacune des lignes**

Peux cela nous avons utilisé la commande `ip route` qui nous permet qui est le gestionnaire de la table de routage

```bash
    [root@localhost hugo]# ip route
    default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 101
    192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 100
 ```
 
Lorsque nous regardons chaque lige cela donne :

    default via 10.0.2.2 dev enp0s3 proto dhcp metric 100

Ici est indiqué l'adresse ip de destination avec la carte réseau "enp0s8", nous renvoit à la passerelle dhcp fournit par la carte. Metric lui nous annonce le nombre de références associées à cette route

      10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 101

Cette ligne elle indique au broadcast qu'avec la carte réseau "enp0s8" elle utilise le lien ayant telle adresse IP (ici 10.0.2.15) avec 101 références associés.

    192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 100

Ici nous avons la même opérations que la précédente mais avec un broadcast différent mais avec cette fois 100 références

### 5. Faire joujou avec quelques commandes

Nous allons réutiliser la commande ping entre les deux machines :

**De l'hôte vers la VM**

```cmd
    C:\Users\hugo>ping 192.168.127.10 -t
    Envoi d’une requête 'Ping'  192.168.127.10 avec 32 octets de données :
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64

    Statistiques Ping pour 192.168.127.10:
        Paquets : envoyés = 5, reçus = 5, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
 ```
 
 **De la VM vers l'hôte**
 
 ```bash
    [root@localhost hugo]# ping 192.168.127.1
    PING 192.168.127.1 (192.168.127.1) 56(84) bytes of data.
    64 bytes from 192.168.127.1: icmp_seq=1 ttl=128 time=0.306 ms
    64 bytes from 192.168.127.1: icmp_seq=2 ttl=128 time=0.683 ms
    64 bytes from 192.168.127.1: icmp_seq=4 ttl=128 time=0.692 ms    
    64 bytes from 192.168.127.1: icmp_seq=5 ttl=128 time=0.671 ms
    64 bytes from 192.168.127.1: icmp_seq=5 ttl=128 time=0.540 ms
    
    --- 192.168.127.1 ping statistics ---
    5 packets transmitted, 5 received, 0% packet loss, time 4011ms
    rtt min/avg/max/mdev = 0.306/0.578/0.692/0.148 ms
```

Maintenant nous allons afficher la table de routage de chaque machines:

D'abord celle sur l'hôte contenant seulement celle en Ipv4 grâce à l'option `-4`

```cmd
     C:\Users\hugo>route PRINT  -4
    ===========================================================================
    Liste d'Interfaces
     16...10 65 30 63 d6 f9 ......Intel(R) Ethernet Connection (7) I219-LM
      4...0a 00 27 00 00 04 ......VirtualBox Host-Only Ethernet Adapter
     24...34 e1 2d ca a7 61 ......Microsoft Wi-Fi Direct Virtual Adapter
     21...36 e1 2d ca a7 60 ......Microsoft Wi-Fi Direct Virtual Adapter #2
      7...34 e1 2d ca a7 60 ......Intel(R) Wireless-AC 9560
      1...........................Software Loopback Interface 1
    ===========================================================================

    IPv4 Table de routage
    ===========================================================================
    Itinéraires actifs :
    Destination réseau    Masque réseau  Adr. passerelle   Adr. interface Métrique
              0.0.0.0          0.0.0.0      192.168.0.1     192.168.0.23     50
            127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
            127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
      127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
          169.254.0.0      255.255.0.0         On-link    169.254.70.210    276
       169.254.70.210  255.255.255.255         On-link    169.254.70.210    276
      169.254.255.255  255.255.255.255         On-link    169.254.70.210    276
          192.168.0.0    255.255.255.0         On-link      192.168.0.23    306
         192.168.0.23  255.255.255.255         On-link      192.168.0.23    306
        192.168.0.255  255.255.255.255         On-link      192.168.0.23    306
        192.168.127.0    255.255.255.0         On-link     192.168.127.1    281
        192.168.127.1  255.255.255.255         On-link     192.168.127.1    281
    192.168.127.255  255.255.255.255         On-link     192.168.127.1    281
            224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
            224.0.0.0        240.0.0.0         On-link     192.168.127.1    281
            224.0.0.0        240.0.0.0         On-link      192.168.0.23    306
            224.0.0.0        240.0.0.0         On-link    169.254.70.210    276
      255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      255.255.255.255  255.255.255.255         On-link     192.168.127.1    281
      255.255.255.255  255.255.255.255         On-link      192.168.0.23    306
      255.255.255.255  255.255.255.255         On-link    169.254.70.210    276
    ===========================================================================
    Itinéraires persistants :
      Aucun
 ```
 
Puis celle de la VM

```bash
    [root@localhost hugo]# ip route
    default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 101
    192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 100
```

 Nous communiquons grâce à celle-ci:
 
 Pour l'hôte
 
    192.168.127.255  255.255.255.255         On-link     192.168.127.1    281

Pour la VM

    192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 100

Nous allons maintenant télécharger un fichier avec la commande wget qu'il faut installer dans un premier temps

```bash
    yum install wget
```

Pour ensuite utiliser la commande 

```bash
    [root@localhost hugo]# wget https://github.com/Genisys33/Projet-Pong/blob/master/Ponghugo.py
    --2019-01-13 20:29:07--  https://github.com/Genisys33/Projet-Pong/blob/master/Ponghugo.py
    Resolving to github.com (github.com)... 140.82.118.4, 140.82.118.3,
    Connecting to github.com (github.com)|140.82.118.4|:443...connecté.
    HTTP request, awaiting response... 200 OK
    Lenth: unspecified [text/html]
    Saving to : 'Ponghugo.py'
    
    2019-01-13 20:29:07 (1.73 MB/s) - 'Ponghugo.py' saved [85232]
 ```
 
Ensuite nous faisons un petit `ls` dans le dossier pour lister les fichiers présent et nous obtenons

```bash
     [root@localhost hugo]# ls
     Ponghugo.py
```

Le fichier est donc bien téléchargé.

Maintenant nous utilisons la commande `dig` pour les dns de google.com et ynov.com

```bash
    [root@localhost hugo]# dig ynov.com && dig google.com
    ; <<>> DiG 9.9.4-RedHat-9.9.4-72.el7 <<>> google.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21828
    ;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
    
    ;;OPTPSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 4000
    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             0       IN      A       172.217.22.142

    ;; Query time: 37 msec
    ;; SERVER: 89.2.0.1#53(89.2.0.1)
    ;; WHEN: Sun Janv. 13 20:40:07 CET 2019
    ;; MSG SIZE  rcvd: 55

    ; <<>> DiG 9.9.4-RedHat-9.9.4-72.el7 <<>> ynov.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8283
    ;; flags: qr rd ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

    ;;OPTPSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 4000
    ;; QUESTION SECTION:
    ;ynov.com.                      IN      A

    ;; ANSWER SECTION:
    ynov.com.               0       IN      A       217.70.184.38

    ;; Query time: 42 msec
    ;; SERVER: 89.2.0.1#53(89.2.0.1)
    ;; WHEN: Sun Janv. 13 20:42:53 CET 2019
    ;; MSG SIZE  rcvd: 53
```

# II. Notion de ports et SSH

### 1. Exploration des ports locaux

Ici nous allons lister les port TCP que la VM écoute avec la commande `ss -t -a` le `-t` spécifie que l'on cible le protocole de transport fiable (TCP)  et le `-a` est l'abréviation de `-all`

```bash
    [hugo@localhost hugo]# ss -t -a
    State      Recv-Q Send-Q                                     Local Address:Port                                                      Peer Address:Port
    LISTEN     0      128                                                    *:ssh                                                                  *:*
    LISTEN     0      100                                            127.0.0.1:smtp                                                                 *:*
    ESTAB      0      64                                        192.168.127.10:ssh                                                      192.168.127.1:62657
    LISTEN     0      128                                                   :::ssh                                                                 :::*
    LISTEN     0      100                                                  ::1:smtp                                                                :::*
```
  
Pour détecter l'application qui écoute le port 22 nous devons utiliser la commande `-p` en même temps nous allons lister tout les port tcp en remplaçant le nom par le port tutilisé avec la commande `-n`
  
```bash
    [root@localhost hugo]# ss -4 -n -t -p -l
    State      Recv-Q Send-Q                                       Local Address:Port                                                      Peer Address:Port
    LISTEN     0      128                                                      *:22                                                                   *:*                   
    users:(("sshd",pid=3189,fd=3))
    LISTEN     0      100                                              127.0.0.1:25                                                                   *:*
```

Nous remarquons alors qu'il y a bien effectivement une application qui écoute sur le port 22 et c'est celle-ci

```bash
    LISTEN     0      128                                                      *:22                                                                   *:*                   
    users:(("sshd",pid=3189,fd=3))
 ```
 
 ### 2. SSH

Comme demmandé dans l'annexe 1 nous avons éffectué les commandes indiqué pour désactiver SELinux, 

```bash
    [root@localhost hugo]# setenforce 0
    [root@localhost hugo]# nano /etc/selinux/config
```    

Une fois nano lancé il faut modifier la line `SELINUX=unforcing` et la remplacer par `SELINUX=permissive` ce qui nous donne :

    # This file controls the state of SELinux on the system.
    # SELINUX= can take one of these three values:
    #     enforcing - SELinux security policy is enforced.
    #     permissive - SELinux prints warnings instead of enforcing.
    #     disabled - No SELinux policy is loaded.
    SELINUX=permissive
    # SELINUXTYPE= can take one of three values:
    #     targeted - Targeted processes are protected,
    #     minimum - Modification of targeted policy. Only selected processes are protected.
    #     mls - Multi Level Security protection.
    SELINUXTYPE=targeted

```bash
    [root@localhost hugo]# sestatus
    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   permissive
    Mode from config file:          permissive
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Max kernel policy version:      31
```

Maintenant nous allons configurer putty pour configurer un serveur ssh qui nous aurait bien était utile dans la partie I m'évitant de recopier ligne par ligne le retour cli mais ça mb ;P.

![puty](https://github.com/Genisys33/CCNA/blob/master/Image-Tp2/puttyconf.png)

Pour le configurer il nous suffit juste de mêttre l'adresse ip de notre Vm de d'allumer la Vm (Soyons pas bête) et de nous connecter avec nos identifiant de la Vm.


### 3. Firewall

***A. à faire :***

Nous allons modifier en premier lieu comme demander le numéro du port sur lequel le serveur ssh écoute grâce à la commande `nano /etc/ssh/sshd_config`qui était de 22 à un numéro aléatoire tant qu'il est supérieur à 1024.Ici nous l'avons mis à 6666 (Ne pas oublier d'enlever le # devant le Port).

    # If you want to change the port on a SELinux system, you have to tell
    # SELinux about this change.
    # semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
    #
    Port 6666
    #AddressFamily any
    #ListenAddress 0.0.0.0
    #ListenAddress ::

Ensuite nous redémarons le serveur ssh pour rendre les changement actif grâce à la commande `systemctl restart ssh`. Ensuite nous regardons si les changements ont était pris en compte :

```bash
    [root@localhost hugo]# ss -4 -n -t -l -p
    State      Recv-Q Send-Q Local Address:Port               Peer Address:Port                                                                                             
    LISTEN     0      128          *:6666                     *:*                                                                                                           users:(("sshd",pid=3634,fd=3))
```

Lorsque nous configurons Putty avec le port 6666. Nous ne pouvons pas nous connecter. 

![putty6666](https://github.com/Genisys33/CCNA/blob/master/Image-Tp2/putty-6666.PNG)

Nous obtenons ce résultat :

![puttyfail](https://github.com/Genisys33/CCNA/blob/master/Image-Tp2/PuttyFail.PNG)

Là actuellement le port désiré qui est le `6666` n'est pas ouvert, c'est donc normal que l'opération ne marche pas, pour ceci nous devons utiliser les commandes suivante. Nous allons aussi utiliser la commande `--reload` pour rendre active les modifications.

```bash
    [root@localhost hugo]# firewall-cmd --add-port=6666/tcp --permanent
    success
    [root@localhost hugo]# firewall-cmd --reload
    success
```

***B. à faire :***

Juste ici nous lançons le serveur `netcat`sur le port `5454` :

```bash
    [root@localhost hugo]# nc -l 5454
    azertyuio
    azertyuio
```

Ensuite nous lançons un terminal sur le pc hôte nous nous connectons au serveur, une fois connecté les deux machines pourront communiquer.

```cmd
    C:\Users\hugo\Documents\Programmation\Ynov\netcat-1.11>nc 192.168.127.10 5454
    azertyuio
    azertyuio
```

Ensuite nous allons vérifier si la connexion `netcat` se déroule bien avec la commande `ss`(Commande très importante à ne pas oublier)

```bash
    [root@sam hugo]# ss -ntp
    State      Recv-Q Send-Q Local Address:Port               Peer Address:Port       
    ESTAB      0      0      192.168.127.10:5454               192.168.127.1:63306               users:(("nc",pid=3909,fd=5))
    ESTAB      0      0      192.168.127.10:22                 192.168.127.1:62560               users:(("sshd",pid=3734,fd=3),("sshd",pid=3730,fd=3))
    ESTAB      0      64     192.168.127.10:22                 192.168.127.1:62913               users:(("sshd",pid=3843,fd=3),("sshd",pid=3839,fd=3))
```

Voilà nos deux machines communiquent parfaitement via `netcat` sur le port 5454.



# III. Routage Statique

### 1. Préparation des hôtes (vos PCs)

Après avoir désactivé SeLinux comme vu [ici](https://github.com/It4lik/B1-Reseau-2018/tree/master/tp/3#annexe-1--d%C3%A9sactiver-selinux) nous pouvons commencer cette troisième partie. Pour ceci il nous faut modifier toute les addresse ip de nos machine histoire d'éviter les conflits ou les messages d'erreurs. Voici un petit tableau récapitulatif des modifications à apporter et histoire de ne pas s'embrouiller ainsi que désactiver les carte `enp0s3` sur nos vm pour utiliser exclusivement le port internet.

```bash
    [root@localhost hugo]# ifdown enp0s3
    Device 'enp0s3' successfully disconnected.
```

<table>
  <thead>
    <tr>
      <th>Network 1</th>
      <th></th>
      <th>Network 2</th>
      <th></th>
      <th>Network 12</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Windows Pc 1</td>
      <td>192.168.101.1</td>
      <td>Windows Pc 2</td>
      <td>192.168.102.2</td>
      <td>Windows Pc 1</td>
      <td>192.168.112.1</td>
    </tr>
    <tr>
      <td>Vm CentOs 1</td>
      <td>192.168.101.10</td>
      <td>Vm CentOs 2</td>
      <td>192.168.102.10</td>
      <td>Windows Pc 2</td>
      <td>192.168.112.2</td>
    </tr>
  </tbody>
</table>

Une fois le tout fait nous devons nous assurer que tout fonctionne, nous allons donc ping des deux côtés la vm au pc hôte et les deux pc hôte ensemble cela donne donc : 

* PC1 et PC2 se ping
* VM1 et PC1 se ping
* VM2 et PC2 se ping

Nous faisons un Ping de Windows 2 à Windows 1 :

```cmd
    C:\Users\hugo>ping 162.168.112.1
    Envoi d’une requête 'Ping'  162.168.112.1 avec 32 octets de données :
    Délai d’attente de la demande dépassé.
    Délai d’attente de la demande dépassé.
    Délai d’attente de la demande dépassé.
    Délai d’attente de la demande dépassé.

    Statistiques Ping pour 162.168.112.1:
        Paquets : envoyés = 4, reçus = 0, perdus = 4 (perte 100%)
 ```

Nous faisons un Ping de la Vm 2 à Windows 2 :

```bash
    [root@localhost hugo]# ping 192.168.102.2
    PING 192.168.102.2 (192.168.102.2) 56(84) bytes of data.
    64 bytes from 192.168.102.2: icmp_seq=1 ttl=128 time=0.210 ms
    64 bytes from 192.168.102.2: icmp_seq=2 ttl=128 time=0.625 ms
    64 bytes from 192.168.102.2: icmp_seq=3 ttl=128 time=0.607 ms
    64 bytes from 192.168.102.2: icmp_seq=4 ttl=128 time=0.673 ms
    64 bytes from 192.168.102.2: icmp_seq=5 ttl=128 time=0.718 ms
    64 bytes from 192.168.102.2: icmp_seq=6 ttl=128 time=0.651 ms
    64 bytes from 192.168.102.2: icmp_seq=7 ttl=128 time=0.672 ms
    ^C
    --- 192.168.102.2 ping statistics ---
    7 packets transmitted, 7 received, 0% packet loss, time 6003ms
    rtt min/avg/max/mdev = 0.210/0.593/0.718/0.162 ms
 ```
 
 Nous faisons un Ping de la Vm 1 à Windows 1 :
 
 ```bash
    [root@localhost ~]# ping 192.168.101.1
    PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
    64 bytes from 192.168.101.1: icmp_seq=1 ttl=128 time=0.210 ms
    64 bytes from 192.168.101.1: icmp_seq=2 ttl=128 time=0.625 ms
    64 bytes from 192.168.101.1: icmp_seq=3 ttl=128 time=0.607 ms
    64 bytes from 192.168.101.1: icmp_seq=4 ttl=128 time=0.673 ms
    64 bytes from 192.168.101.1: icmp_seq=5 ttl=128 time=0.718 ms
    64 bytes from 192.168.101.1: icmp_seq=6 ttl=128 time=0.651 ms
    64 bytes from 192.168.101.1: icmp_seq=7 ttl=128 time=0.672 ms
    ^C
    --- 192.168.101.1 ping statistics ---
    7 packets transmitted, 7 received, 0% packet loss, time 6003ms
    rtt min/avg/max/mdev = 0.210/0.593/0.718/0.162 ms
 ```
 Ensuite nous n'avons plus qu'à activer les routages sur les pc grâce aux explications donné et nous pouvon sdonc passer à la suite.
 
 ### 2. Configuration du routage

Nous allons commencer par ajouter la route du PC1 vers le PC2 :

```cmd
    C:\Windows\system32>route add 192.168.102.0/24 mask 255.255.255.0 192.168.112.2
    OK!
```

Ensuite le PC2 ajoute la route vers le PC1 :

```cmd
    C:\WINDOWS\system32> route add 192.168.101.0/24 mask 255.255.255.0 192.168.112.1
    OK!
```

Maintenant au tour des Vm d'ajouter les routes :

La VM1 qui ajoute les routes vers la VM2 et le PC2 :

```bash
    [root@centos ~]# ip route add 192.168.102.0/24 via 192.168.101.1 dev eth1
    [root@centos ~]# ip route add 192.168.112.0/24 via 192.168.101.1 dev eth1
```

Puis la VM2 qui ajoute les routes vers VM1 et le PC2 :

```bash
    [root@localhost hugo]# ip route add 192.168.101.0/24 via 192.168.102.1 dev enp0s8
    [root@localhost hugo]# ip route add 192.168.112.0/24 via 192.168.102.1 dev enp0s8
```

Maintenant nous allons ping tout ce beau monde

Le PC1 via le réseau 2 au PC2 :

```cmd
    C:\WINDOWS\system32> ping 192.168.102.1

    Envoi d’une requête 'Ping'  192.168.102.1 avec 32 octets de données :
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54

    Statistiques Ping pour 192.168.102.1:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms
```

Le PC2 via le réseau 1 au PC1 :

```cmd
    C:\Windows\system32>ping 192.168.101.1

    Envoi d’une requête 'Ping'  192.168.101.10 avec 32 octets de données :
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63

    Statistiques Ping pour 192.168.101.10:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms
```

La VM1 via le réseau 2 et 12 vers le PC2 :

```bash
    [root@centos ~]# ping 192.168.102.1
    PING 192.168.102.1 (192.168.102.1) 56(84) bytes of data.
    64 bytes from 192.168.102.1: icmp_seq=1 ttl=126 time=0.977 ms
    64 bytes from 192.168.102.1: icmp_seq=2 ttl=126 time=1.32 ms
    64 bytes from 192.168.102.1: icmp_seq=3 ttl=126 time=1.02 ms
    64 bytes from 192.168.102.1: icmp_seq=4 ttl=126 time=1.51 ms
    ^C
    
    [root@centos ~]# ping 192.168.112.2
    PING 192.168.112.2 (192.168.112.2) 56(84) bytes of data.
    64 bytes from 192.168.112.2: icmp_seq=1 ttl=127 time=1.27 ms
    64 bytes from 192.168.112.2: icmp_seq=2 ttl=127 time=1.47 ms
    64 bytes from 192.168.112.2: icmp_seq=3 ttl=127 time=1.52 ms
    64 bytes from 192.168.112.2: icmp_seq=4 ttl=127 time=1.60 ms
    ^C
```

Et pour terminer la VM2 via le réseau 12 et 1 vers le PC1 :

```bash
    [root@localhost hugo]# ping 192.168.112.1
    PING 192.168.112.1 (192.168.112.1) 56(84) bytes of data.
    64 bytes from 192.168.112.1: icmp_seq=1 ttl=127 time=1.39 ms
    64 bytes from 192.168.112.1: icmp_seq=2 ttl=127 time=1.82 ms
    64 bytes from 192.168.112.1: icmp_seq=3 ttl=127 time=1.86 ms
    64 bytes from 192.168.112.1: icmp_seq=4 ttl=127 time=1.13 ms
    
    [root@localhost hugo]# ping 192.168.101.1
    PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
    64 bytes from 192.168.101.1: icmp_seq=1 ttl=126 time=1.10 ms
    64 bytes from 192.168.101.1: icmp_seq=2 ttl=126 time=1.19 ms
    64 bytes from 192.168.101.1: icmp_seq=3 ttl=126 time=1.11 ms
    64 bytes from 192.168.101.1: icmp_seq=4 ttl=126 time=1.13 ms
```

### 3. Configuration des noms de domaine

Nous allons rentrer tout les noms de domaines dans chacun des fichiers host sur les Vm :

Sur la VM1 nous ajoutons ces lignes au fichier `/etc/hosts`:

```bash
  192.168.112.1   pc1 pc1.tp3.b1
  192.168.112.2   pc2 pc2.tp3.b1
  192.168.102.10  vm2 vm2.tp3.b1
```

Sur la VM2 nous ajoutons ces lignes dans le fichier `hosts` également :


```bash
  192.168.101.10  vm1 vm1.tp3.b1
  192.168.112.1   pc1 pc1.tp3.b1
  192.168.112.2   pc2 pc2.tp3.b1
```

Maintenant nous allons faire la même chose sur les PCs hôtes :

Pc1 :

```cmd
192.168.102.10 vm2.tp3.b1
192.168.101.10 vm1.tp3.b1
192.168.112.2 pc2.tp3.b1
```

Pc2 :

```cmd
192.168.102.10 vm2.tp3.b1
192.168.101.10 vm1.tp3.b1
192.168.112.1 pc1.tp3.b1
```

Une fois les noms de domaine configuré nous allons faire ping tous les FQDN de la VM1 :

```bash
    [root@centos ~]# ping pc1.tp3.b1 
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=1 ttl=127 time=1.13 ms
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=2 ttl=127 time=0.749 ms
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=3 ttl=127 time=1.05 ms
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=4 ttl=127 time=0.984 ms

    [root@centos ~]ping pc2.tp3.b1
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=1 ttl=127 time=1.74 ms
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=2 ttl=127 time=1.92 ms
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=3 ttl=127 time=1.64 ms
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=4 ttl=127 time=2.01 ms

    [root@centos ~]ping vm2.tp3.b1
    64 bytes from vm2.tp3.b1 (192.168.102.10): icmp_seq=1 ttl=62 time=1.71 ms
    64 bytes from vm2.tp3.b1 (192.168.102.10): icmp_seq=2 ttl=62 time=2.34 ms
    64 bytes from vm2.tp3.b1 (192.168.102.10): icmp_seq=3 ttl=62 time=2.02 ms
    64 bytes from vm2.tp3.b1 (192.168.102.10): icmp_seq=4 ttl=62 time=2.12 ms
```

Et ensuite la même chose mais avec la VM2 :

```bash
    [root@localhost hugo]# ping pc1.tp3.b1
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=1 ttl=127 time=1.17 ms
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=2 ttl=127 time=0.816 ms
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=3 ttl=127 time=1.01 ms
    64 bytes from pc1.tp3.b1 (192.168.112.1): icmp_seq=4 ttl=127 time=0.956 ms
    
    [root@localhost hugo]#ping pc2.tp3.b1
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=1 ttl=127 time=0.224 ms
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=2 ttl=127 time=0.284 ms
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=3 ttl=127 time=0.256 ms
    64 bytes from pc2.tp3.b1 (192.168.112.2): icmp_seq=4 ttl=127 time=0.321 ms
    
    [root@localhost hugo]# ping vm1.tp3.b1
    64 bytes from vm1.tp3.b1 (192.168.101.10): icmp_seq=1 ttl=62 time=1.50 ms
    64 bytes from vm1.tp3.b1 (192.168.101.10): icmp_seq=2 ttl=62 time=1.78 ms
    64 bytes from vm1.tp3.b1 (192.168.101.10): icmp_seq=3 ttl=62 time=1.24 ms
    64 bytes from vm1.tp3.b1 (192.168.101.10): icmp_seq=4 ttl=62 time=1.32 ms
```

Maintenant nous faisons un petit netcat avec les noms de domaine :

Vm 2 :

```bash
    [root@localhost hugo]# nc -l 666
    Le bon netcat qui passe
    Compliqué quand même
    Pas du tout
```

Vm 1 :

```bash
    [root@localhost ~]# nc vm2.tp3.b1 666
    Le bon netcat qui passe
    Compliqué quand même
    Pas du tout
```

Le but est donc atteint.

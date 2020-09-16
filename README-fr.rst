Que se passe-t-il quand…
========================

Ce dépôt est une tentative pour répondre à cette vieille question d'interview
"Que se passe-t-il quand vous écrivez google.com dans la barre d'adresse 
de votre navigateur web et que vous appuyez sur la touche Entrée ?"

Hormis l'habituel histoire, nous allons essayer de répondre à cette question
avec autant de détails que possible. Rien ne sera négligé. 

Ceci est un processus collaboratif, alors svp creusez et essayez d'aider !
Il y a des tonnes de détails manquants qui n'attendent que vous pour les 
ajouter ! SVP, envoyez-nous vos requêtes !

Tout ceci est sous les termes de la licence `Creative Commons Zero`_

Vous pouvez lire ceci en `简体中文`_ (Chinois simplifié), `日本語`_ (Japonais) 
et `한국어`_ (Coréen).  NOTE : ces traductions n'ont pas été examinées par les
responsables de alex/what-happens-when.

Table des Matières
====================

.. contents::
   :backlinks: none
   :local:

La touche "g" est appuyée
-------------------------

La section suivante explique les actions physiques du clavier et les 
interruptions du système d'exploitation. Lorsque vous appuyez sur la touche
"g", le navigateur reçoit l'événement et les fonctions d'autocomplétion
s'enclenchent. 
Selon l'algorithme de votre navigateur et si vous êtes en mode privé, ou 
non, diverses suggestions vous seront présentées dans un menu déroulant
sous la barre d'URL. La plupart des algorithmes trient et priorisent les 
résultats selon la recherche historique, les marques-pages, les cookies, 
et les recherches populaires sur Internet. Lorsque vous écrivez "google.com",
de nombreux blocs de code s'exécutent et les suggestions seront affinées
à chaque appui sur une touche. Il peut même vous suggérer "google.com" 
avant que vous ayez fini de l'écrire. 

L'appui sur la touche "Entrée"
------------------------------

Pour commencer, choisissons l'appui sur le bas de la touche "Entrée". À 
ce moment, un circuit électrique spécifique à la touche est verrouillée 
(soit directement, soit de manière capacitive). Ceci permet à une petite
quantité de courant de circuler dans le circuit logique du clavier, qui 
analyse l'état de l'interrupteur de chaque touche, élimine le bruit électrique 
de la fermeture intermittente rapide de l'interrupteur et le convertit en
un entier de code de touche, en l’occurrence 13.
Le contrôleur du clavier encode alors le code de touche pour le transport
vers l'ordinateur. Actuellement, c'est presque universellement au-travers
d'une connexion USB (Universal Serial Bus) ou par Bluetooth, mais historiquement
c'était des connexions de type PS/2 ou ADB. 

*Dans le cas d'un clavier USB :*

- Le circuit USB du clavier est alimenté par l'alimentation 5V (volt) fournit
  sur la broche 1 du contrôleur USB de l'ordinateur.

- Le code de touche généré est enregistré par la mémoire du circuit interne
  du clavier dans un registre appelé "endpoint". 

- Le contrôleur USB analyse ce registre "endpoint" environ toutes les 10 
  millisecondes (valeur minimum déclarée par le clavier), ainsi il 
  enregistre la valeur du code de touche. 

- Cette valeur parvient au moteur d'interface série USB (Serial Interface Engine)
  afin d'être converti dans un ou plusieurs paquets USB selon le protocole
  USB de bas niveau.

- Ces paquets sont envoyés par un signal électrique différentiel sur les
  connecteurs D+ et D- (entre 2) à la vitesse maximale de 1.5 Mo/s (Méga-octet
  par seconde), puisqu'un dispositif IHM (Interface Homme-Machine) est 
  toujours déclaré en tant que "dispositif à faible vitesse" 
  (en conformité avec la norme USB 2.0).

- Ce signal en série est alors décodé par le contrôleur USB de l'ordinateur, 
  puis interprété par le pilote du dispositif universel de clavier IHM
  de l'ordinateur. La valeur de la touche est passée au-travers de la 
  couche d'abstraction matérielle du système d'exploitation.
  
*Dans le cas d'un clavier virtuel (de même pour les écrans tactiles) :*

- Quand l'utilisateur pose son doigt sur un écran tactile capacitif moderne, 
  une quantité infime de courant est transmise au doigt. Cela complète 
  le circuit par le champ électrostatique de la couche conductrice et 
  crée une chute de tension à cet endroit de l'écran. Le ``contrôleur de l'écran``
  lève une interruption rapportant les coordonnées de la touche pressée. 

- Alors le système d'exploitation mobile notifie à l'application en cours 
  de l’événement de pression d'un des éléments de son interface (qui sont
  les boutons de l'interface virtuelle de l'application de clavier).

- Le clavier virtuel peut maintenant lever une interruption logicielle
  afin d'envoyer un message de 'touche pressée' au système d'exploitation. 

- Cette interruption notifie l'application en cours d'un événement de 'touche
  pressée'. 

Déclenchement d'interruption [Hors claviers USB]
------------------------------------------------

Le clavier envoie des signaux sur sa ligne de requêtes d'interruption (IRQ),
qui correspond à un entier ``interrupt vector`` du contrôleur d'interruption.
Le processeur (UCT : ``Unité Centrale de Traitement``) utilise la table de 
descripteurs d'interruptions (IDT : ``Interrupt Descriptor Table``)
qui correspond aux vecteurs d'interruptions vers les fonctions (``interrupt handlers``)
qui sont fournis par le noyau. Lorsqu'une interruption arrive, l'UCT indexe
l'IDT avec le vecteur d'interruptions et exécute le gestionnaire approprié. 
Ainsi, le noyau est introduit. 

(Dans Windows) Un message ``WM_KEYDOWN`` est envoyé à l'application
-------------------------------------------------------------------

Le transport IHM envoie l'événement de touche pressée au pilote ``KBDHID.sys`` 
qui convertit l'utilisation de l'IHM vers un code d'analyse. Dans ce cas, 
le code d'analyse est ``VK_RETURN`` (``0x0D``). Le pilote ``KBDHID.sys`` 
s'interface avec ``KBDCLASS.sys`` (un pilote de classe clavier). Ce pilote
est responsable de toutes les entrées de clavier et clavier numérique de 
manière sécurisée. Il les appelle ensuite dans ``Win32K.sys`` (après avoir
passer le message dans les filtres de clavier tiers installés). 
C'est tout ce qui se passe dans le noyau. 

``Win32K.sys`` détermine quelle fenêtre est active au-travers de l'API 
``GetForegroundWindow()``. Cette API fournit la capture de la fenêtre à 
la boite d'adresse du navigateur. La fenêtre principale "message pump"  appelle
alors ``SendMessage(hWnd, WM_KEYDOWN, VK_RETURN, lParam)``. ``lParam`` est
un masque binaire qui indique des informations complémentaires à la pression
de touche : compteur de répétition (0 dans ce cas), le code d'analyse actuel
(peut être dépendant du fabriquant, mais ne l'est pas généralement pour ``VK_RETURN``),
quelque soit la touche étendue (e.g. alt, shift, ctrl) qui soit aussi 
appuyée (elles ne l'étaient pas), ou dans un autre état. 

L'API Windows ``SendMessage`` est une fonction simple qui ajoute le message
à une queue d'un gestionnaire de fenêtres particulier (``hWnd``). Plus tard,
la fonction principale de traitement des messages (appelée ``WindowProc``) 
assignée à ``hWnd`` est appelée afin de traiter chaque message dans la queue. 

La fenêtre (``hWnd``) qui est active est en fait un contrôleur d'édition, 
et le ``WindowProc`` dans ce cas est un gestionnaire de messages pour ``WM_KEYDOWN``.
Ce code cherche un paramètre tiers qui est passé à ``SendMessage`` (``wParam``), 
parce que ``VK_RETURN`` sait qu'un utilisateur a appuyé sur la touche Entrée.

(Dans OS X) Un NSEvent ``KeyDown``  est envoyé à l'application
--------------------------------------------------------------

Le signal d'interruption déclenche un événement d'interruption dans le pilote
du Kit d'Entrée/Sortie (I/O) kext du clavier. Le pilote traduit le signal 
dans un code de touche qui est passée au process d'OS X ``WindowServer``.
Pour résultat, le ``WindowServer`` envoie un événement à toute application
appropriée (e.g. active ou écoutant) au-travers du port Mach qui est placé
dans la queue d'événements. Les événements peuvent alors être lus depuis
cette queue par des "fils" (appelés threads) disposant des privilèges 
suffisants appelant la fonction ``mach_ipc_dispatch``. Cela arrive
généralement au-travers d'une boucle de gestion principale ``NSApplication`` 
le gérant, via un événement ``NSEvent`` d'un type d'événement 
``NSEventType`` ``KeyDown``.

(Dans GNU/Linux) Le serveur Xorg écoute les codes de touches
------------------------------------------------------------

Lorsqu'un serveur graphique ``X server`` est utilisé, ``X`` utilisera
le pilote d'événement générique ``evdev`` afin d'acquérir l'événement de 
pression de touche. La conversion des codes de touches en codes d'analyse
est faite à l'aide de règles et de keymaps ("cartes de claviers") spécifiques
à ``X server``. 
Lorsque la correspondance du code d'analyse à une touche pressée est complète, 
le ``X server`` envoie le caractère au gestionnaire de fenêtres ``window manager``
(DWM, metacity, i3, etc), afin que le ``window manager`` à son tour envoie
le caractère à la fenêtre en cours. 
L'API graphique de la fenêtre qui reçoit le caractère affiche le symbole 
de police approprié dans le champ approprié ayant le focus. 

Analyse d'URL
-------------

* Le navigateur a maintenant l'information suivante contenue dans l'URL 
  (Uniform Resource Locator) :

- ``Protocol`` "http" : Utilise 'Hyper Text Transfer Protocol'
- ``Resource``  "/" : Récupère la page principale (index)        

Est-ce une URL ou un terme recherché ?
--------------------------------------

Quand aucun protocole ou nom de domaine valide n'est donné, le navigateur 
s'occupe de récupérer le texte donné dans la boite d'adresse au moteur de
recherche web par défaut du navigateur. 
Dans beaucoup de cas, un texte spécial est ajouté à l'URL pour indiquer 
au moteur de recherche qu'il provient de la barre d'URL d'un navigateur 
particulier.

Convertir les caractères Unicode non-ASCII dans le nom d'hôte
-------------------------------------------------------------

* Le navigateur vérifie tous les caractères du nom d'hôte, qui ne soient pas
  ``a-z``,  ``A-Z``, ``0-9``, ``-``, ou ``.``.
* Puisque le nom d'hôte est ``google.com``, il n'y en aura pas ; mais si 
  c'était le cas, le navigateur appliquerait l'encodage `Punycode`_ à la 
  portion du nom d'hôte dans l'URL.

Vérifier la liste HSTS
----------------------

* Le navigateur vérifie sa liste de "HSTS préchargés (HTTP Strict Transport Security)".
  C'est une liste de sites web qui ont requis de n'être contactés seulement
  que sur HTTPS. 
* Si le site web est dans la liste, le navigateur envoie sa requête via 
  HTTPS plutôt qu'en HTTP. Autrement, la requête initiale est envoyée en HTTP.
  (Notez qu'un site web peut toujours utiliser une politique HSTS *sans*
  être dans la liste HSTS. La première requête HTTP au site web faite par
  un utilisateur recevra une réponse demandant que l'utilisateur envoie 
  seulement des requêtes HTTPS. Toutefois, cette unique requête HTTP 
  pourrait potentiellement laisser l'utilisateur vulnérable à une attaque dite 
  `downgrade attack`_ ; c'est la raison pour laquelle la liste HSTS est
  incluse dans les navigateurs web modernes).

Recherche DNS
-------------

* Le navigateur vérifie si le domaine est dans son cache. (Pour voir le 
  cache DNS dans Chrome, écrivez dans la barre d'adresse 
  ``chrome://net-internals/#dns``).
* S'il n'est pas trouvé, le navigateur appelle la fonction de bibliothèque
  ``gethostbyname`` (qui varie selon l'OS) afin de faire la recherche.
* ``gethostbyname`` vérifie si le nom d'hôte peut être résolu par référence
  dans le fichier local ``hosts`` (dont la localisation `varie selon l'OS`_)
  avant d'essayer de résoudre le nom d'hôte au-travers DNS.
* Si ``gethostbyname`` ne le trouve pas dans le cache, ni dans le fichier ``hosts``
  alors elle fait une requête vers le serveur DNS configuré dans la pile
  réseau. C'est typiquement le routeur local ou le serveur DNS cache du
  FAI (Fournisseur Accès Internet). 
* Si le serveur DNS est sur le même sous-réseau, la bibliothèque réseau 
  suit le ``processus ARP`` décrit ci-dessous pour le serveur DNS. 
* Si le serveur DNS est sur un sous-réseau différent, la bibliothèque réseau
  suit le ``processus ARP`` décrit ci-dessous pour l'adresse IP de la 
  passerelle par défaut. 

Processus ARP
-------------

Avant d'envoyer une diffusion ARP (Address Resolution Protocol), la bibliothèque
de la pile réseau a besoin de l'adresse IP cible à rechercher. Elle doit 
aussi connaître l'adresse MAC de l'interface qu'elle utilisera pour envoyer
la diffusion ARP. 

Le cache ARP est vérifié en premier pour trouver une entrée ARP de notre 
IP cible. Si elle est dans le cache, la bibliothèque retourne le résultat : 
IP cible = MAC. 

Si l'entrée n'est pas dans le cache ARP : 

* La table de routage est recherchée, pour voir si l'adresse IP ciblée est
  dans le sous-réseau de la table de routage local. Si elle y est, la 
  bibliothèque utilise l'interface associée au sous-réseau. Si elle n'y
  est pas, la bibliothèque utilise l'interface qui est dans le sous-réseau
  de notre passerelle par défaut. 

* L'adresse MAC de l'interface réseau sélectionnée est recherchée.

* La bibliothèque réseau envoie une requête ARP de la Couche 2 (trame de
  liaison d'adressage physique du `modèle OSI`_) : 

``Requête ARP``::

    Émetteur MAC: interface:mac:address:here
    Émetteur IP: interface.ip.goes.here
    Cible MAC: FF:FF:FF:FF:FF:FF (Broadcast)
    Cible IP: target.ip.goes.here

Cela dépend du type de matériel qui est entre l'ordinateur et le routeur :

Directement connecté : 

* Si l'ordinateur est directement connecté au routeur, le routeur répond
  avec une ``Réponse ARP`` (lire ci-dessous)

Hub : 

* Si l'ordinateur est connecté à un hub, le hub diffusera la requête ARP
  vers tous les autres ports. Si le routeur est connecté sur la même "interface",
  il répondra avec une ``Réponse ARP`` (lire ci-dessous).

Commutateur : 

* Si l'ordinateur est connecté à un commutateur, le commutateur vérifiera
  sa table MAC pour savoir sur quel port est diffusé l'adresse MAC recherchée. 
  Si le commutateur n'a pas d'entrée pour l'adresse MAC, il rediffusera
  la requête ARP vers tous les autres ports. 
  
* Si le commutateur a une entrée dans la table MAC, il enverra une requête
  ARP au port correspondant à l'adresse MAC recherchée. 

* Si le routeur est sur la même "interface", il répondra avec une "Réponse ARP``
  (lire ci-dessous)

``Réponse ARP``::

    Émetteur MAC: target:mac:address:here
    Émetteur IP: target.ip.goes.here
    Cible MAC: interface:mac:address:here
    Cible IP: interface.ip.goes.here

----

Le protocole ARP est nécessaire au fonctionnement d’`IPv4`_, utilisé par dessus
un réseau de type `Ethernet`_. En `IPv6`_, les fonctions ARP ont été reprises dans
le processus de découverte `NDP`_.

----

Maintenant que la bibliothèque réseau a l'adresse IP, soit de notre serveur
DNS, soit de la passerelle par défaut, elle peut reprendre son processus DNS :

* Le client DNS établit un socket vers le port UDP 53 du serveur DNS, utilisant
  un port source au-delà de 1023.
* Si la taille de la réponse est trop grande, TCP sera utilisé à la place. 
* Si le serveur DNS local ou du FAI ne l'a pas, alors une recherche récursive
  est requise et fait remonter la liste des serveurs DNS jusqu'à ce que 
  l'enregistrement SOA (Start Of Authority) soit atteint, et qu'une réponse
  soit retournée. 

Ouverture d'une socket
----------------------

Une fois que le navigateur reçoit l'adresse IP du serveur de destination, 
il la prend ainsi que le numéro de port donné dans l'URL (par défaut, le
protocole HTTP a le port 80, et HTTPS le port 443), puis fait un appel à 
la fonction de la bibliothèque système nommée ``socket`` et requiert un 
flux de socket TCP - ``AF_INET/AF_INET6`` et ``SOCK_STREAM``.

* Cette requête est en premier passé à la Couche de Transport où un segment
  TCP est créé. Le port de destination est ajouté à l'entête, et le port 
  source est choisi parmi une plage de port dynamique du noyau 
  (ip_local_port_range dans Linux).
* Ce segment est envoyé vers la Couche Réseau, qui enveloppe une entête IP
  additionnelle. L'adresse IP du serveur cible aussi bien que celle de 
  la machine courante est insérée pour former un paquet. 
* Le paquet suivant arrive sur la Couche de Liaison. Une entête de trame 
  est ajouté qui inclut l'adresse MAC de l'interface réseau de la machine
  ainsi que l'adresse MAC de la passerelle (le routeur local). Tout comme
  avant, si le noyau ne connaît pas l'adresse MAC de la passerelle, il 
  doit diffuser une requête ARP pour la trouver. 

À partir de ce point, le paquet est prêt à être transmis, soit au-travers :

* `Ethernet`_
* `WiFi`_
* `Réseau de Téléphonie Mobile`_

Pour la plupart des connexions à Internet depuis une maison, ou pour de 
petites entreprises, le paquet passera de votre ordinateur, possiblement
au-travers du réseau local, puis vers un modem (MOdulateur/DEModulateur)
qui convertit les 0 et 1 numériques en signal analogique adapté à la 
transmission par téléphone, câble ou connexions de téléphonie sans fil.
À l'autre extrémité de la connexion se trouve un autre modem qui reconvertit 
le signal analogique en données numériques qui seront traitées par le 
prochain `nœud de réseau`_ où les adresses de départ et d'arrivée seront 
analysées plus en détail.

La plupart des grandes entreprises et certaines connexions résidentielles 
plus récentes disposeront de connexions en fibre optique ou de connexions 
Ethernet directes, auxquels cas les données restent numériques et sont 
transmises directement au prochain `nœud de réseau`_ pour y être traitées.

Éventuellement, le paquet atteindra le routeur gérant le sous-réseau local. 
Depuis là, il continuera à voyager vers le système autonome (AS) au-delà 
du routeur, vers d'autres AS, et finalement atteindra le serveur de destination. 
Chaque routeur, le long du chemin, extrait l'adresse de destination de 
l'entête d'IP et la dirige vers le prochain saut approprié. Le champ TTL 
(Time to Live) dans l'entête de l'IP est décrémenté de un à chaque routeur
traversé. Le paquet sera supprimé si le champ TTL atteint zéro ou si le 
routeur en cours n'a plus d'espace dans sa queue (cela peut être dû à une
congestion du réseau). 

Cet envoi et cette réception arrive de nombreuses fois suivant le flux de
connexion TCP : 

* Le client choisit un numéro de séquence initial (ISN : Initial Sequence Number)
  et envoie le paquet au serveur avec le bit SYN paramétré pour indiquer
  qu'il active l'ISN. 
* Le serveur reçoit le bit SYN et s'il est "d'humeur agréable" : 
   * le serveur choisit son propre numéro de séquence initial
   * le serveur paramètre le bit SYN afin d'indiquer qu'il a choisit son ISN
   * le serveur copie l'ISN du client +1 dans son champ ACK et ajoute le 
     drapeau ACK afin d'indiquer qu'il accuse réception du premier paquet. 
* Le client reconnaît la connexion en envoyant un paquet : 
   * augmentant son propre numéro de séquence
   * augmentant le numéro d'accusé de réception
   * paramètre le champ ACK
* La donnée est transmise ainsi : 
   * Lorsqu'une partie envoie N octets de données, elle augmente sa séquence
     SEQ par un numéro
   * Quand l'autre partie accuse réception du paquet (ou d'une chaîne de 
     paquets), elle envoie un paquet ACK avec une valeur ACK égale à la
     dernière séquence reçue depuis l'autre partie. 
* Pour fermer la connexion : 
   * la partie qui termine la connexion envoie un paquet FIN.
   * l'autre partie accuse réception ACK du paquet FIN et envoie son propre
     paquet FIN. 
   * la première partie accuse réception ACK du paquet FIN de l'autre partie. 

La Poignée de Main TLS
----------------------

* L'ordinateur client envoie un message ``ClientHello`` au serveur avec sa
  version de TLS (Transport Layer Security), une liste d'algorithmes de 
  chiffrement et de méthodes de compression disponibles. 

* Le serveur répond avec un message ``ServerHello`` au client avec la version
  TLS, le chiffrement choisi, les méthodes de compression sélectionnées et
  le certificat public signé par une AC (Autorité de Certification) du serveur.
  Le certificat contient une clé publique qui sera utilisée par le client 
  pour chiffrer le reste de la poignée de main jusqu'à ce qu'une clé symétrique 
  puisse être convenue.

* Le client vérifie que le certificat numérique du serveur soit dans sa 
  liste d'AC de confiance. Si la confiance peut être établie, basée sur 
  l'AC, le client génère une chaîne d'octets pseudo-aléatoires et la 
  chiffre avec la clé publique du serveur. Ces octets aléatoires peuvent 
  être utilisés pour déterminer la clé symétrique. 

* Le serveur déchiffre les octets aléatoires en utilisant sa clé privée 
  puis utilise ces octets pour générer sa propre copie de la clé symétrique
  maître. 

* Le client envoie un message ``Finished`` au serveur, chiffrant un hash 
  de la transmission jusqu'à ce point avec la clé symétrique. 

* Le serveur génère son propre hash, puis déchiffre le hash envoyé par le
  client pour vérifier la correspondance. Si elle existe, il envoie son 
  propre message ``Finished`` au client, le chiffrant aussi avec sa clé
  symétrique. 

* À partir de maintenant la session TLS transmet les données de l'application
  (HTTP) chiffrées avec la clé symétrique agréée. 

Protocole HTTP
--------------

Si le navigateur web utilisé été écrit par Google, au lieu d'envoyer une
requête HTTP pour récupérer la page, il enverra une requête pour négocier
avec le serveur une "mise à jour" du protocole HTTP vers le protocole SPDY.

Si le client utilise le protocole HTTP mais ne prend pas en charge SPDY, 
il envoie une requête au serveur de la forme::

    GET / HTTP/1.1
    Host: google.com
    Connection: close
    [autres entêtes]

où ``[autres entêtes]`` référent à une série de paire de clé et valeur séparée
par le symbole deux points ':', formatées selon la spécification HTTP et 
séparées par d'uniques nouvelles lignes.
(Cela suppose que le navigateur web utilisé n'ait pas de bogues violant la
spécification HTTP. Cela suppose aussi que le navigateur web utilise ``HTTP/1.1``,
autrement il ne pourrait pas inclure l'entête ``Host`` dans la requête ;
la version spécifiée dans la requête ``GET`` serait soit ``HTTP/1.0`` ou ``HTTP/0.9``.)

HTTP/1.1 définit l'option de "fermeture" de la connexion pour que l'expéditeur 
signale que la connexion sera fermée après l'achèvement de la réponse. 
Par exemple : 

    Connection: close

Les applications HTTP/1.1 qui ne prennent pas en charge les connexions 
persistantes DOIVENT inclure l'option de "fermeture" de connexion dans 
chaque message. 

Après l'envoi de la requête et des entêtes, le navigateur web envoie une 
unique nouvelle ligne vierge pour indiquer au serveur que le contenu de 
la requête est fait. 

Le serveur répond avec un code de réponse dénotant le statut de la requête
et avec une réponse de la forme::

    200 OK
    [entêtes de réponse]

Suivies d'une unique nouvelle ligne, il envoie alors la charge du contenu 
HTML de ``www.google.com``. Le serveur peut alors soit fermer la connexion, 
soit si les entêtes envoyées par le client le demande, garder la connexion
ouverte afin d'être réutilisées pour de prochaines requêtes. 

Si les entêtes HTTP envoyées par le navigateur web comportent des informations
suffisantes pour que le serveur web détermine si la version du fichier en 
cache dans le navigateur web n'a pas été modifié depuis la dernière récupération 
(tel que si le navigateur web inclut une entête ``ETag``), il peut alors 
répondre par une requête de la forme::

    304 Not Modified
    [entêtes de réponse]

il n'y aura pas charge utile, et le navigateur web récupérera le HTML depuis
son cache. 

Après l'analyse du HTML, le navigateur web (ainsi que le serveur) répétera 
ce processus pour chaque ressource (image, CSS, favicon.ico, etc) référencée
dans la page HTML, excepté que la requête sera ``GET /$(URL relative à www.google.com) HTTP/1.1``
au lieu de ``GET / HTTP/1.1``.

Si le HTML référence une ressource sur un domaine différent que ``www.google.com``, 
le navigateur web reprendra les étapes invoquées pour résoudre l'autre domaine, 
et suivra toutes les mêmes étapes jusqu'à ce point pour ce domaine. 
L'entête ``Host`` dans la requête sera paramétrée vers le nom du serveur 
approprié plutôt que ``google.com``.

Gestionnaire de Requêtes HTTP du Serveur
----------------------------------------

Le serveur HTTPD (Service HTTP) est un gestionnaire de requêtes et de réponses
côté serveur. Les serveurs HTTPD des plus communs sont Apache ou nginx pour
Linux et IIS pour Windows. 

* Le serveur HTTPD (Service HTTP) reçoit la requête. 
* Le serveur décompose la requête selon les paramètres suivants : 
   * la méthode de requête HTTP (soit ``GET``, ``HEAD``, ``POST``, ``PUT``,
     ``PATCH``, ``DELETE``, ``CONNECT``, ``OPTIONS``, ou ``TRACE``). Dans 
     le cas où l'URL est entrée directement dans la barre d'adresse, elle 
     sera ``GET``.
   * le domaine ; dans ce cas : google.com
   * le chemin ou la page demandé ; dans ce cas : / (puisqu'il n'y a pas
     de chemin ou de page spécifique demandé, / est le chemin par défaut).
* Le serveur vérifie qu'un Hôte Virtuel soit configuré sur le serveur correspondant
  à google.com. 
* Le serveur vérifie que google.com peut accepter les requêtes GET. 
* Le serveur vérifie que le client est autorisé à utiliser cette méthode
  (par l'adresse IP, authentification, etc). 
* Si le serveur a un module de ré-écriture installé (tel que mod_rewrite 
  pour Apache ou URL Rewrite pour IIS), il essaiera la correspondance de 
  la requête avec une des règles configurées. Si une règle correspondante
  est trouvée, le serveur utilise la règle pour ré-écrire la requête. 
* Le serveur envoie le contenu qui correspond à la requête, dans notre cas, 
  il reviendra au fichier index, puisque "/" est le fichier principal
  (dans certains cas, cela peut être surchargé, mais c'est la méthode commune).
* Le serveur analyse le fichier en accord avec le gestionnaire. Si Google
  exécute PHP, le serveur utilise PHP pour interpréter le fichier index, 
  et envoie le flux vers le client. 

La scène derrière le Navigateur
-------------------------------

Une fois que le serveur délivre les ressources (HTML, CSS, JS, images, etc)
au navigateur, il est soumis au processus suivant : 

* Analyse HTML, CSS, JS
* Rendu : construit l'arborescence DOM → l'arborescence de rendu → le plan
  de l'arborescence de rendu → l'affichage de l'arborescence de rendu

Le Navigateur
-------------

La fonction du navigateur est de présenter la ressource web que vous avez
choisi, en la demandant à un serveur et en l'affichant dans la fenêtre du
navigateur. La ressource est habituellement un document HTML, mais peut 
être aussi un PDF, une image, ou tout autre type de contenu. L'endroit de 
la ressource est spécifié par l'utilisateur selon une URI (Uniform Resource Identifier).

La manière dont le navigateur interprète et affiche les fichiers HTML est 
spécifiée dans les spécifications HTML et CSS. Ces spécifications sont 
maintenues par le W3C (World Wide Web Consortium), qui est l'organisation
des standards du web. 

Les interfaces utilisateur de navigation ont beaucoup en commun entre elles. 
Parmi les éléments communs de l'interface utilisateur, on peut citer : 

* une barre d'adresse pour l'insertion d'une URI
* des boutons de retour et d'avance
* des options de marque-pages (favoris)
* des boutons pour rafraîchir et stopper le chargement de documents en cours
* un bouton d’accueil pour vous permettre d'aller à votre page d'accueil. 

**Structure de Haut Niveau du Navigateur**

Les composants des navigateurs sont : 

* **Une Interface Utilisateur** : l'interface utilisateur (UI) inclue la barre 
  d'adresse, les boutons retour/avance, le menu des marque-pages, etc. 
  Chaque partie du navigateur s'affiche, exceptée la fenêtre où vous voyez
  la page demandée. 
* **Le Moteur du Navigateur** : le moteur du navigateur répartit les actions
  entre l'UI et le moteur de rendu. 
* **Le Moteur de Rendu** : le moteur de rendu est responsable d'afficher 
  le contenu demandé. Par exemple, si le contenu demandé est du HTML, le 
  moteur de rendu analyse le HTML et le CSS, et affiche le contenu analysé
  à l'écran.
* **Réseau** : le réseau gère les appels réseau tels que les requêtes HTTP, 
  utilisant différentes implémentations pour les différentes plateformes
  derrière une interface de plateforme indépendante.   
* **Backend UI** : le backend de l'UI est utilisé pour dessiner 
  les widgets basiques tels que les comboboxes et les fenêtres. Ce backend
  expose une interface générique qui n'est pas spécifique à une plateforme.
  En profondeur, il utilise les méthodes de l'interface utilisateur du 
  système d'exploitation.
* **Le Moteur JavaScript** : le moteur JavaScript est utilisé pour analyser
  et exécuter le code JavaScript. 
* **Le Stockage des Données** : le stockage des données est une couche persistante. 
  Le navigateur peut sauvegarder toute sorte de données localement, tels
  que des cookies. Les navigateurs prennent en charge aussi des mécanismes
  de stockage tels que localStorage, IndexedDB, WebSQL et FileSystem.

Analyse du HTML
---------------

Le moteur de rendu démarre l'obtention des contenus du document demandé
depuis la couche réseau. Cela se fait habituellement par morceaux de 8 Ko.

Le premier travail de l'analyseur HTML est d'analyser le langage HTML dans
une arborescence.

La sortie de l'arborescence ("l'arborescence analysée") est une arborescence
des éléments du DOM et des nœuds d'attributs. DOM est l'abréviation de 
Document Object Model. C'est la présentation objet du document HTML et 
l'interface des éléments HTML au monde extérieur tel JavaScript. 
La racine de l'arborescence est l'objet "Document". Avant toute manipulation 
par script, le DOM a une relation quasi-univoque avec le balisage.

**Algorithme d'Analyse**

Le HTML ne peut être analysé par des analyseurs habituels.

Les raisons sont : 

* la nature indulgente du langage.
* le fait que les navigateurs ont une tolérance traditionnelle à l'erreur 
  pour prendre en charge les cas connus de HTML invalides.
* le processus d'analyse est ré-entrant. Pour les autres langages, la source
  ne change pas durant l'analyse, mais en HTML, le code dynamique (tels
  que des éléments de scripts contenant des appels à `document.write()`)
  peut ajouter des jetons supplémentaires ; le processus d'analyse en cours
  modifie alors l'entrée. 

Si le navigateur est incapable d'utiliser les techniques d'analyses régulières,
il utilisera un analyseur personnalisé pour l'analyse HTML. L'algorithme 
d'analyse est décrit en détail par la spécification HTML5. 

L'algorithme consiste en deux phases : mise en jeton et construction de 
l'arborescence. 

**Les Actions lorsque l'Analyse est terminée**

Le navigateur commence par récupérer les ressources externes liées à la page
(CSS, images, fichiers JavaScript, etc). 

Lors de cette étape, le navigateur marque le document comme interactif et 
démarre les scripts d'analyse qui sont dans le mode "différé" : tout ce 
qui doit être exécuté après le document est analysé. L'état du document 
est paramétré sur "complet" et un événement "charge" est levé. 

Notez qu'il n'y a jamais d'erreur "Invalid Syntax" sur une page HTML. 
Les navigateurs corrigent tout contenu invalide et l'envoie. 

Interprétation du CSS
---------------------

* Analyse des fichiers CSS, du contenu des balises ``<style>``, et des 
  valeurs des attributs ``style`` en utilisant la `"syntaxe de grammaire et champ lexical CSS"`_
* Chaque fichier CSS est analysé dans un ``StyleSheet object``, où chaque
  objet contient les règles CSS avec les sélecteur et les objets correspondant
  à la grammaire CSS. 
* Un analyseur CSS peut être descendant ou ascendant lorsqu'un générateur 
  d'analyse spécifique est utilisé.

Rendu de Page
-------------

* Crée une "Arborescence d'Image" ou une 'Arborescence de Rendu" en traversant les 
  nœuds du DOM, et en calculant les valeurs du style CSS pour chaque nœud.
* Calcul la largeur préférée de chaque nœud de l'arbre du cadre de bas 
  en haut en additionnant la largeur préférée des nœuds enfants et les 
  marges horizontales, les bordures et le padding du nœud.
* Calcul la largeur actuelle de chaque nœud de haut en bas en allouant à
  chaque nœud disponible la largeur de ses enfants. 
* Calcul la hauteur de chaque nœud de bas en haut en appliquant un habillage 
  de texte et en additionnant les hauteurs des nœuds enfants, les marges, 
  les bordures et le padding du nœud.
* Calcul les coordonnées de chaque nœud en utilisant l'information calculée
  ci-dessus. 
* Des étapes plus compliquées sont menées lorsque les éléments sont positionnés
  en ``floated``, ``absolutely`` ou ``relatively``, ou lorsque d'autres 
  fonctionnalités plus complexes sont utilisées. Pour avoir plus de détails, voir 
  http://dev.w3.org/csswg/css2/ et http://www.w3.org/Style/CSS/current-work
* Crée des calques pour décrire quelles parties de la page peuvent être 
  animées en tant que groupe sans être re-traitées. Chaque objet d'image 
  ou de rendu peut être assigné à un calque. 
* Des textures sont allouées à chaque calque de la page. 
* Les objets d'image ou de rendu pour chaque calque sont parcourus et des
  commandes de dessein sont exécutées pour leur calque respectif. Ils doivent
  être traités par le CPU ou dessinés par le GPU en utilisant directement
  D2D/SkiaGL.
* Toutes les étapes ci-dessus peuvent réutilisées les valeurs calculées
  depuis la dernière fois où la page web a été rendue, ainsi les changements
  incrémentaux demandent moins de travail. 
* Les calques de page sont envoyés au processus de composition où ils sont
  combinés avec les calques d'autres contenus visibles, tel que le chrome
  du navigateur, les iframes, et les panneaux d'extension. 
* Les positions de calque final sont calculés et les commandes de composition
  sont émises via Direct3D/OpenGL. Les tampons de commande du GPU sont vidés
  vers le GPU pour le rendu asynchrone et l'image est envoyée au serveur 
  de fenêtrage. 

Rendu du GPU
------------

* Durant le processus de rendu, les calques de calcul graphique peuvent 
  utilisés aussi bien le ``CPU`` que le processeur graphique ``GPU``.

* Lors de l'utilisation du ``GPU`` pour le calcul du rendu graphique, les
  calques du logiciel graphique découpe la tâche en de multiples pièces, 
  ainsi il peut utiliser avantageusement le parallélisme massif du ``GPU`` 
  pour le calcul de virgule flottante requis pour le processus de rendu. 

Serveur Windows
---------------

Exécution Post-Rendu et induite par l'utilisateur
-------------------------------------------------

Après que le rendu soit complet, le navigateur exécute le code JavaScript
grâce à un mécanisme de temporisation (tel qu'une animation Google Doodle)
ou à une interaction de l'utilisateur (écrivant une requête dans une boîte
de recherche et recevant des suggestions). 
Des plugins tels que Flash ou Java peuvent aussi être exécutés, mais actuellement
pas depuis la page d'accueil de Google. Des scripts peuvent causer des 
requêtes réseaux additionnelles, modifier la page ou sa mise en page, 
entraînant un nouveau cycle de rendu et de dessin de la page.



.. _`Creative Commons Zero`: https://creativecommons.org/publicdomain/zero/1.0/deed.fr
.. _`"syntaxe de grammaire et champ lexical CSS"`: http://www.w3.org/TR/CSS2/grammar.html
.. _`Punycode`: https://fr.wikipedia.org/wiki/Punycode
.. _`Ethernet`: https://fr.wikipedia.org/wiki/IEEE_802.3
.. _`WiFi`: https://fr.wikipedia.org/wiki/IEEE_802.11
.. _`Réseau de Téléphonie Mobile`: https://fr.wikipedia.org/wiki/R%C3%A9seau_de_t%C3%A9l%C3%A9phonie_mobile
.. _`analog-to-digital converter`: https://fr.wikipedia.org/wiki/Convertisseur_analogique-num%C3%A9rique
.. _`nœud de réseau`: https://fr.wikipedia.org/wiki/R%C3%A9seau_informatique
.. _`varie selon l'OS` : https://fr.wikipedia.org/wiki/Hosts#Localisation
.. _`简体中文`: https://github.com/skyline75489/what-happens-when-zh_CN
.. _`한국어`: https://github.com/SantonyChoi/what-happens-when-KR
.. _`日本語`: https://github.com/tettttsuo/what-happens-when-JA
.. _`downgrade attack`: https://fr.wikipedia.org/wiki/Moxie_Marlinspike#HTTPS_stripping
.. _`modèle OSI`: https://fr.wikipedia.org/wiki/Mod%C3%A8le_OSI
.. _`IPv4`: https://fr.wikipedia.org/wiki/Internet_Protocol
.. _`IPv6`: https://fr.wikipedia.org/wiki/IPv6
.. _`NDP`: https://fr.wikipedia.org/wiki/Neighbor_Discovery_Protocol




# TP1
Q1 : Combien d'étudiants, d'EC et de notes contient la base ? Dessinez le schéma des quatre tables avec leurs clés primaires et étrangères.

• EC (Éléments Constitutifs) : 6
• Notes : 180
La somme fait 186.
• Schéma des tables :
  • filiere (code [PK], libelle)
  • etudiant (matricule [PK], prenom, nom, filiere [FK vers filiere.code], email)
  • ec (code [PK], intitule, credits)
  • note (id [PK], matricule [FK vers etudiant.matricule], ec [FK vers ec.code], valeur)


Q2 :
Dans cette session, le client mariadb et le serveur mariadbd tournent sur la même machine. Sont-ils pour autant le même programme ? Montrez-le avec ps -ef | grep -i mariadb.
Non, ce ne sont pas les mêmes programmes. Le client est une interface (le terminal) qui envoie des requêtes, tandis que le serveur est un service en arrière-plan (daemon) qui traite ces requêtes. 
La commande ps -ef montre que le processus /usr/sbin/mariadbd tourne en tant qu'utilisateur mysql (c'est le serveur), tandis que notre session utilisateur (ex: souley) utilise le client pour s'y connecter.



Q3 :
 Que signifie le code d'erreur 10061 de la première tentative ? La machine VM1 était-elle joignable ? Le serveur MariaDB fonctionnait-il ?
L'erreur 10061 (Connection refused) signifie que le client a bien trouvé la machine, mais que le service sur le port 3306 a refusé la connexion. 
La machine VM1 était joignable (le ping fonctionne, comme vu sur vos captures), mais le serveur MariaDB n'écoutait pas sur l'interface réseau externe (il était configuré sur 127.0.0.1 uniquement).



Q4 : Quelle différence y a-t-il entre écouter sur 127.0.0.1 et sur 0.0.0.0 ? Pourquoi la configuration par défaut d'Ubuntu choisit-elle 127.0.0.1 ?

• 127.0.0.1 (localhost) : Le serveur n'accepte que les connexions provenant de la machine locale.
• 0.0.0.0 : Le serveur écoute sur toutes les interfaces réseau disponibles (permet les connexions distantes).
• Ubuntu choisit 127.0.0.1 par défaut pour des raisons de sécurité, afin d'éviter qu'une base de données fraîchement installée ne soit exposée sur le réseau sans configuration préalable.


Q5 :
Relevez le résultat de SELECT USER(), CURRENT_USER(). Expliquez la différence.
• USER() : Affiche l'utilisateur et l'adresse IP depuis laquelle la connexion a été initiée.
• CURRENT_USER() : Affiche le compte utilisé par le serveur pour vérifier les droits (le compte qui a été authentifié).
• Différence : Si un utilisateur se connecte en tant que etudiant@192.168.1.50, le serveur l'authentifie peut-être sous le compte etudiant@192.168.56.%. USER() montre qui se connecte, CURRENT_USER() montre quel profil de droits est appliqué.



Q6 :
 Avec quel compte Windows pourrait-il se connecter en tant que root depuis le réseau ? Pourquoi est-ce une bonne chose ?

Aucun compte, car le compte root de MariaDB n'est configuré que pour localhost. C'est une excellente pratique de sécurité : cela empêche un attaquant de tenter de bruteforcer le mot de passe root depuis le réseau.



Q7 :
 Collez la sortie de PROCESSLIST et de ss. Pour chaque client, relevez l'adresse IP et le port source. Qui a choisi ces ports, et pourquoi sont-ils différents de 3306 ?
• Windows (Client 1) : IP 192.168.252.1 (ou équivalent), port source élevé (63231).
• VM2 (Client 2) : IP 192.168.252.128, port source élevé (55942).
• Le port 3306 est le port d'écoute fixe du serveur. Les ports sources sont choisis dynamiquement par le système d'exploitation du client (ports éphémères) pour assurer le suivi de la connexion TCP.


Q8 :
 Relevez les messages d'erreur obtenus. Où sont enregistrés les droits des comptes, et qui les vérifie ?
• Le compte etudiant reçoit une erreur de type "Access denied" pour l'UPDATE, mais peut faire le SELECT.
• Le compte secretariat peut faire l'UPDATE mais reçoit une erreur pour le DELETE.
• Les droits sont enregistrés dans les tables système de MariaDB (base mysql). C'est le serveur qui vérifie les droits, pas le client. Le client ne fait qu'envoyer la requête.
• Conclusion : La sécurité est centralisée et robuste, un client modifié ne pourra pas contourner les droits.



Q9 :
 Le client Windows voit-il immédiatement la note modifiée depuis VM2 ?

Oui, car les données sont centralisées sur le serveur unique. Dès que VM2 valide son UPDATE, la donnée est modifiée sur le disque du serveur. La requête SELECT suivante de Windows lira cette nouvelle valeur.



Q10 :
 Identifiez dans la sortie de curl -v la requête du client et la réponse du serveur.

• Requête : > GET / HTTP/1.1 (méthode GET, chemin /, protocole HTTP 1.1).
• Réponse : < HTTP/1.1 200 OK (succès) avec les en-têtes Server: Apache... et Content-Type: text/html.
Quelles informations le serveur a-t-il enregistrées dans access.log ?

Le serveur a enregistré : l'adresse IP du client, la date et l'heure de la requête, la méthode HTTP (GET), l'URL demandée (/), le code de statut de la réponse (200), la taille de la réponse et le "User-Agent" (le type de navigateur ou l'outil comme curl).
Q12 : 
VM1 fait tourner deux serveurs. Comment le système sait-il à quel programme remettre un segment TCP ?

Le système d'exploitation se base sur le numéro de port de destination. Un paquet arrivant sur le port 80 est remis à Apache, un paquet sur le port 3306 est remis à MariaDB.



Q13 :
Avec MariaDB, la connexion reste ouverte ; avec HTTP, chaque page est indépendante. Avantage/Inconvénient.

• MariaDB (persistant) : Avantage : Rapide pour des requêtes successives, gestion d'état facile. Inconvénient : Consomme des ressources serveur tant que la session est ouverte.
• HTTP (requête-réponse) : Avantage : Très scalable, le serveur libère les ressources après chaque requête. Inconvénient : Pas d'état par défaut.
• Solution HTTP pour la mémoire : Le serveur utilise des cookies ou des sessions pour identifier un utilisateur connecté entre deux requêtes.



Q14 :
 Quel message obtient chaque client quand le serveur est arrêté ? Comment appelle-t-on ce défaut d'architecture ?

Les clients obtiennent une erreur de type "Connection refused" ou "Lost connection to server". 
Ce défaut est le point unique de défaillance (Single Point of Failure). Si le serveur tombe, tous les clients sont bloqués.



Q15 :
 Le mot de passe apparaît-il en clair dans la capture tcpdump ?
Oui, dans les captures tcpdump -A, les échanges sont en clair. On peut voir le texte de la requête SQL et les données de la réponse. 
Solution : Utiliser le protocole TLS/SSL pour chiffrer les échanges entre le client et le serveur, ou utiliser des tunnels SSH.



Q16.
 Définissez le modèle client-serveur...

Le modèle client-serveur est une architecture logicielle où un programme (le serveur) centralise des ressources et des services, et attend les requêtes d'autres programmes (les clients). Les clients initient la communication, envoient des requêtes et attendent les réponses. Le serveur, de son côté, traite les requêtes, applique les règles de sécurité et renvoie les données.

Avantages :

1. Centralisation et cohérence : Les données et les règles de sécurité sont gérées à un seul endroit, ce qui garantit que tous les clients voient les mêmes informations en temps réel.
2. Sécurité renforcée : Les clients n'ont pas d'accès direct aux données ; ils passent par le serveur qui contrôle les accès (comme vu avec les restrictions d'UPDATE/DELETE).

Inconvénients :

1. Point unique de défaillance : Si le serveur tombe en panne ou subit une attaque, l'ensemble du service est indisponible pour tous les clients.
2. Goulot d'étranglement : Si le nombre de clients augmente fortement, le serveur peut devenir un goulot d'étranglement en termes de performances (CPU, réseau) s'il n'est pas dimensionné correctement.

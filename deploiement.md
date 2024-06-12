## Script de mise en prod 
Pour faire des modifications en toute sureté, il est préferable de faire ses modifications sur un environnement de production qui a les memes caractéristiques que l'environnement de base, nous allons donc rédiger un script de mise en production automatique.

Nous avons dans un premier temps besoin de stocker dans un fichier updatedb.sql les commandes que nous avons a appliquer en production, elles pourront varier et etre changées directement dans ce script.(Dans notre cas ce sera ce que nous avons fait avant dans l'étape 4.Développement)

````sql
RENAME TABLE Visiteur TO Employe;

CREATE TABLE typeEmploye (
    code CHAR(1) PRIMARY KEY,
    nom VARCHAR(255) NOT NULL
);

INSERT INTO typeEmploye (code, nom) VALUES ('C', 'Comptable'), ('V', 'Visiteur');

ALTER TABLE Employe 
    ADD COLUMN TypeEmploye CHAR(1), 
    ADD CONSTRAINT fk_typeEmploye FOREIGN KEY (TypeEmploye) REFERENCES typeEmploye(code);

UPDATE Employe SET TypeEmploye = 'C' WHERE id = 1;
````

Nous rédigeons ensuite le script de mise en production :

``````bash
#!/bin/bash

# Variables
DB_SERVER="atelier2db"
WEB_SERVER="atelier2web"
PROD_DB_SERVER="proddb"
PROD_WEB_SERVER="prodweb"
DEV_DIR="/chemin/vers/le/repertoire/developpement"
PROD_DIR="/chemin/vers/le/repertoire/production"
DB_SCRIPT="updatedb.sql"

# Fonction pour mettre à jour la base de données
update_database() {
    echo "Mise à jour de la base de données..."
    # Exécuter le script de mise à jour de la structure de la base de données
    ssh $DB_SERVER "mysql -u username -ppassword < $DB_SCRIPT"
    echo "La base de données a été mise à jour avec succès."
}

# Fonction pour copier les fichiers vers le serveur web de production
copy_files() {
    echo "Copie des fichiers vers le serveur web de production..."
    # Copier les fichiers nécessaires vers le serveur web de production
    scp -r $DEV_DIR/* $PROD_WEB_SERVER:$PROD_DIR
    echo "Les fichiers ont été copiés avec succès vers le serveur web de production."
}

# Fonction principale
main() {
    # Appeler la fonction de mise a jour de la db
    update_database

    # Appeler la fonction qui copie les fichiers sur le serveur web
    copy_files

    # Indiquer que la mise en prod est terminée
    echo "La mise en production est terminée."
}

# Appeler la fonction principale
main
``````

Pour optimiser et anticiper le fait que les dossiers peuvent différer et l'endroit où nous devons appliquer les changements ne changent, nous allons laisser l'utilisateur mettre les deux chemins relatifs, le script pourra donc être éxécuté de la manière suivante :
``````bash
./miseenprod.sh /chemin/vers/le/repertoire/developpement /chemin/vers/le/repertoire/production
``````

## Collaboration, plan de sauvegarde et de reprise d'activité

a) Pour les fichiers de code source et autres documents partagés, j'utiliserais un système de gestion de versions comme Git. Chaque membre de l'équipe peut cloner le dépôt Git sur sa propre machine virtuelle Multipass pour accéder aux fichiers partagés et collaborer sur le code. Cela confère également un point de sauvegarde supplémentaire

b) Si mes vm tombent en panne, ce n'est pas un problème, il suffit de régulièrement faire des Export/imports de Vm comme multipass le propose, et les stocker sur un drive ainsi qu'un périphérique en physique, dans l'idéal, on le fait avant chaque modification qui peut impacter les VM, si il y a une panne ou un problème, on réimporte la dernière sauvegarde que l'on a fait.

c) Comme dit dans la question ci dessus, grâce aux sauvegardes sur un périphérique externe nous avons normalement toujours la dernière sauvegarde de notre projet entre les mains, si malencontrueusement ce n'est pas le cas, nous avons toujours les fichiers sur le repo git, ou bien si cela ne marche pas non plus, nous avons comme dernière solution le drive, sur lequel nous sommes censés faire des mises à jour à chaque fois que l'on touche à notre vm

# Livrables n°1 : Plan et Diagrammes

Chaque étape a été suivie de manière méthodique pour assurer le bon déroulement du projet.

## Développement

**4. Comment distinguer les visiteurs des comptables ?**

Suggestion : Modifiez la table "Visiteur", renommez-la "Employe", ajoutez un champ "typeEmploye" en char(1) qui fait référence à une table "typeEmploye" (code, nom) avec les valeurs C=Comptable, V=Visiteur.

## Modifications de la base de données

### Connexion à la base de données et commandes pour la navigation

```batch
mysql --host=atelier2db.mshome.net --user=operations --password=operations123;
use gsb;
show databases;
show tables;
```

Pour différencier les visiteurs des comptables, nous attribuons à chaque employé un type ('C' pour comptable et 'V' pour visiteur) dans la colonne "typeEmploye".

### Étapes

- **Renommer la table "Visiteur" en "Employe" :**

  ```sql
  ALTER TABLE visiteur RENAME employe;
  ```

  ![renamevisiteurtoemploye](img/renamevisiteurtoemploye.png)

- **Ajouter une nouvelle colonne "typeEmploye" de type char(1) à la table :**

  ```sql
  ALTER TABLE employe ADD COLUMN typeEmploye char(1);
  ```

  ![ajoutcolumntypeemploye](img/ajoutcolumntypeemploye.png)

- **Créer une nouvelle table "typeEmploye" avec deux colonnes : "code" (char(1)) comme clé primaire et "nom" (CHAR(30)) :**

  ```sql
  CREATE TABLE typeemploye
  (
      code char(1) PRIMARY KEY NOT NULL,
      nom CHAR(30)
  );
  ```

  ![creationtabletypeemploye](img/creationtabletypeemploye.png)

- **Ajouter une contrainte de clé étrangère (foreign key) sur la colonne "typeEmploye" de la table "Employe" qui référence la colonne "code" de la table "typeEmploye" :**

  ```sql
  ALTER TABLE employe ADD CONSTRAINT typeEmploye_fk FOREIGN KEY (typeEmploye) REFERENCES typeEmploye(code);
  ```

  **Explication :** La contrainte `FOREIGN KEY` sur la colonne "typeEmploye" de la table "Employe" garantit que chaque valeur de "typeEmploye" correspond à une valeur existante dans la colonne "code" de la table "typeEmploye", assurant que chaque employé a un type valide.

- **Ajouter les entrées dans la table "typeEmploye" :**

  ```sql
  INSERT INTO typeemploye (code, nom) VALUES ('C', 'Comptable');
  INSERT INTO typeemploye (code, nom) VALUES ('V', 'Visiteur');
  ```

  ![c_et_vcolumn](img/c_et_vcolumn.png)

### Nouveau schéma de la base de données

![schemadb](img/schemadb.png)

## Modifications du code PHP

Avant de modifier le code PHP, il faut ajouter une vue dans la base de données pour que les connexions en tant que visiteur fonctionnent toujours.

### Ajouter une vue

```sql
CREATE VIEW visiteur AS SELECT * FROM employe WHERE typeEmploye = 'V';
```

### Mettre à jour deux utilisateurs dans la base de données

```sql
UPDATE employe SET typeEmploye = 'V' WHERE id = 'a17';
```

**Nous pouvons maintenant continuer.**

## Modifier le code PHP pour distinguer Visiteur et Comptable

Après avoir analysé la documentation et le code source, nous avons identifié que la logique de différenciation se trouve dans le contrôleur `c_connexion.php`.

### Modification de la requête `getInfosUtilisateur`

```php
// Modification de la requête pour sélectionner le typeEmploye
public function getInfosUtilisateur($login, $mdp)
{
    // Récupérer typeEmploye 'C' ou 'V'
    $req = "select id as id, nom as nom, prenom as prenom, typeEmploye as typeEmploye from employe
            where employe.login='$login' and employe.mdp='$mdp'";
    $rs = PdoGsb::$monPdo->query($req);
    $ligne = $rs->fetch();
    return $ligne;
}
```

### Modification du contrôleur de connexion

```php
else {
    $id = $utilisateur['id'];
    $nom = $utilisateur['nom'];
    $prenom = $utilisateur['prenom'];
    $typeEmploye = $utilisateur['typeEmploye']; // Nouvelle ligne pour récupérer le type d'employé
    connecter($id, $nom, $prenom);

    if ($typeEmploye == 'V') {
        // Visiteur
        include("vues/v_menu_visiteur.php");
    } elseif ($typeEmploye == 'C') {
        // Comptable
        include("vues/v_menu_comptable.php");
    }
}
```

**Résumé :** Pour différencier les visiteurs des comptables, nous avons modifié la base de données et le code PHP. La table "Visiteur" a été renommée "Employe" avec une nouvelle colonne "typeEmploye" pour indiquer le type ('C' pour comptable, 'V' pour visiteur). Une table "typeEmploye" a été créée pour stocker les types d'employés. Le code PHP a été ajusté pour tenir compte de cette distinction lors de la connexion, affichant des menus différents selon le type d'utilisateur.
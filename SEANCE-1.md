# LDD, LMD, and Transactions

| **Langage de Définition de Données** | **Langage de Manipulation de Données** | **Langage de Contrôle** |
| ------------------------------------ | -------------------------------------- | ------------------------ |
| CREATE                               | SELECT                                 | COMMIT                   |
| ALTER                                | INSERT                                 | ROLLBACK                 |
| DROP                                 | UPDATE                                 | SAVEPOINT                |
|                                      | DELETE                                 |                          |

---

## LDD (Langage de Définition de Données)

### `CREATE`
```sql
CREATE TABLE nom_Table (
	colonne1 type [DEFAULT valeur] [contrainte de la colonne1],
	colonne2 type [DEFAULT valeur] [contrainte de la colonne2],
	-- Contraintes sur la table
	CONSTRAINT [nom de la contrainte] [contrainte],
	CONSTRAINT [nom de la contrainte] [contrainte]
);
```
#### Exemple:
```sql
CREATE TABLE etudiants (
	id NUMBER(7),
	nom VARCHAR2(50) NOT NULL,
	email VARCHAR2(100),
	note_globale NUMBER(4,2),
	date_inscription DATE DEFAULT TO_DATE('2020-01-01', 'YYYY-MM-DD'),
	-- Contraintes
	CONSTRAINT PK_ID_ETUD PRIMARY KEY (id),
	CONSTRAINT FK_NOTE_ETUD FOREIGN KEY (note_globale) REFERENCES exam(note),
	CONSTRAINT CHK_EMAIL_NN CHECK (email IS NOT NULL)
);
```
_(Voir Cours 1 - Page 30 pour une synthèse de `CREATE` et plus d'exemples.)_

---

### `ALTER`

#### `ADD`
```sql
ALTER TABLE etudiants ADD nom_filiere VARCHAR2(50);
```

#### Ajouter une contrainte
```sql
ALTER TABLE etudiants ADD CONSTRAINT FK_FILIERE_ETUD FOREIGN KEY (nom_filiere) REFERENCES filieres(nm_filiere);
```

#### Renommer une colonne
```sql
ALTER TABLE etudiants RENAME COLUMN nom TO nom_de_famille;
```

#### Renommer une table
```sql
RENAME etudiants TO victimes;
```

#### Supprimer une colonne ou une contrainte
```sql
ALTER TABLE etudiants DROP COLUMN email;
ALTER TABLE etudiants DROP CONSTRAINT CHK_EMAIL_NN;
```

---

### `DROP`
```sql
DROP TABLE etudiants;
```

---

## LMD (Langage de Manipulation de Données)

### `INSERT INTO`
#### Avec colonnes spécifiées
```sql
INSERT INTO etudiants (nom, email, note_globale) VALUES ('Arthur Morgan', 'goodman@ambarinomail.com', 10.50);
```

#### Sans colonnes spécifiées
```sql
INSERT INTO etudiants VALUES (2, 'Gustavo Fring', 'gus@pollashermanas.gov.us', 18.75, TO_DATE('2003-01-01', 'YYYY-MM-DD'));
```

#### Insérer des données filtrées
```sql
INSERT INTO mojtahidin (id, nom, email, note_globale)
SELECT etudiant_id, nom, email, note_globale
FROM etudiants
WHERE note_globale > 17.00;
```

---

### `SELECT FROM`
```sql
SELECT * FROM etudiants;
SELECT id, nom, email FROM etudiants;
SELECT nom, email FROM etudiants WHERE note_globale > 10.0;
```

#### `ORDER BY`
```sql
SELECT id, nom, note_globale FROM etudiants ORDER BY note_globale DESC;
```

#### `GROUP BY`
```sql
SELECT date_inscription, COUNT(*) AS total_etudiants FROM etudiants GROUP BY date_inscription;
```

#### `HAVING`
```sql
SELECT date_inscription, COUNT(*) AS total_etudiants
FROM etudiants
GROUP BY date_inscription
HAVING COUNT(*) > 100;
```

---

### `UPDATE SET`
```sql
UPDATE etudiants SET name = 'Jesse Pinkman' WHERE id = 3;
```

---

### `DELETE FROM`
```sql
DELETE FROM etudiants WHERE id = 2;
```

---

## Transactions

### Commandes sur les Transactions

#### `COMMIT`
```sql
COMMIT;
```

#### `ROLLBACK`
```sql
ROLLBACK;
ROLLBACK TO checkpoint_un;
```

#### `SAVEPOINT`
```sql
SAVEPOINT nom_du_checkpoint;
```

---

### Exemples
```sql
-- Exemples de transactions
BEGIN;

UPDATE etudiants SET note_globale = 10.5 WHERE id = 1;
SAVEPOINT checkpoint_un;

UPDATE etudiants SET note_globale = 19.0 WHERE id = 2;
ROLLBACK TO checkpoint_un;
COMMIT;
```

_(Pour plus d'exemples, voir Chapitre 4, p. 23.)_


# Vues et Jointures

## Vues

### Définition Générale

Une vue est comme une **fenêtre** sur une table, permettant d'accéder uniquement à certains attributs. Une fois créée, elle se comporte comme une table et peut être utilisée avec les commandes LDD et LMD (si les restrictions le permettent).

```sql
CREATE OR REPLACE VIEW [nom_vue] AS [requete_select]
[WITH CHECK OPTION | WITH READ ONLY];
-- CHECK OPTION : Les modifications sont restreintes aux colonnes incluses dans la vue.
-- READ ONLY    : Aucune modification possible via la vue.
```

---

### Exemples

#### Vue non filtrée (sans `WHERE`)

```sql
CREATE OR REPLACE VIEW vue_etudiants_notes AS
SELECT nom, note_globale
FROM etudiants
WITH READ ONLY;
```

Utilisation :
```sql
SELECT * FROM vue_etudiants_notes;
```

| **nom**        | **note_globale** |
|----------------|------------------|
| brad bilik     | 10.5             |
| linc da sinc   | 9.0              |
| t bag          | 19.5             |

---

#### Vue filtrée (avec `WHERE`)

```sql
CREATE VIEW vue_etudiants_excellents AS
SELECT nom, note_globale
FROM etudiants
WHERE note_globale > 17.0;
```

Utilisation :
```sql
SELECT * FROM vue_etudiants_excellents;
```

| **nom**  | **note_globale** |
|----------|------------------|
| t bag    | 19.5             |

---

#### Modifier, Renommer, et Supprimer une Vue

* **Renommer**
```sql
RENAME vue_etudiants_excellents TO mojtahidin;
```

* **Modifier**
```sql
CREATE OR REPLACE VIEW vue_etudiants_excellents AS
SELECT nom, note_globale
FROM etudiants
WHERE note_globale > 18.0;
```

* **Supprimer**
```sql
DROP VIEW vue_etudiants_excellents;
```

---

## Jointures

_(Voir page 6 du cours, Chapitre 3, pour plus d'explications.)_

Les jointures permettent de relier des tables via des clés primaires et étrangères.

| **Type de Jointure**         | **Description**                                   | **Clause SQL**                                                    |
|------------------------------|---------------------------------------------------|-------------------------------------------------------------------|
| INNER JOIN                   | Correspondances **exactes** entre A et B          | `INNER JOIN ... ON`                                               |
| LEFT JOIN                    | **Toutes** les lignes de A + correspondances de B | `LEFT JOIN ... ON`                                                |
| RIGHT JOIN                   | **Toutes** les lignes de B + correspondances de A | `RIGHT JOIN ... ON`                                               |
| FULL OUTER JOIN              | **Toutes** les lignes de A et B                   | `FULL OUTER JOIN ... ON`                                          |
| LEFT JOIN avec `WHERE`       | Lignes de A **sans correspondance** dans B        | `LEFT JOIN ... ON ... WHERE B.Key IS NULL`                        |
| RIGHT JOIN avec `WHERE`      | Lignes de B **sans correspondance** dans A        | `RIGHT JOIN ... ON ... WHERE A.Key IS NULL`                       |
| FULL OUTER JOIN avec `WHERE` | Lignes **sans correspondance** dans A ou B        | `FULL OUTER JOIN ... ON ... WHERE A.Key IS NULL OR B.Key IS NULL` |

---

### Exemples

#### INNER JOIN
```sql
SELECT A.col1, B.col2
FROM TableA A
INNER JOIN TableB B ON A.Key = B.Key;
```

#### LEFT JOIN
```sql
SELECT A.col1, B.col2
FROM TableA A
LEFT JOIN TableB B ON A.Key = B.Key;
```

#### RIGHT JOIN
```sql
SELECT A.col1, B.col2
FROM TableA A
RIGHT JOIN TableB B ON A.Key = B.Key;
```

#### FULL OUTER JOIN
```sql
SELECT A.col1, B.col2
FROM TableA A
FULL OUTER JOIN TableB B ON A.Key = B.Key;
```

#### Lignes sans correspondance (LEFT JOIN avec `WHERE`)
```sql
SELECT A.col1
FROM TableA A
LEFT JOIN TableB B ON A.Key = B.Key
WHERE B.Key IS NULL;
```

---

Pour une meilleure compréhension des jointures, utilisez des outils comme [SQL Joins Visualizer](https://sql-joins.leopard.in.ua/).


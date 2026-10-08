# TP1 JEE : Persistance avec JPA / Hibernate et H2

Mini-projet Java (Maven) qui illustre les bases de la persistance avec **JPA** et **Hibernate** sur une base **H2 en mémoire**. Une entité `Produit` est mappée vers une table, puis une application console insère trois produits et les relit avec une requête JPQL et une recherche par identifiant.

---

## Sommaire

1. [Objectifs pédagogiques](#objectifs-pédagogiques)
2. [Technologies et dépendances](#technologies-et-dépendances)
3. [Structure du projet](#structure-du-projet)
4. [Architecture et fonctionnement](#architecture-et-fonctionnement)
5. [Détail des composants](#détail-des-composants)
6. [Prérequis](#prérequis)
7. [Installation et exécution](#installation-et-exécution)
8. [Résultat attendu](#résultat-attendu)
9. [Configuration de la base de données](#configuration-de-la-base-de-données)
10. [Tests](#tests)
11. [Limites connues et pistes d'amélioration](#limites-connues-et-pistes-damélioration)
12. [Auteur](#auteur)

---

## Objectifs pédagogiques

- Configurer une **unité de persistance JPA** (`persistence.xml`) avec Hibernate comme fournisseur.
- Mapper une classe Java vers une table avec les annotations `@Entity`, `@Id`, `@GeneratedValue`.
- Gérer le cycle de vie d'un `EntityManagerFactory` et d'un `EntityManager`.
- Réaliser des **transactions** en mode `RESOURCE_LOCAL` (begin / commit / rollback).
- Exécuter une requête **JPQL** et une recherche par clé primaire avec `em.find`.
- Utiliser une base **H2 en mémoire** avec génération automatique du schéma.

---

## Technologies et dépendances

| Élément | Version | Rôle |
|---|---|---|
| Java | voir [Prérequis](#prérequis) | Langage |
| Maven | 3.x | Gestion du build et des dépendances |
| `javax.persistence-api` | 2.2 | API JPA (espace de noms `javax.*`) |
| `hibernate-core` | 5.6.5.Final | Implémentation JPA |
| `h2` | 2.1.214 | Base de données embarquée en mémoire |
| `slf4j-api` | 1.7.36 | Façade de journalisation |
| `slf4j-simple` | 1.7.36 | Implémentation simple des logs SLF4J |

- **groupId** : `com.example`
- **artifactId** : `tpp1`
- **version** : `1.0-SNAPSHOT`
- **packaging** : `jar`
- **encodage des sources** : UTF-8

---

## Structure du projet

```
Tp1-JEE/
├── .idea/                          # Configuration IntelliJ IDEA (partielle)
├── .gitignore
├── pom.xml                         # Configuration Maven et dépendances
└── src/
    ├── main/
    │   ├── java/com/example/
    │   │   ├── App.java            # Point d'entrée : insertion puis lecture
    │   │   └── model/
    │   │       └── Produit.java    # Entité JPA
    │   └── resources/META-INF/
    │       └── persistence.xml     # Unité de persistance "hibernate-demo"
    └── test/java/com/example/
        └── AppTest.java            # Test JUnit 3 généré par défaut
```

---

## Architecture et fonctionnement

```
 App.main()
    │
    ├─ Persistence.createEntityManagerFactory("hibernate-demo")
    │        └─ lit META-INF/persistence.xml  ──►  Hibernate  ──►  H2 (mémoire)
    │                                                   └─ crée le schéma (create-drop)
    ├─ insererProduits(emf)   ─► transaction : persist x3, commit
    ├─ lireProduits(emf)      ─► JPQL "SELECT p FROM Produit p" + em.find(Produit.class, 2L)
    └─ emf.close()            ─► libère les ressources, le schéma est supprimé
```

Déroulement :

1. L'`EntityManagerFactory` est créée à partir de l'unité `hibernate-demo`. Hibernate se connecte à H2 et génère la table `Produit`.
2. `insererProduits` ouvre un `EntityManager`, démarre une transaction, persiste trois produits puis valide. En cas d'exception, un `rollback` est effectué si la transaction est active, et l'`EntityManager` est toujours fermé dans le bloc `finally`.
3. `lireProduits` ouvre un nouvel `EntityManager`, liste tous les produits via JPQL, puis recherche le produit d'identifiant `2`.
4. La fabrique est fermée. Comme la base est en mémoire avec `create-drop`, toutes les données disparaissent à la fin du programme.

---

## Détail des composants

### `com.example.model.Produit` (entité)

| Attribut | Type | Mapping |
|---|---|---|
| `id` | `Long` | Clé primaire, `@GeneratedValue(strategy = GenerationType.IDENTITY)` (auto-incrément géré par la base) |
| `nom` | `String` | Colonne `nom` |
| `prix` | `BigDecimal` | Colonne `prix` (type décimal, adapté aux montants) |

La classe fournit un constructeur sans argument (obligatoire pour JPA), un constructeur `Produit(String, BigDecimal)`, les getters/setters et un `toString()` de la forme :

```
Produit{id=1, Nom='Laptop', Prix=999.99}
```

### `com.example.App` (application console)

| Méthode | Rôle |
|---|---|
| `main` | Crée l'`EntityManagerFactory`, appelle l'insertion puis la lecture, ferme la fabrique |
| `insererProduits` | Persiste dans une transaction : Laptop (999.99), Smartphone (499.99), Tablette (299.99) |
| `lireProduits` | Affiche tous les produits (JPQL), puis cherche le produit d'ID 2 avec `em.find` |

### `META-INF/persistence.xml`

Unité de persistance `hibernate-demo`, en mode `RESOURCE_LOCAL`, avec `org.hibernate.jpa.HibernatePersistenceProvider` comme fournisseur. Voir la section [Configuration de la base de données](#configuration-de-la-base-de-données).

---

## Prérequis

- **JDK** installé et `JAVA_HOME` configuré. Le `pom.xml` ne fixe pas de version de compilation ; Hibernate 5.6 est conçu pour les JDK 8 à 17. Le projet est configuré dans IntelliJ avec un JDK 25, mais cette combinaison n'a pas été vérifiée : en cas de problème, utilisez un JDK 11 ou 17.
- **Maven 3.x** (ou le Maven intégré à l'IDE).
- Aucune installation de base de données : H2 est embarquée.

---

## Installation et exécution

### Cloner le dépôt

```bash
git clone https://github.com/lahlalimouad6-ux/Tp1-JEE.git
cd Tp1-JEE
```

### Depuis un IDE (recommandé)

1. Ouvrir le dossier dans IntelliJ IDEA (le projet est détecté via `pom.xml`) ou Eclipse (importer comme projet Maven existant).
2. Laisser l'IDE télécharger les dépendances.
3. Exécuter la classe `com.example.App` (clic droit, *Run 'App.main()'*).

### En ligne de commande

Le `pom.xml` ne déclare pas de plugin d'exécution. Vous pouvez néanmoins lancer l'application avec le plugin `exec` appelé directement :

```bash
mvn compile org.codehaus.mojo:exec-maven-plugin:3.1.0:java -Dexec.mainClass=com.example.App
```

> Pour simplifier la commande, ajoutez `exec-maven-plugin` à la section `<build>` du `pom.xml` (voir [pistes d'amélioration](#limites-connues-et-pistes-damélioration)).

---

## Résultat attendu

Avec `hibernate.show_sql=true` et `hibernate.format_sql=true`, la console affiche les requêtes SQL générées par Hibernate, mêlées aux messages du programme. Les lignes importantes sont :

```
Produits insérés avec succès !

Liste des produits :
Produit{id=1, Nom='Laptop', Prix=999.99}
Produit{id=2, Nom='Smartphone', Prix=499.99}
Produit{id=3, Nom='Tablette', Prix=299.99}

Recherche du produit avec ID=2 :
Produit{id=2, Nom='Smartphone', Prix=499.99}
```

Parmi les requêtes SQL affichées, on retrouve la création de la table (équivalent à `create table Produit (id bigint generated by default as identity, nom varchar(255), prix decimal(19,2), primary key (id))`), trois `insert` et un `select`. Le détail exact des logs peut varier selon la version de Hibernate et de H2.

---

## Configuration de la base de données

| Propriété | Valeur | Signification |
|---|---|---|
| `javax.persistence.jdbc.driver` | `org.h2.Driver` | Pilote JDBC H2 |
| `javax.persistence.jdbc.url` | `jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1` | Base en mémoire nommée `testdb`, conservée tant que la JVM tourne |
| `javax.persistence.jdbc.user` | `sa` | Utilisateur par défaut de H2 |
| `javax.persistence.jdbc.password` | *(vide)* | Pas de mot de passe |
| `hibernate.dialect` | `org.hibernate.dialect.H2Dialect` | Dialecte SQL pour H2 |
| `hibernate.hbm2ddl.auto` | `create-drop` | Crée le schéma au démarrage, le supprime à la fermeture de la fabrique |
| `hibernate.show_sql` | `true` | Affiche les requêtes SQL |
| `hibernate.format_sql` | `true` | Formate les requêtes SQL pour les rendre lisibles |

### Passer à une base persistante

Pour conserver les données sur disque, remplacez l'URL par un fichier H2, par exemple `jdbc:h2:./data/tp1db`, et utilisez `hibernate.hbm2ddl.auto=update` au lieu de `create-drop`.

---

## Tests

Le fichier `AppTest.java` est le test JUnit 3 généré par l'archétype Maven : il ne contient que `assertTrue(true)` et ne teste pas encore la logique de l'application.

---

## Limites connues et pistes d'amélioration

Points relevés dans la configuration actuelle :

- **Dépendance JUnit manquante** : `AppTest` importe `junit.framework.*`, mais `pom.xml` ne déclare aucune dépendance JUnit. La compilation des tests (`mvn test`, `mvn package`) échouera tant que cette dépendance n'est pas ajoutée, par exemple :
  ```xml
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
  </dependency>
  ```
- **Version de Java non fixée** : ajouter `maven.compiler.source` / `maven.compiler.target` (ou `maven.compiler.release`) dans `<properties>` pour garantir un build reproductible.
- **Pas de plugin d'exécution** : ajouter `exec-maven-plugin` avec `mainClass` à `com.example.App` pour lancer `mvn exec:java`.
- **Données figées** : les trois produits sont codés en dur dans `App`.

Idées d'évolution :

- Ajouter des opérations de mise à jour et de suppression (CRUD complet).
- Extraire l'accès aux données dans une classe DAO ou Repository.
- Ajouter des tests unitaires réels (insertion, recherche, produit inexistant).
- Ajouter des contraintes de validation (`@Column(nullable = false)`, `@Table`, etc.).
- Migrer vers `jakarta.persistence` (Hibernate 6) ou vers une architecture Java EE / Jakarta EE avec serveur d'applications.

---

## Auteur

Projet réalisé par **lahlalimouad6-ux** dans le cadre d'un TP de Java EE.

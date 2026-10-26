# TP2 REST : Service Web « Calculatrice » avec JAX-RS et Tomcat

**Faculté des Sciences de Sfax**  
**Département d'informatique et des Communications**  
*Année Universitaire : 2026-2027*  
*Matière :* Architecture MicroServices  
*Filière :* MPSRCC1  
*Enseignantes :* Corinne Amel ZAYANI, Amal ABID  

---

## 📋 Table des Matières
1. [Présentation et Objectifs](#-présentation-et-objectifs)
2. [Architecture et Structure du Projet](#-architecture-et-structure-du-projet)
3. [Partie A : Conception du Service (Questions A.2)](#-partie-a--conception-du-service)
4. [Partie C : Premier Service REST - La Somme (Question C.1)](#-partie-c--premier-service-rest--la-somme)
5. [Partie D : Compléter la Calculatrice (Questions D.1, D.2, D.3)](#-partie-d--compléter-la-calculatrice)
6. [Partie E : Échanger des Données en JSON (Questions E.1, E.2)](#-partie-e--échanger-des-données-en-json)
7. [Partie F : La Ressource « operations » (Tests F.3, F.4, F.5)](#-partie-f--la-ressource--operations-)
8. [Questions de Synthèse](#-questions-de-synthèse)

---

## 🎯 Présentation et Objectifs

Ce projet implémente un service web RESTful de calculatrice avec historique des opérations en utilisant le framework **JAX-RS** (implémentation **Jersey 3.1.9**) déployé sur le conteneur de servlets **Apache Tomcat 10.1.x** (compatible **Jakarta EE 10** / Java 17+).

### Technologies utilisées
- **Java SE 17**
- **Jakarta RESTful Web Services (JAX-RS 3.1.0)** (`jakarta.ws.rs-api`)
- **Jersey 3.1.9** (`jersey-container-servlet`, `jersey-hk2`, `jersey-media-json-binding`)
- **Apache Tomcat 10.1.61**
- **Apache Maven 3.9+**
- Outils de test : **Postman**, **cURL**

---

## 🏗️ Architecture et Structure du Projet

```text
tp2-ms/
├── pom.xml
├── capture/                  # Captures d'écran des tests curl et Postman
├── question/
│   └── reponse               # Brouillon des réponses
├── operation.json            # Payload exemple pour POST /operations
├── modification.json         # Payload exemple pour PUT /operations
└── src/
    └── main/
        └── java/
            ├── com/exemple/rest/
            │   ├── Resultat.java             # DTO pour retour JSON (/somme/detail, /division/detail)
            │   └── Operation.java            # Modèle d'opération pour l'historique
            └── org/example/
                ├── Calculatrice.java         # Logique métier pure (Java standard)
                ├── RestApplication.java      # Déclaration de l'application (@ApplicationPath("/api"))
                ├── CalculRest.java           # Ressource REST /calcul
                └── OperationRest.java        # Ressource REST CRUD /operations
```

---

## 📝 Partie A : Conception du Service

### A.1 Tableau des Familles d'URI

#### Famille 1 : Les calculs directs (sans persistance)
| Verbe HTTP | URI | Rôle |
| :--- | :--- | :--- |
| `GET` | `/calcul/somme?a=2&b=3` | Retourne $a + b$ |
| `GET` | `/calcul/soustraction?a=2&b=3` | Retourne $a - b$ |
| `GET` | `/calcul/produit?a=2&b=3` | Retourne $a \times b$ |
| `GET` | `/calcul/division?a=6&b=3` | Retourne $a \div b$ |

#### Famille 2 : L'historique des opérations (ressources persistées en mémoire)
| Verbe HTTP | URI | Rôle |
| :--- | :--- | :--- |
| `POST` | `/operations` | Crée une opération (calculée et enregistrée) |
| `GET` | `/operations` | Retourne toutes les opérations enregistrées |
| `GET` | `/operations/{id}` | Retourne l'opération numéro `id` |
| `PUT` | `/operations/{id}` | Modifie l'opération numéro `id` |
| `DELETE` | `/operations/{id}` | Supprime l'opération numéro `id` |

---

### A.2 Questions de Conception

#### 1. Pourquoi les calculs directs utilisent-ils le verbe GET et non POST ?
> **Réponse :**  
> Les calculs directs sont de simples consultations ou calculs à la volée qui ne créent ni ne modifient aucune ressource sur le serveur. En architecture REST, le verbe **`GET`** est sûr (*safe*) et idempotent, ce qui est adapté pour demander un résultat sans effet de bord, contrairement à **`POST`** qui sert à soumettre des données pour créer ou modifier un état sur le serveur.

#### 2. Dans l'URI `/calcul/somme?a=2&b=3`, comment s'appelle la partie située après le point d'interrogation ?
> **Réponse :**  
> La partie située après le point d'interrogation (`?a=2&b=3`) s'appelle la **chaîne de requête** (ou ***query string*** / paramètres de requête).

#### 3. Quelle différence y a-t-il entre `/operations` et `/operations/3` ?
> **Réponse :**  
> `/operations` désigne la **collection complète** de toutes les opérations enregistrées, tandis que `/operations/3` désigne une **ressource individuelle précise** au sein de cette collection, identifiée par son identifiant unique `id = 3`.

#### 4. Quel code de statut HTTP le serveur doit-il renvoyer si le client demande `/operations/99` et que cette opération n'existe pas ?
> **Réponse :**  
> Le serveur doit renvoyer le code de statut HTTP **`404 Not Found`** (Non trouvé), indiquant que la ressource ciblée par l'identifiant `99` n'existe pas.

---

## ⚡ Partie C : Premier Service REST : la somme

### Question C.1
**Remplacez GET par POST dans Postman et renvoyez la requête sur `/api/calcul/somme?a=2&b=3`. Quel code de statut obtenez-vous ? Expliquez-le.**

- **Code de statut obtenu :** `405 Method Not Allowed`
- **Capture d'écran correspondante :** [`capture/Capture d'écran 2026-10-24 161649.png`](capture/Capture%20d'écran%202026-10-24%20161649.png)
- **Explication :**  
  La méthode `somme` dans la classe Java `CalculRest` est annotée exclusivement avec `@GET`. Le serveur Tomcat / Jersey reconnaît l'URI `/calcul/somme`, mais constate qu'aucune méthode Java n'est mappée pour traiter le verbe HTTP `POST` sur cette route. Il renvoie donc une erreur 405 pour indiquer que la méthode HTTP n'est pas autorisée sur cette ressource.

---

## 🔍 Partie D : Compléter la Calculatrice

### D.3 Observations

#### 1. Requête `GET /RestWebApp/api/calcul/somme?a=2` (paramètre `b` absent) :
- **Statut obtenu :** `200 OK`
- **Corps obtenu :** `2`
- **Capture d'écran :** [`capture/Capture d'écran 2026-10-24 162423.png`](capture/Capture%20d'écran%202026-10-24%20162423.png)

#### 2. Requête `GET /RestWebApp/api/calcul/somme?a=abc&b=3` (paramètre `a` non entier) :
- **Statut obtenu :** `404 Not Found` (ou `400 Bad Request` selon les configurations JAX-RS / Tomcat)
- **Corps obtenu :** Page HTML d'erreur standard de Tomcat
- **Capture d'écran :** [`capture/Capture d'écran 2026-10-24 162507.png`](capture/Capture%20d'écran%202026-10-24%20162507.png)

---

### Questions D.1, D.2 et D.3

#### Question D.1 : Quelle valeur JAX-RS donne-t-il à un paramètre entier absent ?
> **Réponse :**  
> Si le paramètre annoté `@QueryParam` est de type primitif Java (`int`), JAX-RS lui affecte la valeur par défaut du type en Java, c'est-à-dire **`0`**. Ainsi, `somme(2, 0)` retourne `2`.  
> *(Remarque : Si le type avait été l'objet wrapper `Integer`, sa valeur aurait été `null`).*

#### Question D.2 : Quel statut obtenez-vous lorsque le paramètre n'est pas un entier ? La méthode somme a-t-elle été exécutée ?
> **Réponse :**  
> On obtient le statut HTTP **`404 Not Found`** (ou `400 Bad Request`).  
> **Non**, la méthode `somme()` n'a **pas** été exécutée : JAX-RS échoue lors de la phase de désérialisation et de conversion du paramètre en entier (`Integer.parseInt`), ce qui interrompt immédiatement le traitement et rejette la requête avant tout appel à la méthode métier.

#### Question D.3 : Quelle différence y a-t-il entre `@QueryParam` et `@PathParam` ? Laquelle de ces deux annotations utilisera-t-on pour lire l'identifiant dans `/operations/{id}` ?
> **Réponse :**  
> - **`@QueryParam`** extrait une valeur transmise dans les paramètres de la chaîne de requête URL après le point d'interrogation `?` (ex. `?a=2&b=3`). Ces paramètres sont typiquement utilisés pour filtrer, paginer ou passer des options secondaires.  
> - **`@PathParam`** extrait un segment variable directement intégré dans le chemin de l'URI (ex. `/produit/{a}/{b}` ou `/operations/{id}`).  
> Pour lire l'identifiant dans `/operations/{id}`, on utilise obligatoirement **`@PathParam("id")`**, car l'identifiant fait partie intégrante du chemin d'accès à la ressource.

---

## 📦 Partie E : Échanger des Données en JSON

### Question E.1
**Vous n'avez écrit aucune ligne de code pour fabriquer ce texte JSON. Qui a effectué la conversion ? D'où viennent les noms a, b, operation et resultat ?**

> **Réponse :**  
> - **Qui effectue la conversion ?**  
>   La conversion automatique de l'objet Java `Resultat` en flux JSON est assurée par le fournisseur de liaison JSON de Jersey (`jersey-media-json-binding` / Jackson / JSON-B) configuré dans les dépendances Maven. Grâce à l'annotation `@Produces(MediaType.APPLICATION_JSON)`, JAX-RS sélectionne le bon `MessageBodyWriter` pour sérialiser l'objet.
> - **D'où viennent les noms des champs ?**  
>   Les noms `a`, `b`, `operation` et `resultat` proviennent des propriétés de la classe `Resultat`, découverts via la convention JavaBean à partir des accesseurs (*getters* : `getA()`, `getB()`, `getOperation()`, `getResultat()`).

---

### Question E.2
**Pourquoi la seconde requête (`curl -i -H "Accept: application/xml" ...`) est-elle refusée ?**

- **Statut obtenu :** `406 Not Acceptable`
- **Captures d'écran :**
  - Accept `application/json` (200 OK) : [`capture/Capture d'écran 2026-10-24 170917.png`](capture/Capture%20d'écran%202026-10-24%20170917.png)
  - Accept `application/xml` (406 Not Acceptable) : [`capture/Capture d'écran 2026-10-24 171020.png`](capture/Capture%20d'écran%202026-10-24%20171020.png)
- **Explication :**  
  Le client spécifie via l'en-tête `Accept: application/xml` qu'il n'accepte qu'une réponse au format XML. Or, la ressource `sommeDetail` ne déclare produire que du JSON via `@Produces(MediaType.APPLICATION_JSON)`. Le serveur ne disposant d'aucun moyen de convertir la réponse au format demandé par le client (échec de négociation de contenu), il rejette la requête avec le statut `406 Not Acceptable`.

---

### E.4 Exercice : divisionDetail en JSON
- Méthode `GET /calcul/division/detail?a=10&b=2` avec `Accept: application/json`
- **Statut :** `200 OK`
- **Corps :** `{"a":10,"b":2,"operation":"division","resultat":5.0}`
- **Capture d'écran :** [`capture/Capture d'écran 2026-10-24 171609.png`](capture/Capture%20d'écran%202026-10-24%20171609.png)

---

## 🚀 Partie F : La Ressource « operations »

### F.3 Série de Tests (Création et Lecture)

Base URL : `http://localhost:8080/tp2_ms_war_exploded/api/operations`

| N° | Méthode & URI | Corps envoyé | Statut attendu | Corps & En-têtes attendus | Capture d'écran associée |
| :---: | :--- | :--- | :---: | :--- | :--- |
| **1** | `POST /operations` | `{"operateur":"somme","a":8,"b":2}` | `201 Created` | En-tête `Location: .../operations/1`<br>`{"a":8,"b":2,"id":1,"operateur":"somme","resultat":10.0}` | [`capture/Capture d'écran 2026-10-24 174755.png`](capture/Capture%20d'écran%202026-10-24%20174755.png) (curl)<br>[`capture/Capture d'écran 2026-10-24 175228.png`](capture/Capture%20d'écran%202026-10-24%20175228.png) (Postman) |
| **2** | `POST /operations` | `{"operateur":"division","a":7,"b":2}` | `201 Created` | En-tête `Location: .../operations/2`<br>`{"a":7,"b":2,"id":2,"operateur":"division","resultat":3.5}` | [`capture/Capture d'écran 2026-10-24 175423.png`](capture/Capture%20d'écran%202026-10-24%20175423.png) |
| **3** | `GET /operations` | *Aucun* | `200 OK` | Tableau JSON contenant toutes les opérations créées | [`capture/Capture d'écran 2026-10-24 175505.png`](capture/Capture%20d'écran%202026-10-24%20175505.png) |
| **4** | `GET /operations/2` | *Aucun* | `200 OK` | Objet JSON de l'opération d'identifiant `2` | [`capture/Capture d'écran 2026-10-24 175535.png`](capture/Capture%20d'écran%202026-10-24%20175535.png) |
| **5** | `GET /operations/99` | *Aucun* | `404 Not Found` | Corps vide ou `{"erreur":"Operation introuvable"}` | [`capture/Capture d'écran 2026-10-24 175611.png`](capture/Capture%20d'écran%202026-10-24%20175611.png) |
| **6** | `POST /operations` | `{"operateur":"division","a":7,"b":0}` | `400 Bad Request` | `{"erreur":"Calcul impossible"}` | [`capture/Capture d'écran 2026-10-24 175720.png`](capture/Capture%20d'écran%202026-10-24%20175720.png) |
| **7** | `POST /operations` | `{"operateur":"racine","a":9,"b":0}` | `400 Bad Request` | `{"erreur":"Calcul impossible"}` | [`capture/Capture d'écran 2026-10-24 175806.png`](capture/Capture%20d'écran%202026-10-24%20175806.png) |

---

### F.4 Modifier une opération : PUT

| Situation | Méthode & URI | Corps envoyé | Statut | Réponse obtenue | Capture d'écran |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **Modification réussie** | `PUT /operations/1` | `{"operateur":"somme","a":3,"b":3}` | `200 OK` | `{"a":3,"b":3,"id":1,"operateur":"somme","resultat":6.0}` | [`capture/Capture d'écran 2026-10-24 180859.png`](capture/Capture%20d'écran%202026-10-24%20180859.png) |
| **Identifiant inexistant** | `PUT /operations/4` | `{"operateur":"somme","a":3,"b":3}` | `404 Not Found` | `{"erreur":"Operation introuvable"}` | [`capture/Capture d'écran 2026-10-24 180942.png`](capture/Capture%20d'écran%202026-10-24%20180942.png) |
| **Calcul impossible (div/0)** | `PUT /operations/2` | `{"operateur":"division","a":3,"b":0}` | `400 Bad Request` | `{"erreur":"Calcul impossible"}` | [`capture/Capture d'écran 2026-10-24 181105.png`](capture/Capture%20d'écran%202026-10-24%20181105.png) |

---

### F.5 Supprimer une opération : DELETE

Ordre d'exécution des requêtes de test :
1. `DELETE /operations/3` (opération existante) :
   - **Statut obtenu :** `204 No Content`
   - **Corps :** Vide
   - **Capture d'écran :** [`capture/Capture d'écran 2026-10-24 182116.png`](capture/Capture%20d'écran%202026-10-24%20182116.png)
2. `GET /operations/3` (vérification de la suppression) :
   - **Statut obtenu :** `404 Not Found`
   - **Corps :** `{"erreur":"Operation introuvable"}`
   - **Capture d'écran :** [`capture/Capture d'écran 2026-10-24 182241.png`](capture/Capture%20d'écran%202026-10-24%20182241.png)
3. `DELETE /operations/3` (suppression d'une ressource déjà supprimée) :
   - **Statut obtenu :** `404 Not Found`
   - **Corps :** `{"erreur":"Operation introuvable"}`
   - **Capture d'écran :** [`capture/Capture d'écran 2026-10-24 182313.png`](capture/Capture%20d'écran%202026-10-24%20182313.png)

---

## 💡 Questions de Synthèse

### 1. Tableau complet des opérations avec les codes de statut HTTP possibles

> Capture d'écran du récapitulatif : [`capture/Capture d'écran 2026-10-24 182615.png`](capture/Capture%20d'écran%202026-10-24%20182615.png)

| Verbe HTTP | URI | Rôle | Codes HTTP possibles |
| :--- | :--- | :--- | :--- |
| **GET** | `/calcul/somme?a=2&b=3` | Retourne $a + b$ | `200 OK`, `400 Bad Request` / `404 Not Found` |
| **GET** | `/calcul/soustraction?a=2&b=3` | Retourne $a - b$ | `200 OK`, `400 Bad Request` / `404 Not Found` |
| **GET** | `/calcul/produit?a=2&b=3` | Retourne $a \times b$ | `200 OK`, `400 Bad Request` / `404 Not Found` |
| **GET** | `/calcul/division?a=6&b=3` | Retourne $a \div b$ | `200 OK`, `400 Bad Request` (division par 0) |
| **POST** | `/operations` | Crée et enregistre une opération | `201 Created`, `400 Bad Request` |
| **GET** | `/operations` | Retourne toutes les opérations enregistrées | `200 OK` |
| **GET** | `/operations/{id}` | Retourne l'opération numéro `id` | `200 OK`, `404 Not Found` |
| **PUT** | `/operations/{id}` | Modifie l'opération existante numéro `id` | `200 OK`, `400 Bad Request`, `404 Not Found` |
| **DELETE** | `/operations/{id}` | Supprime l'opération existante numéro `id` | `204 No Content`, `404 Not Found` |

---

### 2. Annotations JAX-RS utilisées dans ce TP et leur rôle

| Annotation | Rôle en une phrase |
| :--- | :--- |
| `@Path` | Définit le chemin d'URI relatif assigné à une classe de ressource ou à une méthode web. |
| `@GET` | Indique que la méthode Java traitera les requêtes HTTP ayant pour verbe `GET`. |
| `@POST` | Indique que la méthode Java traitera les requêtes HTTP ayant pour verbe `POST`. |
| `@PUT` | Indique que la méthode Java traitera les requêtes HTTP ayant pour verbe `PUT`. |
| `@DELETE` | Indique que la méthode Java traitera les requêtes HTTP ayant pour verbe `DELETE`. |
| `@PathParam` | Injecte dans un argument Java la valeur d'une variable déclarée dans le gabarit d'URI (ex: `{id}`). |
| `@QueryParam` | Injecte dans un argument Java la valeur d'un paramètre transmis dans la chaîne de requête (après `?`). |
| `@Produces` | Spécifie le(s) type(s) MIME que la ressource génère en réponse (ex: `application/json`, `text/plain`). |
| `@Consumes` | Spécifie le(s) type(s) MIME que la méthode accepte dans le corps de la requête reçue (ex: `application/json`). |
| `@ApplicationPath` | Configure le préfixe de chemin d'accès de base pour l'ensemble des ressources JAX-RS (ici `/api`). |
| `@Context` | Permet d'injecter des objets contextuels du conteneur JAX-RS (comme `UriInfo` pour manipuler l'URI courante). |

---

### 3. Quelle différence y a-t-il entre JAX-RS et Jersey ?
> **Réponse :**  
> - **JAX-RS** (*Jakarta RESTful Web Services*) est une **spécification standard** de Jakarta EE définissant les interfaces et annotations (`@Path`, `@GET`, etc.) sans fournir de code exécutable.  
> - **Jersey** est l'**implémentation logicielle concrète** de cette spécification (développée par Eclipse/GlassFish) qui fournit le moteur d'exécution, la sérialisation JSON, le cycle de vie des requêtes et l'intégration avec le conteneur servlet Tomcat.

---

### 4. Quel est le rôle de Tomcat ? Pourquoi le projet produit-il un fichier .war et non un fichier .jar exécutable ?
> **Réponse :**  
> - **Rôle de Tomcat :** Tomcat est un conteneur de servlets (*Servlet Container*) et serveur web qui fournit l'environnement d'exécution réseau, gère le cycle de vie des composants web, écoute sur le port 8080 et dispatche les requêtes HTTP vers l'application.  
> - **Pourquoi `.war` et non `.jar` ?**  
>   Le format `.war` (*Web Application Archive*) est le standard d'archivage déployable dans un conteneur d'applications externe tel que Tomcat. Contrairement aux architectures avec serveur embarqué (comme Spring Boot avec `.jar` autonome), ce TP repose sur un serveur Tomcat externe gérant le déploiement du livrable `.war`.

---

### 5. Pourquoi la conservation des opérations ne contredit-elle pas le principe stateless (sans état) ?
> **Réponse :**  
> Le principe **stateless** interdit au serveur de stocker un état de session ou de contexte conversationnel propre à un client entre deux requêtes ; chaque requête doit être auto-suffisante.  
> La collection des opérations stockée sur le serveur représente l'**état des ressources du domaine** (les données applicatives, à l'image d'une base de données), et non un état de session utilisateur. Chaque requête contient l'intégralité des informations nécessaires à son exécution (identifiant, données JSON). Le principe stateless est donc parfaitement respecté.

---

### 6. Comparaison de l'effet de la répétition des requêtes POST et PUT

- **Répéter deux fois `POST /operations` avec le même corps :**  
  La méthode `POST` n'est **pas idempotente**. Chaque exécution crée une nouvelle ressource avec un nouvel identifiant incrémenté :
  - 1ère exécution : crée l'opération `id = 1`
  - 2ème exécution identique : crée une nouvelle opération `id = 2`
  - *Effet :* Deux enregistrements distincts sont ajoutés dans la collection.

- **Répéter deux fois `PUT /operations/1` avec le même corps :**  
  La méthode `PUT` est **idempotente**. Elle remplace la ressource située précisément à l'identifiant `/operations/1` :
  - 1ère exécution : met à jour l'opération `id = 1` avec les nouvelles valeurs
  - 2ème exécution identique : réapplique les mêmes valeurs à l'opération `id = 1`
  - *Effet :* L'état final du système reste strictement inchangé après la première mise à jour.
#   - T P 2 - R E S T - S e r v i c e - w e b - C a l c u l a t r i c e - a v e c - J A X - R S - e t - T o m c a t  
 
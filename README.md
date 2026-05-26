TP 20 – APPLICATION NUMBER BOOK : CONTACTS ANDROID ET API DISTANTE VIA RETROFIT

Cours : Programmation Mobile – Android avec Java


CONTEXTE GÉNÉRAL

Ce laboratoire consiste à développer une application Android nommée Number Book. L’application lit les contacts du téléphone, les affiche dans une interface, puis les envoie vers un serveur distant pour stockage dans une base de données. Une fois les contacts enregistrés, l’application permet d’effectuer une recherche distante par nom ou par numéro.

Le TP couvre plusieurs aspects essentiels :
- accès aux données système (contacts)
- gestion des permissions dynamiques
- communication client/serveur (API REST)
- sérialisation JSON
- utilisation de Retrofit pour les requêtes réseau
- affichage avec RecyclerView


OBJECTIFS PÉDAGOGIQUES

- Lire les contacts de l’appareil (ContactsContract).
- Gérer les permissions runtime (READ_CONTACTS, INTERNET).
- Créer une base de données distante MySQL et une API PHP.
- Consommer l’API avec Retrofit (GET, POST).
- Afficher les contacts locaux et les résultats de recherche distante dans un RecyclerView.
- Comprendre les appels asynchrones et le thread principal.


RÉSULTAT ATTENDU

L’application finale permet de :
- visualiser la liste des contacts du téléphone
- envoyer tous les contacts vers un serveur distant
- rechercher un contact par nom ou par numéro via l’API distante
- afficher les résultats de recherche


SCÉNARIO DE FONCTIONNEMENT

1. L’utilisateur accorde les permissions.
2. L’application charge les contacts locaux (nom + numéro) et les affiche.
3. L’utilisateur clique sur “Synchroniser” → les contacts sont envoyés au serveur (POST).
4. L’utilisateur saisit un nom ou numéro → requête GET vers l’API → affichage des correspondances.


PARTIE 1 – CONCEPTION DE LA BASE DE DONNÉES DISTANTE

Créer une base MySQL nommée “numberbook”. Une seule table “contacts” avec :

id INT PRIMARY KEY AUTO_INCREMENT
name VARCHAR(100) NOT NULL
phone VARCHAR(30) NOT NULL
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP


PARTIE 2 – DÉVELOPPEMENT DU BACKEND DISTANT (PHP)

Créer deux scripts PHP dans le dossier du serveur (htdocs) :

A. add_contact.php (POST)
- Reçoit name et phone via $_POST.
- Exécute INSERT dans la table contacts.
- Retourne JSON {“success”: true/false}.

B. search_contact.php (GET)
- Reçoit paramètre “query” via $_GET.
- Recherche WHERE name LIKE %query% OR phone LIKE %query%.
- Retourne JSON {“results”: [ {“name”: “...”, “phone”: “...”} ] }.

Utiliser PDO avec requêtes préparées pour éviter les injections SQL.


PARTIE 3 – DÉVELOPPEMENT ANDROID

Créer un projet Android (Empty Activity). Nom : NumberBook. Langage Java. Min SDK 24.

Ajouter les dépendances dans build.gradle (Module : app) :

implementation ‘com.squareup.retrofit2:retrofit:2.9.0’
implementation ‘com.squareup.retrofit2:converter-gson:2.9.0’
implementation ‘androidx.recyclerview:recyclerview:1.3.2’
implementation ‘com.google.android.material:material:1.11.0’


PARTIE 4 – INTERFACE UTILISATEUR (LAYOUT)

activity_main.xml :

- EditText pour saisie de recherche (id = edtSearch)
- Button “RECHERCHER” (id = btnSearch)
- Button “SYNCHRONISER” (id = btnSync)
- RecyclerView (id = rvContacts) pour afficher les résultats ou la liste locale
- TextView ou ProgressBar optionnel


PARTIE 5 – MODÈLES ANDROID

Créer une classe ContactModel (package model) :

public class ContactModel {
    private String name;
    private String phone;
    // constructeurs, getters, setters
}


PARTIE 6 – COMMUNICATION RÉSEAU AVEC RETROFIT

Créer une interface ApiService (package network) :

public interface ApiService {
    @POST(“add_contact.php”)
    @FormUrlEncoded
    Call<ResponseBody> addContact(
        @Field(“name”) String name,
        @Field(“phone”) String phone
    );

    @GET(“search_contact.php”)
    Call<SearchResponse> searchContact(@Query(“query”) String query);
}

Créer une classe SearchResponse (ou utiliser un objet générique avec JsonArray).

Initialiser Retrofit dans une classe ApiClient :

Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(“http://IP_SERVEUR/numberbook/”)
    .addConverterFactory(GsonConverterFactory.create())
    .build();


PARTIE 7 – AFFICHAGE AVEC RECYCLERVIEW

Créer un adapter ContactAdapter (extends RecyclerView.Adapter). ViewHolder avec deux TextView (name, phone). La liste est mise à jour via setContacts() et notifyDataSetChanged().


PARTIE 8 – MAINACTIVITY

Dans MainActivity :

- Déclarer : RecyclerView, ContactAdapter, List<ContactModel> contactList, ApiService.
- Dans onCreate : initialiser les vues, le layout manager, l’adapter.
- Vérifier la permission READ_CONTACTS. Si non accordée, la demander.
- Appeler loadLocalContacts() pour lire les contacts du téléphone et les afficher.
- Au clic sur “SYNCHRONISER” : itérer sur la liste locale et appeler addContact() pour chaque contact (ou envoyer tous en une fois si API modifiée).
- Au clic sur “RECHERCHER” : récupérer le texte, appeler searchContact() via Retrofit, et mettre à jour l’adapter avec les résultats.


PARTIE 9 – EXPLICATION DU CODE DE MAINACTIVITY

- loadLocalContacts() : utiliser ContentResolver, query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI), récupérer DISPLAY_NAME et NUMBER. Construire la liste ContactModel.
- Les appels Retrofit sont asynchrones (enqueue). Le callback onResponse met à jour l’UI (runOnUiThread ou directement car le callback s’exécute sur le thread principal dans les versions récentes).
- Gérer les erreurs réseau dans onFailure (Toast).


PARTIE 10 – TESTS DÉTAILLÉS

Test 1 – Lecture locale :
Lancer l’app. Si permission accordée, la liste des contacts s’affiche.

Test 2 – Synchronisation :
Cliquer sur “SYNCHRONISER”. Vérifier dans phpMyAdmin que les contacts sont insérés (sans doublons – gérer l’unicité ou envoyer seulement les nouveaux).

Test 3 – Recherche distante :
Saisir un nom existant ou un numéro. Les résultats doivent s’afficher dans le RecyclerView.

Test 4 – Recherche sans résultat :
Saisir une chaîne absente. Afficher “Aucun contact trouvé”.

Test 5 – Rotation d’écran :
La liste doit être conservée (via ViewModel si implémenté, ou rechargée depuis la base distante).


PARTIE 11 – BONNES PRATIQUES

- Utiliser ViewModel pour conserver les données lors des rotations.
- N’effectuer les opérations réseau que sur un thread background (Retrofit le fait déjà).
- Gérer les erreurs (timeout, serveur indisponible) avec des messages utilisateur.
- Ne pas logger les numéros de téléphone sensibles en production.
- Utiliser des requêtes préparées côté PHP.
- Ajouter un indicateur de progression (ProgressBar) pendant les requêtes.


PARTIE 12 – QUESTIONS DE COMPRÉHENSION (auto-évaluation)

1. Pourquoi Retrofit doit-il être utilisé avec un convertisseur JSON ?
2. Quelle est la différence entre enqueue() et execute() ?
3. Comment éviter les doublons lors de la synchronisation des contacts ?
4. Pourquoi ne peut-on pas faire un appel réseau sur le thread principal ?
5. Quel est le rôle du ContentResolver dans l’accès aux contacts ?


PARTIE 13 – EXTENSIONS POSSIBLES

- Ajouter la suppression d’un contact distant (DELETE).
- Mettre à jour un contact existant (PUT).
- Utiliser un ViewModel pour partager les données entre fragments.
- Ajouter un écran de détail.
- Implémenter la synchronisation automatique périodique (WorkManager).


CONCLUSION

Ce laboratoire a permis de construire une application Android complète communiquant avec une API REST personnalisée. Les compétences acquises incluent la lecture des contacts système, l’utilisation de Retrofit, la gestion des permissions, et l’affichage dynamique avec RecyclerView. Ces concepts sont essentiels pour toute application nécessitant une synchronisation entre un téléphone et un serveur distant.

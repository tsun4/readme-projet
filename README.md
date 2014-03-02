#Guide You - DEV#


##INTRODUCTION##


Ce projet est realisé dans le cadre de la validation des acquis de Developpement, Webdesign, et Gestion de projet.

***

##STRUCTURE DU PROJET##


Architecture du projet, MVC de type framework:

	|_Projet
		|-- configuration.php
		|-- index.php
		|-- /global				-->(template)
		|-- /libs				-->(Facebook,PHPMailer,lib core Guideyou)
		|-- /local				-->(fichier traduction)
		|-- /modules
			|-- /mod_404
				|-- /actions	--> (controleurs)
				|-- /img
				|-- /styles
				|-- /views		--> (views)
				|-- index.php 
			|-- /mod_dashboard
				|-- /actions	--> (controleurs)
				|-- /img
				|-- /styles
				|-- /js
				|-- /views		--> (views)
				|-- index.php 
			|_ /mod_home
				|-- /actions	--> (controleurs)
				|-- /img
				|-- /styles
				|-- /js
				|-- /views		--> (views)
				|-- index.php 
			|_ /mod_user
				|-- /actions	--> (controleurs)
				|-- /img
				|-- /styles
				|-- /js
				|-- /views		--> (views)
				|-- index.php 
			|_ /mod_offer
				|-- /actions	--> (controleurs)
				|-- /img
				|-- /styles
				|-- /js
				|-- /views		--> (views)
				|-- index.php 
		|_ README.md


##FONCTIONNEMENT FRAMEWORK##

Ce Framework MVC fonctionne de la facon suivante: 

	Racine Index.php (dispatcheur principal) ==> module index.php (dispatcheur de module) ==> action.php (controleur) ==> view.view.php (view)

###Dispatcheur Principal & configuration.php###

Le dispatcheur principal, _**index.php**_ en racine, requière le fichier de configuration _**configuration.php**_.
C'est le fichier qui instancie les classes **Guideyou** et **Translation**, et définit les variables d'environnement.

La classe **Guideyou**, est la classe qui contient les fonctionalités *"coeur"* du Framework.
Elle définit les méthodes d'appels, des views, des modèles, actions, modules, de connection BDD, de création de session, ...

La classe **Translation** est la classe qui s'occupe de traduire le contenu que l'on a définit grâce à _**get_text**_, dans les fichiers *.mo* du dossier *locale*.

Ce dispatcheur de premier niveau permet "*d'aguiller*" sur le bon module, par défaut, si aucun module n'est renseigner, la **Home** devient le module courant:

```php	
	<?php $module = (!empty($_GET['module'])) ? $_GET['module'] : 'home'; ?>
```

Il vérifie alors que le dispatcheur du module appelé existe bien, grâce à la fonction **getModule**, de la classe *Guideyou*:

```php
	<?php
		if(!$Guideyou->getModule($module)) {                      
		    // Redirection 404
		    header('Location: '.SERVERROOT.'404/'); 	
		}
	?>
```

###Dispatcheur de module###

Le dispatcheur de module, prend alors le relais.
Tout d'abord, les variables d'environnement sont initialisés (répertoire courant):


```php
	<?php
	// Définition du répertoire du module
		define('MODULEROOT', __DIR__.'/');
	// Définition de l'adresse du module par rapport au serveur
	$path = explode('modules', MODULEROOT);
		define('MODULEPATH', SERVERROOT.'modules'.$path[1]);	
	?>
```

Chaque module possède un dossier *actions*, qui contient les controleurs pour chaque fonctionnalité du module (ajout, suppression, ...).
Les actions sont ainsi recupèrés via url ("http://guideyou.co/module/action").

Si une action est definie, on recupère le fichier action.php, sinon main.php est inclu par défaut:

```php
	<?php 
		$action = (!empty($_GET['action'])) ? $_GET['action'] : 'main';
	?>
```

Si le fichier action existe, on l'inclue sinon redirection 404 (index.php du module 404), vérifié par la fonction **getAction()**, de la classe *Guideyou*:

```php
	<?php
		if(!$Guideyou->getAction($action)) {                      
		    //redirection 404
		    header('Location: '.SERVERROOT.'404/');
		}
	?>
```

###Actions###

Chaque action inclut la view qui lui correspond grâce à la fonction **getAction()** de la classe *Guideyou*.

```php
	<?php
		$Guideyou->getview('main', 
		array(
			'title' => 'Guideyou', 
			'offerList' => $offerList, 'userInformations' => $userInformations	// paramètres passées à la vue
		), 
		array(
			'css' => array(
				array('path' => 'global', 'file' => 'global.css'),  			// css global
				array('path' => 'module', 'file' => 'home.css')					// css du module
			), 
			'js' => array(
				array('path' => 'global', 'file' => 'jquery-2.0.3.min.js'),		// js global	
				array('path' => 'global', 'file' => 'jquery-ui-1.10.4.min.js'),	// js global
				array('path' => 'global', 'file' => 'global.js'),				// js global
				array('path' => 'module', 'file' => 'home.js')					// js du module
			)
		)
	);
	?>
```
De même, chaque module possède un dossier view qui contient un fichier main.view.php et un fichier action.view.php.
Les main.view.php sont les fichiers view de defaut qui contiennent du HTML.

Ils utilisent les dossiers CSS,IMG,JS commun (**/global**), et ceux de leur module si ils possèdent des demandes specifiques appélé par le paramètre **$files** de la fonction **getView()** de la classe **Guideyou**. 

***

##MODELISATION##


La modélisation est disponible dans le fichier .mwb.

Elle est composé de 6 tables:

- Utilisateur
- Prestation
- Commande
- Abus
- Calendrier
- Commentaires

La table *Utilisateur* répertorie tous les utilisateurs: Guide & client.
Il existe donc un champ qui permet d'identifier les guides (uti_gui_IS) & un champ permettant d'identifier les guides professionels
(uti_gui_PRO).

La table *Prestation* répertorie toutes les offres.
Elle est liée à la table Utilisateur, avec la foreign key pre_uti_ID, permettant d'identifier les appartenances de chaque prestation.

La table *Commande* répertorie toutes les commandes passées.
Elle est liée à la table Prestation, avec la foreign key com_pre_ID, permettant d'identifier quel prestation est concerné et donc le guide qui la propose.
De même, elle est liée à la table Utilisateur, via com_uti_ID, permettant d'identifier le client ayant passé la commande.

La table *Abus* répertorie les abus signalés par un client x sur une prestation y.
Elle est donc liée à la table Commande, via abu_Commande, afin de récupérer le client, le guide, et l'offre concernée.

La table *Calendrier* permet de stocker les jours de disponibilités d'un guide x sur une prestation Y.
Elle est donc liée à la table Prestation, via le champs cal_presta_ID.

La table *Commentaires* répertorie tous les commentaires d'une.
Elle est liée à la table Commande afin de savoir à quel numéro de commande et à quel prestation appartient le commentaire via le champs com_pre_ID (FK).

***

##FONCTIONNALITES##


Ici seront décrites les différentes fonctionnalités du site, par découpage de module:

###Dashboard###

Le Dashboard est une interface de gestion qu'utilise un guide pour créer, modifier ou supprimer une/des offres.

Resumé des fonctionalités:

- La création d'une offre
- La modification d'une offre
	- Le calendrier
	- Le save
	- Upload des images
- La suppression d'une offre

###CREATION D'UNE OFFRE###

La création des offres se déclenche lors du click sur le bouton *Call to action* Post an Offer.
Cette création ne fait intervenir en aucun cas du script php.

En effet, tant que le guide ne **save** pas une offre, elle reste un element du DOM uniquement.
Seul le save assure son enregistrement en base.

Le fonction crée alors dans le DOM, un nouvelle élement à la liste des offres (en la placant en tête), et initialise tous les paramètres de celles à null.

Il est donc de la responsabilité du guide de remplir tacitement tous ces champs. 
Il suffira en suite au guide de la sauvegarder pour la garder en memoire.

Par defaut, si le guide crée une offre, et quitte sa modification sans sauvegarder ses modifications, il est avertit via alerte que cela va faire disparaitre/supprimer cette offre (Element temporaire du DOM).

C'est le fichier **dashboard.js** qui se charge de cette fonctionnalité.

Lors de la création de l'offre, on peut renseigner des *Tags/Catégories* de celle-ci.
On utilise alors le js _**selectize**_.

Pour l'upload de photo, c'est _**Jquery_Fileupload**_ que l'on utilise afin de rendre l'expérience utilisateur plus vivante.

***

###MODIFICATION D'UNE OFFRE###

La modification des offres est declenchée (réelement effective) via le click sur le bouton **Save**.
La modification tolère le renvoie de champs vides (certains uniquement).

Il existe trois grosses sous-parties dans la modification des offres:

####LE CALENDRIER####

Le calendrier est génerer via code PHP.
Le modèle utilisé est _**Calendar.model.php**_.

Il se construit de la façon suivante:

La première étape est la verification afin de savoir si nous traitons le calendrier pour une offre crée (pas en BDD), ou une offre rapatriée depuis la BDD.

Dans le deuxième cas, la construction va nécessiter de recuperer les jours de disponibilités, le reste étant commun au deux cas.

***

#####CONSTRUCTION DU CALENDRIER#####

On recupère la valeur du mois (Entier transforme en string), de l'annee courante, les mois juxtaposés au mois courant (suivant et precèdant) pour la construction des liens, et le premier jour du mois (il faut déterminer si celui-ci est un mardi ou un dimanche par exemple).
 
La fonction **getFirstDay**, a pour rôle de définir qu'elle sera le premier jour du mois courant.
 
Si nous sommes dans le cas d'une offre importée de la BDD, il faut également récuperer les jours de disponibilités grâce à la fonction **$getOfferDates()**.
 
On crée alors le tableau, en affectant la class *empty* aux jours du mois precedant.
 
On vérifie si la date active est valide et appartient bien au mois courant, on ajoute alors le jour.

Dans le cas d'une offre déjà enregistrée, on regarde si le jour est un jour disponible, dans ce cas il hérite de la class *selected*. 


 
Le calendrier en lui même, fonctionne de la façon suivante:
La modification des jours peut utiliser deux fonctions: **selectday()** & **selectdays()**.

Ce sont 2 versions de la fonction, l'une traite une liste de jours (**selectdays()**), l'autre traite un jour singulié.

**Selectdays()** est la fonction qui traite le tableau temporaire des jours cliqués pour une nouvelle offre.

La fonction **selectday()** est utilisée, pour les offres contenues en BDD, via appel dans le code javascript du fichier **app.js**.

Le changement de mois, lui, est declenché lors du click sur les liens présentes en tête du calendrier (de part et d'autres du mois).

***

####SAUVEGARDE D'UNE OFFRE####

La Sauvegarde est la fonction js contenue dans le fichier **app.js** qui enregistre toutes les modifications/insertions des informations d'une offre.

Apres verifications des champs, leurs valeurs sont envoyées en BDD par les fonctions:

- **addOffer()**, dans le cas d'une nouvelle offre
- **updateOffer()**, pour les champs propre à une prestation (update de la table prestation)
- **updateImage()**, addImage, updateCoverImage, selectPictures & selectCover pour les champs images et cover de la table prestation


***

####UPLOAD IMAGES####

Géré en Javascript (Jquery), via les fichiers **app.js** et **jquery.fileupload.js** (drag & drop).
La fonction JS déclenche l'action **upload.php** qui s'occupe de sauvegarder l'image sur le serveur.

En fonction des paramètres, les fonctions **updateCoverImage()**, **updateImage()** ou **addImage()** sont utilisées.

Les fonctions **selectPictures()** & **selectCover()** sont des fonctions du modèle Dashboard, qui permettent au guide de pouvoir placer une image déjà uploadée en tant que cover.


***

###SUPPRESSION D'UNE OFFRE###

La suppression est declenchée lors du click sur le bouton de suppression (croix rouge en haut à droite).

Il y a deux cas de suppression: suppression nouvelle offre & suppression normal.

Dans le cas de la suppression nouvelle offre, aucun code PHP est appelé, on supprime juste les elements de l'offre du DOM après confirmation du guide.


Dans le cas de la suppression normal, on declenche l'action **deleteoffer.php** et l'offre est retiré du DOM (FadeOut).

En cas d'erreur, on retournera un message d'erreur, pour prévenir le guide.

***

##MODULE USER##

Le module *User* est le module qui se charge de la connexion ou de l'inscription, d'un client/guide.
Il se charge également de l'affichage de la page de profil d'un utilisateur.

Fonctionnalites:

- Connexion
	- normale
	- via Facebook
- Inscription
	- Verification de l'email

###CONNEXION###

La connexion sur le site GuideYou peut se dérouler de deux façons:

- l'utilisateur décide de se connecter via le formulaire avec les informations données lors de son inscription.
- ou il utilise le "login with Facebook" qui automatise le processus de login.

***

####CONNEXION NORMALE####

Une fois le formulaire remplit, l'action login.php se charge de logguer l'utilisateur:
Aprés avoir récuperer l'email et le password remplit dans le formulaire (valeur type POST), l'action utilise la méthode **loginUser($email, $password)** du modèle *Login* précedement instancié.

***

####CONNEXION FACEBOOK####

On utilise l'*API Facebook Login* javascript.
La librairie est stockée dans le répertoire **libs** en racine de l'architecture (*/libs/Facebook/base_facebook.php*,*/libs/Facebook/facebook.php* & */libs/Facebook/fb_ca_chain_bundle.crt*).

*Login_facebook.php* est le controleur qui se charge de récupérer les informations de l'utilisateur FB et, de loguer celui-ci sur le site en comparant les informations FB à celle en BDD.

Seul l'adresse email du compte FB, est ainsi récuperer.

On inscrit alors l'utilisateur avec les informations récupérés dans notre base de données si celui-ci n'existe pas en s'assurant de passer le champs *uti_FB* à 1, le mot de passe est généré automatiquement, son nom, prénom eux aussi renseigner.


***

###INSCRIPTION###

L'inscription est gérée par l'action **signup.php**, qui récupère les valeurs *POST* du formulaire d'inscription.
Elle utilise ensuite la fonction **signupUser()** du modèle Signup précedement instancié.

***

####VERIFICATION EMAIL####

Une fois le formulaire rempli, les informations sont stockées en base et une clé de vérification est générée.
Celle-ci est envoyé via mail à l'utilisateur avec un lien personnalisé renvoyant sur l'action **signup.php**,
mais avec une Url personnalisée de format : http://guideyou.co/user/cle,mail.

La cle est le chiffrage SHA-1 du lastname, firstname et du mail de l'utilisateur. Elle est comparée via la génération de la clé lors de la vérification email (on ne stocke pas la clé en BDD).


***

###PAGE PROFIL###

Cette page affiche les informations d'un utilisateur.
Elle est gérée par l'action **profile.php** qui utilise le modèle *User*.

Dans le cas où l'utilisateur est un guide, on affiche également la liste des offres de ce dernier.

Si le guide est de type pro, un petit badge le signalera en tant que tel.


***

###PAGE SETTINGS###

C'est la page qui sert à un guide ou un client, de modifier les informations personnelles de son compte.
Elle est constitué d'un formulaire.

Si l'utilisateur est un guide, alors cette page laisse apparaitre un second formulaire pour demander à passer PRO.
Cette demande déclenche un mail à l'admin du site et une notification.


##MODULE OFFER - FRONT CLIENT##

Le module *Offer* est le module qui se charge de récuperer les informations pour une offre particulière et de les afficher, afin de les présenter aux utilisateurs (clients).
Elle se charge egalement de la gestion des commentaires, de réserver une date pour cette offre, et de la recherche de guide ou d'offres.

###PAGE OFFRE###

On utilise le modèle *Dashboard* du module mod_dashboard.
Cette page utilise 3 fonctions du modèle cité plus haut:

- **getOfferInformations()**, qui récupère les informations d'une offre.
- **getNoteOffer()**, qui récupère/fabrique la note d'une offre
- **getGuideInformations()**, qui récupère les informations du guide pour cette offre.


La commande SQL de la fonction **getNoteOffer()**, récupère toutes les notes des commentaires de l'offre et fait la moyenne de ces valeurs.

###CHECKOUT - PANIER###

Guideyou utilise la technologie *ExpressCheckout* de Paypal, pour le payement de ses offres.
Le bouton *"Purchase this offer"* rajoute l'offre en variable de session et envoie sur l'action payment.php qui se charge d'afficher le panier.

Les informations du panier sont stockés **$_SESSION['produits']**.
On affiche alors les produits stockés sur les paniers tous en les récupérant dans le tableau **$params** qui est utilisé par la classe *Paypal*.

Cette classe, contient les informations de configuration, concernant Guideyou en tant que compte Paypal, qui permettent d'initialiser les variables pour dialoguer avec l'API de Paypal.

On utilisera la méthode *SetExpressCheckout* pour initialiser et envoyer les informations du payement à Paypal et *GetExpressCheckoutDetails* pour récupérer les informations du payement et les mettre en base de données.

Les données du client sont ainsi directement renseigner sur Paypal et non sur GuideYou, évitant ainsi toutes étapes de payement inutiles puisque ces informations peuvent être récupérer si la commande est validé.

Une fois, celle-ci validé, on récupère donc les informations de la commande, de l'utilisateur.
Si la commande, possède plusieurs produits, ils pourront être regroupés par le numéro de la commande (*TRANSACTION_ID*), fournie par Paypal, mais apparaissent en base come des commandes uniques.

***

###PAGE DE RECHERCHE###

Action **search.php**. Modèle utilisé: *Offer.model.php*
2 types de retour:

- soit le formulaire est rempli, et on cherche des résultats cohérent avec cette recherche, grâce à la fonction **getOffersSearch()**.

- soit , on informe l'utilisateur qu'il n'y a pas de résultats pour sa recherche parmis notre Base de données, et on lui renvoie les 5 dernières offres, via la méthode **getLastOffersSearch()**.

On peut filtrer les résultats via la barre d'outils et en appuyant sur le bouton *Update Search* pour rafraichir les résultats.

***

##BACKOFFICE - Panel d'administration##

Le backoffice de Guideyou possède 3 onglets:
- Commandes
- Utilisateurs
- Prestations

L'onglet commande affiche les commandes sous forme de tableaux selon leur état de traitement (abus, versé, en cours, ...).

L'onglet Utilisateur affiche les utilisateurs, les guides, et guides PRO, tout en permettant de les supprimer.

L'onglet Prestation affiche les prestations disponibles sur le site, leur prix et leur catégories.

Le backoffice possède également un système de notification qui lui permet de voir, et traité les demandes des guides qui souhaitent passés Guide Pro.

***

##FAIBLESSES##

Ce projet étant dans le cadre scolaire, on excusera les faiblesses ci-dessous, et le manque de rigueur dans certaines parties.

- Notifications du backoffice peu efficaces, nécessite un réel travail à part en tiers, avec une table dans la base.
- Journalisation des commandes en logs, action en Backoffice (potentiel discard changes).
- Commentaires non surveillés efficacement via un système anti-spam, et un dictionnaire de mots interdit.
- Système de commentaires et de notes non implémenté

***

##AMELIORATIONS##

###Global###

- Connexion plus sécurisée via cryptage plus haut (_**Sha-1**_ et sel).
- Calcul des performances du framework et optimisation.
- Sécurisation du code.
- Passage vers un moyen de payement plus traditionnel

###Dashboard###

- Implémentation du système de disponibilités avancés avec quotas de réservations et de personnes + horaires 
- Statistiques de l'offre visible par le guide (Total de visite)
- Accueil du dashbord plus général, avec des notifications, rappel, liaison API avec application mobile pour les guides (version business).
- Ajout de l'onglet payement (compte paypal, transaction effectué, en cours, ...)

###Offres###

- Filtrage ajax des offres lors de la recherche
- Filtrage actif en fonction d'informations personelles (les offres que nous vous recommandons, les offres premium/payés pour être en top liste).
- Calendrier pour selectionner le ou les dates désirées.
- Pagination Ajax des résultats ou scroll infiny
- Tri des offres
- Système de notation et de commentaire à implémenter

###Profile###

- Liaison avec autres API (google, linkedin, facebook,...) pour dynamisation social de la page.
- Système de vérification anti-Spam pour les commentaires, même si il est necessaire de recevoir un token pour voter. Implémentation d'un dictionnaire liste noire de mots.

###Backoffice###

- Implémentation de statistiques
- Système d'onglets Ajax
- Pagination tableau Ajax
- Améliorations/ajout divers au système de notification
- Implémentation d'un système de pseudo comptabilité
- Journalisation des actions du backoffice (Delete, ... + Discard changes)

###Application Mobile###

- Création d'une API avec renvoie de donnée JSON.
- Intégration du Template joint dans le dossier Design.

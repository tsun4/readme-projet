Guide You - DEV
===============

INTRODUCTION
------------

Ce projet est realisé dans le cadre de la validation des acquis de Developpement, Webdesign, et Gestion de projet.

***

STRUCTURE DU PROJET
-------------------

Architecture du projet, MVC Custom:

	|_Projet
		|_ /CSS
		|_ /IMG
		|_ /JS
		|_ /Modules
			|_ /mod_404
				|_ /Actions	--> (controlleurs)
					|_ action1.php
					|_ action2.php
					|_ ...
				|_ /img
				|_ /styles
				|_ /views	--> (views)
					|_ view1.view.php
					|_ view2.view.php
					|_ ...
				|_ index.php
			|_ /mod_dashboard
				|_ /Actions --> (controlleurs)
					|_ action1.php
					|_ action2.php
					|_ ...
				|_ /img
				|_ /js
				|_ /models  --> (modeles)
					|_ model1.model.php
					|_ model2.model.php
					|_ ...
				|_ /styles
				|_ /views	--> (views)
					|_ view1.view.php
					|_ view2.view.php
					|_ ...
				|_ index.php
			|_ /mod_home
				|_ /Actions --> (controlleurs)
					|_ action1.php
					|_ action2.php
					|_ ...
				|_ /img
				|_ /js
				|_ /styles
				|_ /views	--> (views)
					|_ view1.view.php
					|_ view2.view.php
					|_ ...
				|_ index.php
			|_ /mod_user
				|_ /Actions --> (controlleurs)
					|_ action1.php
					|_ action2.php
					|_ ...
				|_ /models  --> (modeles)
					|_ model1.model.php
					|_ model2.model.php
					|_ ...
				|_ /views	--> (views)
					|_ view1.view.php
					|_ view2.view.php
					|_ ...
				|_ index.php
			|_ /mod_offer
				|_ /Actions --> (controlleurs)
					|_ action1.php
					|_ action2.php
					|_ ...
				|_ /models  --> (modeles)
					|_ model1.model.php
					|_ model2.model.php
					|_ ...
				|_ /views	--> (views)
					|_ view1.view.php
					|_ view2.view.php
					|_ ...
		|_ configuration.php
		|_ index.php
		|_ README.md


FONCTIONNEMENT MVC
------------------
Ce MVC fonctionne de la facon suivante: 

	Racine Index.php > module index.php > action.php > view.view.php

Le fichier **index.php** en racine, requière le fichier de configuration **configuration.php**.
Si un des modules est appelé, on le recupère dans la variable $module, sinon on affiche la home par default:
	
	<?php $module = (!empty($_GET['module'])) ? $_GET['module'] : 'home'; ?>

On verifie alors si le fichier **index.php** du dossier module appelé existe:

	<?php if(is_file(PATHROOT.'/modules/mod_'.$module.'/index.php')) ?>

On inclut alors ce fichier:

	<?php include(PATHROOT.'/modules/mod_'.$module.'/index.php'); ?>

Fichier **index.php** du repertoire module
On définit tout d'abord les variables d'environnement (répertoire courant):

	<?php
	define('MODULEROOT', __DIR__.'/');
	$path = explode('modules', MODULEROOT);
	define('MODULEPATH', SERVERROOT.'/modules'.$path[1]);?>

Chaque module possède un dossier actions, chaque fichier de ce dossier contient les controlleurs pour chaque fonctionnalité du module (ajout, suppression, ...).
Les actions sont recupèrés via url.
Si une action est definie, on recupère le fichier action.php, sinon main.php est inclu:

	<?php $action = (!empty($_GET['action'])) ? __DIR__.'/actions/'.$_GET['action'].'.php' : __DIR__.'/actions/main.php'; ?>

Si le fichier action existe, on l'inclue sinon redirection 404 (index.php du module 404):

	<?php is_file($action) ? include($action) : include(PATHROOT.'/modules/mod_404/index.php'); ?>

Chaque action inclut (require) la view qui lui correspond.
De même, chaque module possède un dossier view qui contient un fichier main.view.php et un fichier action.view.php.
Ces views dites annexes sont la plupart du temps un encodage JSON de la variable qui est retourné par le fichier action:

	<?php echo json_encode($offer); ?>

Les main.view.php sont les fichiers view de defaut qui contiennent du HTML.
Ils utilisent les dossiers CSS,IMG,JS commun (racine du site), et ceux de leur module si ils possèdent des demandes specifiques. 

***

MODELISATION
------------

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
Elle est liée à la table prestation afin de savoir à quel prestation appartient le commentaire via le champs com_pre_ID (FK).

***

FONCTIONNALITES
---------------

Ici seront decrites les differentes fonctionnalités du site, par decoupage de module:

**Dashboard**

Le Dashboard est une interface de gestion qu'utilise un guide pour creer, modifier ou supprimer une/des offres.

Resumé des fonctionalités:

- La création d'une offre
- La modification d'une offre
	- Le calendrier
	- Le save
	- Upload des images
- La suppression d'une offre

###CREATION D'UNE OFFRE###

La création des offres se declenche lors du click sur le bouton *Call to action* Post an Offer.
Cette création ne fait intervenir en aucun cas du script php.

En effet, tant que le guide ne **save** pas une offre, elle reste un element du DOM uniquement.
Seul le save assure son enregistrement en base.

Le fonction crée alors dans le DOM, un nouvelle élement à la liste des offres (en la placant en tête), et initialise tous les paramètres de celles à null.

Il est donc de la responsabilité du guide de remplir tacitement tous ces champs. 
Il suffira en suite au guide de la sauvegarder pour la garder en memoire.

Par defaut, si le guide crée une offre, et quitte sa modification sans sauvegarder ses modifications, il est avertit via alerte que cela va faire disparaitre/supprimer cette offre (Element temporaire du DOM).

C'est le fichier **app.js** qui se charge de cette fonctionnalité.

***

###MODIFICATION D'UNE OFFRE###

La modification des offres est declenchée (réelement effective) via le click sur le bouton **Save**.
La modification tolère le renvoie de champs vides (certains uniquement).

Il existe trois grosses sous-parties dans la modification des offres:

####LE CALENDRIER####

Le calendrier est génerer via du Code PHP.
Le modèle utilisé est *Calendar.model.php*.

Il se construit de la façon suivante:

La première étape est la verification afin de savoir si nous traitons le calendrier pour une offre crée (pas en BDD), ou une offre rapatriée depuis la BDD.

Dans le deuxième cas, la construction va necessiter de recuperer les jours de disponibilités, le reste étant commun au deux cas.

***

#####CONSTRUCTION DU CALENDRIER#####

On recupère la valeur du mois (Entier transforme en string), de l'annee courante, les mois juxtaposés au mois courant (suivant et precèdant) pour la construction des liens, et le premier jour du mois (il faut déterminer si celui-ci est un mardi ou un dimanche par exemple)

```php
<?php
$results = array();
// On récupère la variable POST du mois et on la transforme en sa valeur string (2->Février)
$results['month'] = $_POST['oMonth'];
$results['month_letter'] = $calendar->getMonth($month);
$results['year'] = $year;
// on récupère le mois précedant et le suivant
$results['next_date'] = $calendar->getNext($month, $year, 1);
$results['previous_date'] = $calendar->getNext($month, $year, -1);

$tab_days = array();
// on récupère le premier jour du mois (lundi, mardi, ...)
$num_day = $calendar->getFirstDay($month, $year);
?>
```

 
Cette fonction **getFirstDay** est construite de la maniere suivante:

```php
<?php
public function getFirstDay($month, $year){
	$tmstp = mktime(0,0,0,$month,1,$year);
	$day = date("w",$tmstp);
	$tab_day = array(0=>7,1=>1,2=>2,3=>3,4=>4,5=>5,6=>6);

	return $tab_day[$day];
}?>
```
 
Si nous sommes dans le cas d'une offre importée de la BDD, il faut également récuperer les jours de disponibilités.

```php
<?php
// Si l'id de l'offre n'est pas null
if($_POST['oID'] != -1) {
	// On récupère les jours de disponibilité de l'offre
	$dates = $calendar->getOfferDates($_POST['oID']);
	// Chaque date est ajouté au tableau $arr_dates au format J/M/A
	foreach($dates AS $date) {
		$arr_dates[] = $date['cal_JOUR'].'/'.$date['cal_MOIS'].'/'.$date['cal_ANNEE'];
	}
}
?>
```
 
On crée alors le tableau, en affectant la class *empty* aux jours du mois precedant.

```php
<?php
if($count < $num_day) {
		$tab_days[$count]='<td class="empty"></td>';
	}
?>
```

 
On vérifie si la date active est valide et appartient bien au mois courant, on ajoute alors le jour.

Dans le cas d'une offre déjà enregistrée, on regarde si le jour est un jour disponible, dans ce cas il hérite de la class *selected*. 

```php
<?php
if(checkdate($month, $num_current_day, $year)) {
	// Si la date est contenu dans le tableau des jours de dispo, on rajoute class active au td
	$active = (in_array($num_current_day.'/'.(int) $month.'/'.$year, $arr_dates)) ? ' class="active"' : null;
	$tab_days[$count]='<td'.$active.' id="'.$num_current_day.'-'.(int) $month.'-'.$year.'">'.$num_current_day.'</td>';
	$num_current_day++;
}
else {
	// si on est dans les jours du mois suivant
	$tab_days[$count]='<td class="empty"></td>';
}?>
```


 
Le calendrier en lui même, fonctionne de la façon suivante:
La modification des jours peut utiliser deux fonctions: **selectday()** & **selectdays()**.

Ce sont 2 versions de la fonction, l'une traite une liste de jours (**selectdays()**), l'autre traite un jour singulié.

**Selectdays()** est la fonction qui traite le tableau temporaire des jours cliqués pour une nouvelle offre.

La fonction **selectday()** est utilise pour les offres BDD via appel dans le code javascript du fichier **app.js**:

```javascript
$.post("selectday/", { oID: oID, oDay: idD[0], oMonth: idD[1], oYear: idD[2], oAction: 1 }, function(data) {
	// On parse les données json renvoyées
	var results = $.parseJSON(data);
	// Si tout est OK, on ajoute la classe active
	if(results.status == 1) { tElt.addClass('active'); }
	// Sinon, on affiche un message d'erreur
	else { 
		$('.message-response').removeClass().addClass('message-response error').html('An error occured during selectioning this day !').fadeIn(500);
		setTimeout(function() {
			$('.message-response').fadeOut(500);
		}, 5000);
	}
});
``` 

Le changement de mois est declenché lors du click sur les liens:

```javascript
$.post("getmonthcalendar/", { oID: oID, oMonth: oMonth, oYear: oYear }, function(data) {
	// On parse les données json renvoyées
	var results = $.parseJSON(data);
	// On affiche le MOIS et l'ANNÉE
	ptitle.html(results.month_letter+' '+results.year+'<div style="display: none;" class="currentMonth">'+results.month+'</div><div style="display: none;" class="currentYear">'+results.year+'</div>').fadeIn(500);
	// On modifie le lien du mois précédent
	$('.calendar .table-title a.left-arrow').attr('id', 'pm-'+results.previous_date['month']+'-'+results.previous_date['year']);
	// On modifie le lien du mois suivant
	$('.calendar .table-title a.right-arrow').attr('id', 'nm-'+results.next_date['month']+'-'+results.next_date['year']);
	// On initialise days
	var days = '';
	// On initialise le compteur à 1
	var count = 1;
	// On parcourt le tableau tab_days renvoyé
	for(var key in results.tab_days) {
		// Si on est sur la première ligne
		if(count == 1) { days += '<tr class="days">'; days += results.tab_days[key]; }
		// Si on est à la fin d'une ligne
		else if(count == 8) { days += '</tr><tr class="days">'; days += results.tab_days[key]; count = 1; }
		// Pour tout le reste
		else { days += results.tab_days[key]; }
		// On incrèmente le compteur
		count++;
	}
}
``` 

***

####SAUVEGARDE D'UNE OFFRE####

La Sauvegarde est la fonction js contenue dans le fichier **app.js** qui enregistre toutes les modifications/insertions des informations d'une offre.

Apres verifications des champs, leurs valeurs sont envoyées en BDD par les fonctions:

* **addOffer()**, dans le cas d'une nouvelle offre
* **updateOffer()**, pour les champs propre à une prestation (update de la table prestation)
* **updateImage()**, addImage, updateCoverImage, selectPictures & selectCover pour les champs images et cover de la table prestation


***

####UPLOAD IMAGES####

Géré en Javascript (Jquery), via les fichiers **app.js** et **jquery.fileupload.js** (drag & drop).
La fonction JS déclenche l'action **upload.php** qui s'occupe de sauvegarder l'image sur le serveur.

En fonction des paramètres, les fonctions **updateCoverImage()**, **updateImage()** ou **addImage()** sont utilisées.

Les fonctions **selectPictures()** & **selectCover()** sont des fonctions du modèle Dashboard, qui permettent au guide de pouvoir placer une image déjà uploadée en tant que cover.

```sql
UPDATE prestation set pre_COVER = :oCover WHERE pre_ID = :oID
```

***

###SUPPRESSION D'UNE OFFRE###

La suppression est declenchée lors du click sur le bouton de suppression (croix rouge en haut à droite).

Il y a deux cas de suppression: suppression nouvelle offre & suppression normal.

Dans le cas de la suppression nouvelle offre, aucun code PHP est appele, on supprime juste les elements de l'offre du DOM après confirmation du guide.

```javascript
// Si on est dans une nouvelle offre
if(newOffer == 1) {
	// On réinitialise le tableau des jours sélectionnés
	tab_days_selected = new Array();
	// On affiche une alerte demandant à l'utilisateur s'il est sur ou non
	var c = confirm('Would you really remove this new offer ? All changes will be lost !');
	// Si la réponse est OK
	if(c == true) {
		// On supprime les éléments correspondant à la nouvelle offre dans la liste
		$('.offers-list ul li:first-child').remove();
		$('.offers-list ul .sep:first-child').remove();
		// On cache les divs container-offer et options
		$('.container-offer').fadeOut(500);
		$('.options').fadeOut(500);
		// Après 500ms, on affiche la div welcome-offer
		setTimeout(function() {
			$('.welcome-offer').fadeIn(500);
		}, 500);
		// On dit qu'on est plus dans une nouvelle offre
		newOffer = 0;
	}
```

Dans le cas de la suppression normal, on declenche l'action **deleteoffer.php** et l'offre est retiré du DOM (FadeOut).

```javascript
$.post("deleteoffer/", { offID: offID }, function(data)
```

En cas d'erreur, on retournera un message d'erreur.

```javascript
// On affiche un message d'erreur
$('.message-response').removeClass().addClass('message-response error').html('An error occured during the deleting !').fadeIn(500);
setTimeout(function() {
	$('.message-response').fadeOut(500);
}, 5000);
```

***

##CONNEXION/INSCRIPTION - MODULE USER##

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

Une fois le formulaire remplit, l'action login.php se charge de logguer l'utilisateur.

```php
<?php
// Instaciation du modèle Login
$login = new Login();
// Récupération des champs du formulaire logins
$email = $_POST['email'];
$password = $_POST['password'];

$results = $login->loginUser($email, $password);
// on renvoie l'encode json du résultat $resultss
require(MODULEROOT.'views/login.view.php');
?>
```
***

####CONNEXION FACEBOOK####

On utilise l'API facebook login en js.
Cf. tutorial developpers.facebook.com

***

###INSCRIPTION###

L'inscription est gérée par l'action **signup.php**, qui récupère les valeurs *POST* du formulaire d'inscription.
Elle utilise ensuite la fonction **signupUser()** du modèle Signup précedement instancié.

```php
<?php
// Instaciation du modèle Signup
$signup = new Signup();
// Récupération des champs du formulaire signup
$lastname = $_POST['lastname'];
$firstname = $_POST['firstname'];
$mailaddress = $_POST['mailaddress'];
$upassword = $_POST['upassword'];

$is_guide = (isset($_POST['beaguide'])) ? 1 : 0;

$results = $signup->signupUser($lastname, $firstname, $mailaddress, $upassword, $is_guide);
?>
```
***

####VERIFICATION EMAIL

Une fois le formulaire rempli, les informations sont stockées en base et une clé de vérification est générée.
Celle-ci est envoyé via mail à l'utilisateur avec un lien personnalisé renvoyant sur l'action **signup.php**,
mais avec une Url personnalisée de format : http://guideyou.co/user/cle,mail.

La cle est le chiffrages SHA-1 du lastname, firstname et du mail de l'utilisateur. Elle est comparée via la génération de la clé lors de la vérification email (on ne stocke pas la clé en BDD).


```php
<?php
public function checkKey($mail, $key) {
	global $connexion;

	$signup_check = $connexion->prepare('select uti_ID, uti_NOM, uti_PRENOM from utilisateur where uti_MAIL = :mail');
	$signup_check->bindParam(':mail', $mail, PDO::PARAM_STR);
	$signup_check->execute();
	$r = $signup_check->fetch();

	$results = array();

	if($key == sha1($r['uti_NOM'].$r['uti_PRENOM'].$mail)) {
		$signup_a = $connexion->prepare('update utilisateur set uti_ACTIVATION = 1 where uti_MAIL = :mail');
		$signup_a->bindParam(':mail', $mail, PDO::PARAM_STR);
		$signup_a->execute();

		$_SESSION['uID'] = 'uti_'.$r['uti_ID'];

		header('Location: '.SERVERROOT);
	}
}
?>
```

***

###PAGE PROFIL###

Cette page affiche les informations d'un utilisateur.
Elle est gérée par l'action **profile.php** qui utilise le modèle *User*.

```php
<?php
// Instanciation du modèle
$user = new User();
// On récupère les infos de l'utilisateur
$userInformations = $user->getInformationsUser($p);
?>
```

Dans le cas où l'utilisateur est un guide, on affiche également la liste des offres de ce dernier.

```php
<?php
// Si l'utilisateur est un guide
if($userInformations['uti_gui_IS'] == 1) {
	$dashboard = new Dashboard();
	// On récupère les offres du guide
	$offers = $dashboard->getGuideOffers($p);
	// On récupère tous les commentaires sur ce guide
	$commentaires = $dashboard->getCommentsUser($p);
	// Pour chaque offre, on récupère également sa note.
	foreach($offers AS $offer) {
		$note_temp = $dashboard->getNoteOffer($offer['pre_ID'], 1);
		$note[$offer['pre_ID']] = (isset($note_temp)) ? $note_temp : null;
	}
}
?>
```

***

##MODULE OFFER - FRONT CLIENT##

Le module *Offer* est le module qui se charge de récuperer les informations pour une offre particulière et de les afficher, afin de les présenter aux utilisateurs (clients).
Elle se charge egalement de la gestion des commentaires, de réserver une date pour cette offre, et de la recherche de guide ou d'offres.

###PAGE OFFRE###

On utilise le modèle *Dashboard* du module mod_dashboard.
Cette page utilise 3 fonctions du modèle cité plus haut:

- **getOfferInformations()**, qui récupère les informations d'une offre.
- **getNoteOffer()**, qui récupère/fabrique la note d'une offre
- **getGuideInformations()**, qui récupère les informations du guide pour cette offre.

```php
<?php
// Instanciation du modèle
$dashboard = new Dashboard();
// On récupère les informations de l'offre d'id $p
$offerInformations = $dashboard->getOfferInformations($p);
// On récupère la note de l'offre
$noteOffer = $dashboard->getNoteOffer($p, 0);
// On récupère les informations du guide de cette offre.
$guideInformations = $dashboard->getGuideInformations($offerInformations['pre_uti_ID']);
?>
```

La commande SQL de la fonction **getNoteOffer()**, récupère toutes les notes des commentaires de l'offre et fait la moyenne de ces valeurs :

```sql
SELECT TRIM(TRAILING ".0" FROM ROUND(AVG(com_NOTE),:dec)) from commentaire where com_pre_ID = :oID');
```
***

###PAGE DE RECHERCHE###

Action **search.php**. Modèle utilisé: *Offer.model.php*
2 types de retour:

- soit le formulaire est rempli, et on affiche les résultats de la recherche

```php
<?php
//INSTANCIATION DU MODELE
$offer = new Offer();
// On récupère la liste des offres qui ressemble à la recherche demandée
$offerList = $offer->getOffersSearch($_POST['search_offer']);
// Si aucun résultat ne correspond, on affiche les dernières offres
if(count($offerList) == 0) {
	$offerList = $offer->getLastOffersSearch();}
?>
```

où la fonction **getOffersSearch()** est une fonction recherche de la forme :

```sql
select * from prestation, utilisateur where pre_uti_ID = uti_ID AND (
	pre_TITRE like :search
	OR
	pre_DESCRIPTION like :search
	OR
	pre_LIEU like :search
	) 
	order by pre_ID desc'
```

- soit on renvoie les dernières offres:

```php
<?php $offerList = $offer->getLastOffersSearch(); ?>
```

où la fonction **getLastOffersSearch()** est de la forme:

```sql
select * from prestation, utilisateur where pre_uti_ID = uti_ID order by pre_ID desc limit 5
```
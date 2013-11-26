title="AngularJS : i18n as a service"
tags="angularjs angular,javascript,framework,i18n"
author="dhoko"
description="i18n et AngularJS, comment rendre l'internationalisation simple et transparente."

==POST==

i18n et AngularJS, une vaste question. On trouve beaucoup de scripts et pas un seul DRY, j'ai donc fait le mien. Voilà pourquoi.

## Au commencement, l'i18n sous Angular

C'est le néant, rien prévu pour dans Angular. Et nous pauvres développeurs comment devons-nous faire ?

### Le résultat au niveau de nos templates

```
<!-- Version de base Angular -->
<h1>{{title}}<h1>

<!-- Version qu'on rencontre souvent -->
<h1>{{title | i18n}}<h1>

<!-- Version non logique  -->
<h1 data-i18n="title"><h1>
```

#### Version avec un filter

C'est une solution logique si l'on suit le fonctionnement de ses templates, cependant elle possède quelques désavantages :

- Mettre systématiquement cette directive
- Viable si on n'a que 2/3 pages. Trop lourd pour une SPA
- DRY ??? non

### Version avec une directive

Celle-ci n'est pas logique du tout. On enlève du template son affichage et on traduit un peu à [la rache](http://byatoo.com/la-rache/index.php?p=page&name=presentation&id=2&PHPSESSID=cd8hoks49dberr256sntlmpn27 "la rache"). Elle cumule aux défauts de la solution avec le fiter, la lourdeur.

## i18n as a service

Quoi de plus logique que de se dire, i18n est actif oui ou non ? Si oui ok, mais ça ne change pas la logique de mon application. Elle ne doit pas jouer sur nos templates, c'est au niveau de l'app que se joue tout.

### Step 1 : Notre service

Je bosse sur une SPA dont les données arrivent via une API REST, je me suis basé sur ça pour mon internationalisation. Je contacte mon API et elle me renvoit un JSON clé => valeur. Simple et super flexible.

#### Avant de commencer, le routing

Dans `$routeProvider` ou `$stateProvider` d'[UI-router](https://github.com/angular-ui/ui-router "UI-Router for Nested Routing by the AngularUI Team!") on retrouve la fameuse clé `resolve` [cf documentation](https://github.com/angular-ui/ui-router/wiki/Quick-Reference#resolve "About resolve - UI router AngularJS"). Celle-ci possède quelques avantages :
- Exécution avant le controller courant
- Permet de gérer des dépendances
- Résoud automatiquement les promesses

Partant de ça, on peut faire beaucoup de choses.

#### Notre service

```
'use strict';
app.factory('localize', ['$http', function ($http) {
    return {
        get: function(myCurrentView) {
          return $http.get('/i18n/'+myCurrentView, {cache: true});
        }
    };
}]);
```

Ce petit service **localize** est tout simple, il récupère notre traduction et la met dans un cache. Son API est simpliste : `localize.get(myCurrentView)`.

#### Le service en fonctionnement dans le routeur

```
.state('home', {
	url : '/',
	templateUrl: 'home.html',
	controller : 'homeController',
	resolve : {
		i18n : ['localize',function (localize) {
			return localize.get('home');
		}]
	}
})
```

Ici notre service est injecté dans le `resolve`. Celui-ci va donc aller chercher dans mon API la traduction pour la vue `home`.

Ainsi dans mon controller je peux utiliser la traduction en injectant i18n, la clé définie dans mon resolve.

#### Dans mon controller

```
'use strict';

app.controller("homeController", [
	'$scope', 'i18n', 
	function ($scope, i18n) {
    $scope = i18n.data
}]);
```

La solution est ici un peu brutale ( *personnellement je préfère mettre mes traductions dans $scope.panel* ), on injectera rarement les données ainsi.
Comme on peut le voir, nous n'avons non plus une promesse, mais directement son retour. Ainsi `i18n.data` contient bien mon json de traduction ( *Il n'est pas sous forme de string* hein :)).

#### Usage avancé dans le routeur

```
.state('home', {
	url : '/',
	templateUrl: 'home.html',
	controller : 'homeController',
	resolve : {
		i18n : ['localize',function (localize) {
			return localize.get('home');
		}]
	},
	onEnter : function (i18n, PageTitleService) {
		PageTitleService.set(i18n.data['title']);
	}
})
```

Ici je peux aussi utiliser i18n dans `onEnter()` vu que celui-ci s'exécute après mon resolve.

## Conclusion

Voilà désormais, que je fasse toute ma webapp, ou seulement une partie de celle-ci, je ne touche plus à mes templates. Tout est transparent. 

*On peut aussi ne pas faire des appels vers une API, mais aller charger des fichiers JSON*.

### I18n avec Angular, quelques liens

- [Internazionalization (i18n) with AngularJS](http://blog.brunoscopelliti.com/internazionalization-i18n-with-angularjs)
- [angular-translate](https://github.com/PascalPrecht/angular-translate)
- [ng-i18n - Internationalization module for angular.js](https://github.com/gertn/ng-i18n)
- [AngularJS and i18n](https://coderwall.com/p/uyrtpq)
- [Traduction des libellés dans les vues AngularJS](http://www.frangular.com/2012/12/traduction-des-libelles-dans-les-vues-angularjs.html)

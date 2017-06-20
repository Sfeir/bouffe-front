---
layout: post
title:  "Bouffe front de mai"
date:   2017-05-24
categories: prez bouffe front
---

## CSRF dans les frameworks

### L'attaque CSRF, kézako ?

C'est une des attaques les plus répandues d'après l'[OWASP](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)).

Ce monsieur là l'explique parfaitement:

<iframe width="560" height="315" src="https://www.youtube.com/embed/vRBihr41JTo" frameborder="0" allowfullscreen></iframe>

TL;DR quand t'es connecté sur le site de ta banque mal protégée, rien n'empêche une autre page (fourbe !) de faire un post sur l'api pour virer un montant de ton compte à quelq'un d'autre.

On va voir un peu comment on peut se protéger de ça côté front, avec des exemples dans différents frameworks. Il faut bien garder en tête que la partie back de notre application doit implémenter les mécanismes pour fournir et valider les tokens de protection !

### C'est parti !

Voir le [dépot avec les exemples](https://gitlab.com/SiegfriedEhret/csrf-examples) (les codes présentés ici montrent le strict minimum pour l'activation de CSRF, ils ne sont pas fonctionnels !)

#### Angular*

Dans Angular.JS, c'est cool: il faut jouer avec la [configuration de $http](https://docs.angularjs.org/api/ng/service/$http).

```javascript
const csrfApp = angular.module('csrfApp', []);
csrfApp.config(function($httpProvider) {
  $httpProvider.defaults.xsrfCookieName = 'CSRF-TOKEN';
  $httpProvider.defaults.xsrfHeaderName = 'X-CSRF-Token';
});
```

Pour Angular.Not.JS, c'est tout pareil mais en différent !

```javascript
// On crée le provider pour activer la protection

const csrfProvider = {
  provide: ng.http.XSRFStrategy,
  useValue: new ng.http.CookieXSRFStrategy('CSRF-TOKEN', 'X-CSRF-Token')
};

// On init notre app avec le provider

const AppModule = ng.core
  .NgModule({
    imports: [
      ng.platformBrowser.BrowserModule,
      ng.http.HttpModule
    ],
    declarations: [AppComponent],
    providers: [csrfProvider],
    bootstrap: [AppComponent]
  })
  .Class({
    constructor: function() {
    }
  });

// Et on active la protextion CSRF au sein de notre composant:

const AppComponent = ng.core
  .Component({
    selector: 'csrf-example',
    providers: [ng.http.XSRFStrategy],
    template: `<button type="button" (click)="doPost()">Post</button>`
  })
...
```

#### BackboneJS

Il faut overrider la méthode sync pour passer les headers qui vont bien automatiquement:

```javascript
const originalSync = Backbone.sync;
Backbone.sync = function(method, model, options) {
  options.headers = options.headers || {};

  // Comment these lines to disable CSRF
  _.extend(options.headers, {
    'X-CSRF-Token': token
  });

  return originalSync.call(model, method, model, options);
};
```
#### JavaScript 

##### Avec Axios

[Axios](https://www.npmjs.com/package/axios) est une super lib pour les appels HTTP pour le browser et Node.JS. On a le même genre d'options que pour Angular*.

```javascript
const options = {
    xsrfCookieName: 'CSRF-TOKEN',
    xsrfHeaderName: 'X-CSRF-Token',
};

axios
  .post('https://my.api/uri', {}, options)
  .then(...)
  .catch(...);
```

##### Avec jQuery

Baway [jQuery](http://jquery.com/) ça existe encore. `$.ajax` a tout ce qu'il faut pour pimper nos requêtes.

```javascript
$.ajax({
  method: 'POST',
  url: URL,
  data: {
    ...
  },
  headers: {
    'X-CSRF-Token': token
  });
```

##### Avec Fetch

Pour du JS avec fetch, gros gros fail. À l'aide !

#### Ember

C'est comme pour JavaScript avec jQuery, on peut modifier l'objet passé à `Ember.$.ajax`.

#### React

React c'est une lib pour la vue, les appels API ne sont pas sont problème. Il faut donc utiliser les solutions de la partie JavaScript.

#### Vue

Sans douleur avec [vue-resource](https://github.com/pagekit/vue-resource) en plaçant un intercepteur:

```javascript
Vue.http.interceptors.push(function(request, next) {
  request.headers.set('X-CSRF-Token', token);
  next();
});

```

### Mais c'est old !

On a les «[Samesite cookies](https://tools.ietf.org/html/draft-west-first-party-cookies)» !

> Same-site cookies (née "First-Party-Only" (née "First-Party")) allow servers to mitigate the risk of CSRF and information leakage attacks by asserting that a particular cookie should only be sent with requests initiated from the same registrable domain.

Oui, mais en fait non. Voir sur [Caniuse.com](http://caniuse.com/#search=samesite): en gros c'est cool que pour les navigateurs blink (Chrome, Opera, Android...).

Donc non, on n'est pas encore débarrassé des attaques CSRF !

---
title:  "e2e tests using Nightmare/Nightwatch.js!"
date:   2017-04-24
categories: prez bouffe front
link: http://slides.com/alexandrebarbier/deck-5-6#/
---

## Tests e2e avec Nightmare/Nightwatch.js
Dans cette présentation je me suis intéressé au concept de **tests e2e** ou tests end to end, c'est à dire des test d'une application web qui à travers l'interface de l'application vont tester l'ensemble de l'application (front-end, back-end, base de donnée...). On peut se demander comment ces test se positionnent dans une démarche globale de qualité pour un projet.

### Stratégie de tests
La première chose à indiquer est que le positionnement des tests e2e est différent de celui des tests unitaires. En effet, les tests unitaires sont rapides à exécuter, et faciles à écrire et à maintenir, ce qui permet de s'approcher d'une couverture importante (le chiffre exact est sujet à débats, 100% de couverture pouvant être considéré trop extrême ou posant des problèmes d'architectures).

En revanche les tests e2e sont plus long à s'exécuter et plus compliqués à maintenir au fil du temps : l'interface a tendance à beaucoup bouger et les sélecteurs CSS doivent donc être actualisés. Une bonne pratique est donc de définir les chemins critiques pour le coeur de métier de l'application. Par exemple ~20% des fonctionnalités : créer un compte, se connecter, payer, réserver, envoyer un message...
Un concept intéressant est donc celui de  [Pyramide des tests](https://martinfowler.com/bliki/TestPyramid.html) comme le décrit Martin Fowler.

Une autre question est la place du service QA par rapport aux tests e2e automatisés. Ces tests n'ont pas pour but de remplacer la QA, qui peut aider à les définir et/ou les écrire. Cependant ils peuvent libérer du temps pour faire de l'*exploratory testing*, c'est-à-dire des tests poussés qui ne suivent pas forcément des scénarios pré-établis, mais peuvent être basés sur des données issues des analytics et sur l'expérience du domaine métier du testeur. Voir le poste de Joel Spolsky (cofondateur de Stack Overflow) sur les [5 mauvaises raisons pour lesquelles vous n'avez pas de testeurs](https://www.joelonsoftware.com/2000/04/30/top-five-wrong-reasons-you-dont-have-testers/).

### Nightmare.js

La librairie de tests sur laquelle j'ai choisi de mettre l'accent est [Nightmare.js](http://www.nightmarejs.org/). En effet, elle présente plusieurs avantages :

- Entièrement en JS (utilise Electron)
- Très facile à installer : `npm i nightmare` et rien d'autre à configurer
- API claire et [bonne doc/exemples sur GitHub](https://github.com/segmentio/nightmare#api)
- S'utilise avec mocha pour écrire les suites de tests. C'est une solution connue de la communauté et il n'y a pas de "magie" derrière dans l'exécution des tests. On peut choisir la librairie d'assertion qu'on préfère ou bien Cucumber JS également pour écrire des tests style Gherkin.

Voyons un premier exemple minimaliste : se connecter au site de Sfeir et cliquer sur la première news.
{% highlight javascript %}
var nightmare = require('nightmare')({ show: true });
nightmare
  .goto('https://sfeir.com')
  .wait(2000)
  .click('#tuiles a[data-type="ajxN2"]')
  .wait(2000)
  .end()
  .then(function () {
    console.log('test finished');
  });
{% endhighlight %}
On peut lancer ce scénario directement avec la commane `node index.js`. Le scénario va s'exécuter dans une fenêtre avec Electron et je pourrai voir le résultat.

Si on veut écrire un vrai système de tests avec mocha on peut écrire les tests comme ceci :
{% highlight javascript %}
const Nightmare = require('nightmare')
var expect = require('chai').expect;

describe('Load a Page', function() {
  // Recommended: 5s locally, 10s to remote server, 30s from airplane ¯\_(ツ)_/¯
  this.timeout('30s')

  let nightmare = null
  beforeEach(() => {
    nightmare = new Nightmare()
  })

  describe('/ (Sfeir first news)', () => {
    it('should be Bouffe front', done => {
      nightmare
        .goto('https://sfeir.com')
        .wait(2000)
        .click('#tuiles a[data-type="ajxN2"]')
        .wait(2000)
        .evaluate(function () {
          return document.querySelector('h1').innerText;
         })
        .end()
        .then(function (text) {
          expect(text).to.equal('BOUFFE FRONT');
          done()
        })
        .catch(function (error) {
          console.error('Search failed:', error);
        });
    })
  })
})
{% endhighlight %}
C'est quasiment le même scénario sauf que je vérifie la valeur du texte du titre de l'article pour faire passer ou non le test. On peut lancer le test avec la commande `mocha`ou `npm test` selon la configuration qu'on utilise.

### Autres librairies de tests e2e
Beaucoup d'autres libraires de tests e2e existent. Je vais les lister rapidement avec les raisons qui m'ont fait préfer nightmare :
- Nightwatch.js : a besoin de Java et d'un serveur Selenium. Peut être utile quand on a besoin de tester sur plusieurs navigateurs mais plus lourd à configurer.
- Protractor : la solution préconisée pour tester les application Angular. Je n'avais pas envie de limiter les tests possible à un framework.
- CodeceptJS : API "high-level" qui permet d'écrire les scnéarios de tests et de choisir ensuite comment les faire tourner (Selenium, WebDriverIO, Nightmare.js...). L'idée est intéressante mais assez compliquée dans mon cas d'utilisation.

## Conclusion

- Les parties les plus compliquées sont de définir quels tests on veut écrire, et trouver les sélecteurs CSS dont on a besoin.
- La règle de la pyramide des tests où on doit tester en e2e uniquement les features les plus importantes pour le business me paraît très importante
- Dans la majorité des cas tester sur un seul navigateur me paraît suffisant, si il faut une précision pixel perfect on devra sans doute passer par des tests avec screenshots.

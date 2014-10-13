Framework sohoa
=====

Sohoa est un framework MVC écrit en PHP sur la base des bibliothèques de [Hoa](http://www.hoa-project.net).

Installation
-----

[composer](http://getcomposer.org/doc/00-intro.md#locally)
permet d'installer sohoa très rapidement au moyen de la commande suivante :

```BASH
composer create-project -s dev sohoa/sohoa myProject
```

ou ```myProject``` est le nom que vous souhaitez donner à votre projet.

Configuration du serveur HTTP
-----

Le but est de créer un hôte virtuel et d'activer la réécriture d'URL pour le serveur
HTTP. La documentation de Hoa sur la [configuration des serveurs HTTP](http://hoa-project.net/Fr/Literature/Learn/Appendix_http_servers.html)
est applicable directement à Sohoa mais le répertoire cible de l'hôte virtuel est
le dossier ```Public``` (et non pas ```Application/Public```).

Routeur
-----

Les routes sont décrites dans le fichier ```Application/Config/Route.php```. Dans
ce fichier ```$this``` correspond à une instance de Sohoa\Framework\Router.

Sohoa\Framework\Router étends [Hoa\Router](https://github.com/hoaproject/Router),
par conséquent il est possible d'utiliser directement l'API de cette librairie et
de profiter de son [excellente documentation](http://hoa-project.net/Fr/Literature/Hack/Router.html).

Toutefois, Sohoa ajoute également des fonctions de plus haut niveau à Hoa\Router,
notamment :

* le nommage des routes avec l'option ```as```:
```PHP
$this->get('/login', array('as' => 'login',
                           'to' => 'Login#new'));
```
ce qui permet d'accéder à cette route par son nom ```login```.

* l'option ```to``` qui permet de faire pointer une route vers un contrôleur/méthode :
```PHP
$this->get('/login', array('as' => 'login',
                           'to' => 'Login#new'));
```
envoie automatiquement les requêtes sur ```/login``` vers la méthode ```new``` du
contrôleur ```Login```.

* la gestion des ressources ([REST](http://en.wikipedia.org/wiki/Representational_state_transfer#Applied_to_web_services))
avec la méthode ```resource``` et ses options
```only``` et ```except``` :
```PHP
$this->resource('vehicles', array('only' => array('index' , 'show')))
     ->resource('fireman' , array('only' => array('index')));
```
ce qui va générer les routes suivantes :
```
indexVehicles           get    /vehicles/                                  Vehicles#index
showVehicles            get    /vehicles/(?<vehicles_id>[^/]+)             Vehicles#show
indexVehiclesFireman    get    /vehicles/(?<vehicles_id>[^/]+)/fireman/    Fireman#index
```

* les alias (pour les ressources mais aussi pour les routes standards) avec l'option ```alias```:
```PHP
  $router->resource('vehicles', array('alias' => 'foo' , 'only' => array('index' , 'show')))
         ->resource('fireman' , array('alias' => 'bar' , 'only' => array('index')));
```
ce qui va générer les routes suivantes :
```
indexVehicles           get    /foo/                              Vehicles#index
showVehicles            get    /foo/(?<vehicles_id>[^/]+)         Vehicles#show
indexVehiclesFireman    get    /foo/(?<vehicles_id>[^/]+)/bar/    Fireman#index
```

* les préfixes avec l'option ```prefix```:
```PHP
$router->resource('fireman' , array('prefix' => '/admin' , 'only' => array('index', 'show')))
       ->resource('vehicles', array('only' => array('index', 'show')));
```
ce qui va générer les routes suivantes :
```
indexFireman            get    /admin/fireman/                                                       Fireman#index
showFireman             get    /admin/fireman/(?<fireman_id>[^/]+)                                   Fireman#show
indexFiremanVehicles    get    /admin/fireman/(?<fireman_id>[^/]+)/vehicles/                         Vehicles#index
showFiremanVehicles     get    /admin/fireman/(?<fireman_id>[^/]+)/vehicles/(?<vehicles_id>[^/]+)    Vehicles#show
```

Il est possible de consulter les routes générées depuis le fichier de route grâce
à la commande ```sohoa Router:Dump```.

Environnement
-----

Vues Greut
-----

Greut est un moteur de template en PHP intégré à Sohoa et largement inspiré
de [Greut](https://github.com/greut/template), le moteur de template de Yoan Blanc.

Greut est le système de vue par défaut de Sohoa. Pour appeler une vue Greut depuis
une action d'un contrôleur, on utilisera le code suivant :

```PHP
public function IndexAction(){
    $this->greut->render();
}
```

Cette action va automatiquement aller chercher la vue ```/Application/View/Main/Index.tpl.php```

La méthode ```render()``` peut prendre deux types de données en paramètre, soit un
tableau contenant le nom d'un contrôleur et d'une action pour chercher sa vue
correspondante :

```PHP
$this->greut->render(array('myControllerName' , 'myActionName'));
```

Ou le chemin complet vers un fichier de vue :

```PHP
$this->greut->render('/an/path/to/the/view.tpl.php');
```

Dans le premier cas le fichier de vue utilisé sera ```/Application/View/myControllerName/myActionName.tpl.php```

##### Passage de données à la vue

Toutes les données affiliées à ```$this->data``` dans un contrôleur seront
transmises à la vue Greut. Ainsi :

```PHP
$this->data->foo = 'bar';
```

permettra d'utiliser le code suivant dans la vue :

```PHP
echo $foo; // qui retournera bar
```

Glossaire
-----

##### Kit

Un kit est une classe qui apporte un lot de fonction générique à une
classe donnée. Sohoa fournit par exemple un kit qui gère les redirections en ajoutant
cette fonctionnalité à tous les contrôleurs. Il possible d'utiliser les kits pour
ajouter des fonctions métiers (par exemple pour gérer des autorisations d'accès).

##### Helper

Les helpers sont des fonctions regroupées autour d'un thème (catégorie) communes
pour permettre d'ajouter des usages/fonctionnalités au système de vue. Ainsi on
peut avoir le helper suivant dans le fichier ./Application/View/Main/Index.tpl.php :

```PHP
$this->router->unroute('root'); // va générer "/"
$this->html->a('root' , [] , 'foo' , 'bar') // Pourrait générer <a href="/" class="foo">bar</a>
```

##### Service

Un services est une classe php que l'on instancie et que l'on rend disponible à l'ensemble de l'application
selon le principe du patron de conception [singleton](http://fr.wikipedia.org/wiki/Singleton_%28patron_de_conception%29).

Contribution
-----

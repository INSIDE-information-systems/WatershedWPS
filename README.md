# WatershedWPS

Ce repository documente le service de calcul de bassins versants mis en place dans le cadre du [pôle INSIDE](http://www.pole-inside.fr/fr). Il a vocation à accueillir les documentations et aides à l'utilisation, les codes exemples de clients, et à fédérer les _issues_. 

Le service offre la possibilité de calculer un bassin versant en tout point du territoire français et ce, selon différentes configurations préparées à l'avance : 1 configuration = 1 choix préalable de données d'entrées (MNT, réseau hydrographique ayant servi à brûler éventuellement le MNT...) et de paramétrage d'algorithmes aux différentes étapes du prétraitement ("brûlage", calcul de direction, lissage, accumulation...). 

Il se compose principalement d'un service [OGC:WPS](https://www.opengeospatial.org/standards/wps), offrant les méthodes de description des capacités, de calcul et de récupération des bassins versants. Il est également complété par un lot de services annexes facilitant son utilisation (service [OGC:WMS](https://www.opengeospatial.org/standards/wms) des données prétraitées, telles que les bassins versants élémentaires, le réseau hydrographique modélisé...).

## Service de traitement OGC:WPS Bassin Versant

Endpoint du service : http://reseau.eaufrance.fr/geotraitements/bassin-versant/services/wps/ows? 

Les paragraphes suivants décrivent les différentes procédures mises en œuvre sur le calcul de bassins versants (ou BV dans la suite de la documentation). Ces traitements spatiaux sont confectionnées sur la base d'un MNT (Modèle Numérique de Terrain) et éventuellement de données de référence fournissant le réseau hydrographique connu, le tout prétraité ; chaque prétraitement, associé à un choix de données d'entrée (MNT, réseau hydrographique) et d'algorithmie (types d'algorithmes et paramètres aux différentes étapes de prétraitement) donne une configuration distincte, sur la base de laquelle des calculs de bassins versants peuvent être demandés au service ici décrit.

Les procédures s’appuient sur les qualités sémantiques et géométriques des données d'entrée.


### Procédure 1 : Liste des configurations (Identifier=_sie:getconfiglist_)

La procédure _sie:getconfiglist_ (Voir le [DescribeProcess](http://reseau.eaufrance.fr/geotraitements/bassin-versant/services/wps/ows?service=WPS&version=1.0.0&request=DescribeProcess&identifier=sie:getconfiglist)) retourne la liste des configurations utilisables ensuite comme paramètre d'entrée pour la plupart des traitements. A appeler avant toute autre chose, à moins que l'on se satisfasse de la configuration par défaut.


#### Liste des paramètres d'exécution ( DataInputs ) :

Aucun


#### Liste des sorties disponibles ( RawDataOutput ) :
Identifiant 	Définition
result 	Liste des configurations disponibles



### Procédure 2 : Calcul de BV à partir de points et téléchargement des résultats (Identifier=_sie:pts2watershed_)

La procédure _sie:pts2watershed_ »_ (voir le [DescribeProcess](http://reseau.eaufrance.fr/geotraitements/bassin-versant/services/wps/ows?service=WPS&version=1.0.0&request=DescribeProcess&identifier=sie:pts2watershed)) lance le calcul d'un BV pour chaque point (exutoire) passé en entrée sur la base d'une configuration choisie (parmi celles renvoyées par la méthode _sie:getconfiglist_), et retourne les résultats dans le format demandé (voir le [DescribeProcess](http://reseau.eaufrance.fr/geotraitements/bassin-versant/services/wps/ows?service=WPS&version=1.0.0&request=DescribeProcess&identifier=sie:pts2watershed) pour connaître les formats supportés). Derrière cette méthode, le point envoyé en entrée est ramené (_snappé_) sur le réseau hydrographique modélisé, en suivant la plus forte pente jusqu'à retrouver un tronçon.


#### Liste des paramètres d'exécution ( DataInputs ) :
Nom du paramètre | Nature | Définition
---------------- | ------ | ----------
compute_points | Obligatoire | Géométries des points à partir desquels calculer les BV. Ces géométries peuvent être exprimées dans plusieurs formats (listes des formats supportés dans le _DescribeProcess_ de la méthode), certains pouvant embarquer le système de référence (SRS), d'autres ne le pouvant pas
input_EPSG | Facultatif | Code EPSG à attribuer aux géométries envoyées via le paramètre _compute_points_. A utiliser quasi exclusivement lorsque les géométries fournies ne stipulent pas par ailleurs leur SRS. Dans cette version du service, tout autre SRS que le 2154 (Lambert93) sera refusé et lèvera une erreur
configuration_id | Obligatoire | Identifiant de la configuration à utiliser pour ce calcul (récupéré d'un appel préalable à _sie:getconfiglist_)


#### Liste des sorties disponibles ( RawDataOutput ) :
Identifiant | Définition
----------- | ----------
result | Résultats de calcul au format demandé


#### Liste des formats de sortie disponibles ( MimeType ) :
MimeType | Définition
-------- | ----------
application/wfs-collection-1.0 ou text/xml; subtype=wfs-collection/1.0 | Document XML - GML conforme au schéma WFS 1.0
application/wfs-collection-1.1 ou text/xml; subtype=wfs-collection/1.1 | Document XML - GML conforme au schéma WFS 1.1
application/json | Document GeoJSON
application/zip |	Archive ZIP


### Procédure 3 : Calcul de BV à partir de points et récupération du WMS associé (Identifier=_sie:pts2watershedtoken_and_wms_)

La procédure _sie:pts2watershedtoken_and_wms_ (Voir le [DescribeProcess](http://reseau.eaufrance.fr/geotraitements/bassin-versant/services/wps/ows?service=WPS&version=1.0.0&request=DescribeProcess&identifier=sie:pts2watershedtoken_and_wms)) lance le calcul d'un BV pour chaque point (exutoire) passé en entrée sur la base d'une configuration choisie (parmi celles renvoyées par la méthode _sie:getconfiglist_), mais ne retourne pas les résultats directement dans le flux de réponse. Retourne un jeton correspondant à ce calcul (pour réexploitation future) ainsi que les paramètres d'appel (URL et LayerName) d'un service WMS affichant ces résultats. Derrière cette méthode, le point envoyé en entrée est ramené (_snappé_) sur le réseau hydrographique modélisé, en suivant la plus forte pente jusqu'à retrouver un tronçon.


#### Liste des paramètres d'exécution ( DataInputs ) :
Nom du paramètre | Nature | Définition
---------------- | ------ | ----------
compute_points | Obligatoire | Géométries des points à partir desquels calculer les BV. Ces géométries peuvent être exprimées dans plusieurs formats (listes des formats supportés dans le _DescribeProcess_ de la méthode), certains pouvant embarquer le système de référence (SRS), d'autres ne le pouvant pas
input_EPSG | Facultatif | Code EPSG à attribuer aux géométries envoyées via le paramètre _compute_point_. A utiliser quasi exclusivement lorsque les géométries fournies ne stipulent pas par ailleurs leur SRS. Dans cette version du service, tout autre SRS que le 2154 (Lambert93) sera refusé et lèvera une erreur
configuration_id | Obligatoire | Identifiant de la configuration à utiliser pour ce calcul (récupéré d'un appel préalable à _sie:getconfiglist_)


#### Liste des sorties disponibles ( RawDataOutput ) :
Identifiant | Définition
----------- | ----------
token | Identifiant du calcul / de la requête, à utiliser ensuite comme jeton pour appeler d'autres services par la suite (export...)
wms_url | URL du serveur WMS/WFS à appeler par la suite pour afficher ou récupérer une représentation des BV calculés
wms_layer | Nom de la couche du serveur WMS/WFS à appeler par la suite pour afficher ou récupérer une représentation des BV calculés

A partir de ces 3 éléments, il est possible de composer par exemple une URL complète pour afficher les BV en question comme suit : 

    <wms_url>?SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&LAYERS=<wms_layer>&TOKEN=<id>&...


Exemple : http://mapsref.brgm.fr/wxs/rhf/bvruntime?LAYERS=bv_runtime&TRANSPARENT=true&VERSION=1.1.1&FORMAT=image%2Fpng&SERVICE=WMS&REQUEST=GetMap&STYLES=&EXCEPTIONS=application%2Fvnd.ogc.se_inimage&SRS=EPSG%3A2154&X=226591.3420795&Y=6776903.7971716&TOKEN=3007cc81-40fe-4ec4-8c19-046a97a5cbc5&BBOX=226026.45696787,6776616.7244099,226910.16482401,6777212.0365885&WIDTH=668&HEIGHT=450


#### Liste des formats de sortie disponibles ( MimeType ) :
MimeType | Définition
-------- | ----------
application/wfs-collection-1.0 ou text/xml; subtype=wfs-collection/1.0 | Document XML - GML conforme au schéma WFS 1.0
application/wfs-collection-1.1 ou text/xml; subtype=wfs-collection/1.1 | Document XML - GML conforme au schéma WFS 1.1
application/json | Document GeoJSON
application/zip |	Archive ZIP


### Procédure 4 : Téléchargement de BV préalablement calculés (Identifier=_sie:token2watershed_)

La procédure _sie:token2watershed_ (Voir le [DescribeProcess](http://reseau.eaufrance.fr/geotraitements/bassin-versant/services/wps/ows?service=WPS&version=1.0.0&request=DescribeProcess&identifier=sie:token2watershed)) permet de télécharger le résultat (ensemble de BV) d'une requête formulée précédemment en lui fournissant le jeton (obtenu lors de la requête initiale), et le format d'export choisi parmi ceux supportés (décrits dans le _DescribeProcess_).


#### Liste des paramètres d'exécution ( DataInputs ) :
Nom du paramètre | Nature | Définition
---------------- | ------ | ----------
token | Obligatoire | Identifiant d'un calcul / d'une requête exécutée précédemment, et dont on veut exporter les résultats


#### Liste des sorties disponibles ( RawDataOutput ) :
Identifiant | Définition
----------- | ----------
result | Résultats de la requête en question, au format demandé


#### Liste des formats de sortie disponibles ( MimeType ) :
MimeType | Définition
-------- | ----------
application/wfs-collection-1.0 ou text/xml; subtype=wfs-collection/1.0 | Document XML - GML conforme au schéma WFS 1.0
application/wfs-collection-1.1 ou text/xml; subtype=wfs-collection/1.1 | Document XML - GML conforme au schéma WFS 1.1
application/json | Document GeoJSON
application/zip |	Archive ZIP


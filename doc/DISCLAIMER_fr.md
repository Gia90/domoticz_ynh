Domoticz est un syst�me de domotique permettant de controler diff�rents objets et de recevoir des donn�es de divers senseurs
Il peut par exemple �tre utilis� avec :

-des interrupteurs

-des senseurs de portes

-des sonnettes d'entr�es

-des syst�mes de s�curit�

-des stations m�t�o pour les UV, la pluie, le vent...

-des sondes de temp�ratures

-des sondes d'impulsions

-des voltm�tres

-Et bien d'autres

**Version incluse :** Toujours la derni�re version stable. La derni�re version compil�e est r�cup�r�e dans [ce r�pertoire](https://releases.domoticz.com/releases/?dir=./beta)
Une fois install�e, **les mises � jour de l'application sont g�r�es depuis les menus de l'application elle m�me.**. Le script de mise � jour Yunohost mettra uniquement � jour de nouvelles version du package.

## Configuration

### Senseurs, langue et ce genre de choses
Toute la configuration de l'application a lieu dans l'application elle m�me
Main configuration of the app take place inside the app itself.

### Acc�s et API
Par d�faut, l'acc�s aux [API JSON](https://www.domoticz.com/wiki/Domoticz_API/JSON_URL's) est autoris� sur cette URL `/votredomaine.tld/api_/chemindedomoticz`.
Donc, si vous acc�dez � domoticz par https://votredomaine.tld/domoticz, utilisez le chemin suivant pour l'api: `/votredomaine.tld/api_/domoticz/json.htm?votrecommandeapi`

Par d�faut, seuls la mise � jour de senseur et les interrupteurs sont autoris�s. Pour autoriser une nouvelle commande, vous devez (pour l'instant) manuellement �diter le fichier de configuration nginx :
````
sudo nano /etc/nginx/conf.d/yourdomain.tld.d/domoticz.conf
````
Puis �diter le bloc suivant en y ajoutant le regex de la commmande � autoriser :
````
  #set the list of authorized json command here in regex format
  #you may retrieve the command from https://www.domoticz.com/wiki/Domoticz_API/JSON_URL's
  #By default, sensors updates and toggle switch are authorized
  if ( $args ~* type=command&param=udevice&idx=[0-9]*&nvalue=[0-9]*&svalue=.*$|type=command&param=switchlight&idx=[0-9]*&switchcmd=Toggle$) {
    set $api "1";
    }
````
Par exemple, pour ajouter la commmande json pour retrouver le statut d'un �quipement (/json.htm?type=devices&rid=IDX),il faut modifier la ligne comme ceci:
````
  #set the list of authorized json command here in regex format
  #you may retrieve the command from https://www.domoticz.com/wiki/Domoticz_API/JSON_URL's
  #By default, sensors updates and toggle switch are authorized
  if ( $args ~* type=command&param=udevice&idx=[0-9]*&nvalue=[0-9]*&svalue=.*$|type=command&param=switchlight&idx=[0-9]*&switchcmd=Toggle$|type=devices&rid=[0-9]* ) {
    set $api "1";
    }
````

Toutes les adresses IPv4 du r�seau local (192.168.0.0/24) et toutes les adresses IPv6 sont autoris�es pour l'API.
A ma connaissance, il n'y a pas moyen d'effectuer un filtre pour les adresses IPv6 sur le r�seau local, vous pouvez donc retirer leur autorisation en enlevant ou en commentant la ligne suivante dans `/etc/nginx/conf.d/yourdomain.tld.d/domoticz.conf`:
````
allow ::/1;
````
Ceci autorisera seulement les adresses IPv4 local a acc�der aux API de domoticz.
Vous pouvez ajouter des adresses IPv6 de la m�me fa�on.

## Limitations

* Pas de gestion d'utilisateurs ni d'int�gration LDAP. L'application ne [pr�voit pas de g�rer les utilisateurs par LDAP](https://github.com/domoticz/domoticz/issues/838), donc le package non plus.
* Un backup ne peut pas �tre restaur� sur un type de machine diff�rente de celle d'origine (x86, arm...) car les sources compil�es doivent �tre diff�rente

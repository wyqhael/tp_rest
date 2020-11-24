<div style="text-align:center"><img src="./fleur.png" alt="flower" style="zoom:35%;" /></div>



---

# REST : TP

---

## Sommaire

[TOC]





## Utiliser la méthode REST

**Prise : 6534**

On se connecte sur le réseau wifi de la prise.



### Interface WEB

On peut joindre la prise par l'interface web sur l'adresse `192.168.33.1` (doc. section *HTTP Server*).

``` shell
$ wget 192.168.33.1
--2020-11-24 09:46:53--  http://192.168.33.1/
Connecting to 192.168.33.1:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 56460 (55K) [text/html]
Saving to: ‘index.html’

index.html              100%[==============================>]  55,14K  11,1KB/s    in 4,3s    

2020-11-24 09:46:58 (12,9 KB/s) - ‘index.html’ saved [56460/56460]
```

On va sur l'interface depuis le navigateur :

![image-20201124094955599](/home/wyq/.config/Typora/typora-user-images/image-20201124094955599.png)

Sur la droite de l'interface on voit un bouton **POWER**. On peut allumer et éteindre la prise a l'aide de ce bouton :

![image-20201124095145834](/home/wyq/.config/Typora/typora-user-images/image-20201124095145834.png)

Une fois allumée on peut voir la consommation électrique.





### Curl

``` shell
$ curl -X GET 192.168.33.1/settings | json_pp -json_opt pretty,canonical
```

Récupérer la consommation electrique :

 ``` shell
$ curl -X GET 192.168.33.1/meter/0 | json_pp -json_opt pretty,canonical | grep "\"power\""
"power" : 50.09,
 ```

Allumer/éteindre la LED :

``` shell
$ curl http://192.168.33.1/relay/0?turn=on | json_pp -json_opt pretty,canonical
{
   "has_timer" : false,
   "ison" : true,
   "overpower" : false,
   "source" : "http",
   "timer_duration" : 0,
   "timer_remaining" : 0,
   "timer_started" : 0
}


$ curl http://192.168.33.1/relay/0?turn=off | json_pp -json_opt pretty,canonical
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   123  100   123    0     0   6833      0 --:--:-- --:--:-- --:--:--  6833
{
   "has_timer" : false,
   "ison" : false,
   "overpower" : false,
   "source" : "http",
   "timer_duration" : 0,
   "timer_remaining" : 0,
   "timer_started" : 0
}
```

On allume en faisant une requête `turn=on` et on éteint en faisant une requête `turn=off`.



### jq

On récupère la puissance :

``` shell
$ curl -X GET 192.168.33.1/meter/0 | jq .power
0
```

La prise est éteinte donc on a une valeur de 0.

`jq` est bien plus pratique que `grep` pour récupérer des champs json.



### Scripts

Consommation de la Prise 1 :

``` shell
#!/bin/bash
# Recupere la consommation electrique
conso=$(curl -X GET 192.168.33.1/meter/0 --silent | jq .power)
# Affiche la conso
echo -e "Consommation actuelle de la prise : $conso\n"
```

On lance le script :

``` shell
$ ./p1-info 
Consommation actuelle de la prise : 0
```



Allumer la prise :

``` shell
#!/bin/bash
curl http://192.168.33.1/relay/0?turn=on --silent
# Affiche l'etat de la prise pour s'assurer qu'elle est allumee
etat=$(curl http://192.168.33.1/relay/0?turn=on --silent | jq .ison)
# Affiche l'etat de la prise apres l'allumage, si l'etat n'est pas a true previent que le script ne s'est pas bien execute
if [ "$etat" == "true" ]
then
	echo -e "\nLa prise est allumee\n"
else
	echo -e "\nLa prise ne s'est pas bien allumee"
fi
```

Eteindre la prise :

```shell
#!/bin/bash
curl http://192.168.33.1/relay/0?turn=off --silent
# Affiche l'etat de la prise pour s'assurer qu'elle est allumee
etat=$(curl http://192.168.33.1/relay/0?turn=off --silent | jq .ison)
# Affiche l'etat de la prise apres l'avoir eteinte, si l'etat n'est pas a false previent que le script ne s'est pas bien execute
if [ "$etat" == "false" ]
then
	echo -e "\nLa prise est eteinte\n"
else
	echo -e "\nLa prise ne s'est pas bien eteinte"
fi
```



### Échanges HTTP 

L’échange HTTP se fait en 2 trames :

- La trame qui contient la requête "GET", envoyée par le client
- Une réponse en **json**, envoyée par le prise.

Sur wireshark on va dans le section **HTTP** > **GET**, on sélectionne le champs `Request URI Query Parameter: turn=on `, elle fait *7 bytes*, soit *7 octets*



### Tag sur le dernier commit

``` shell
$ git tag "partie_1" <num commit>
```







## MQTT
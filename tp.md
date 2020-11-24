<div style="text-align:center"><img src="../ido/fleur.png" alt="flower" style="zoom:35%;" /></div>



**GERARDIN Clemence**



---

# REST : TP

---



**Repo GIT** : https://github.com/wyqhael/tp_rest/



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

On cree 3 scripts :

- info-prise qui retourne la consommation de la prise
- prise-on qui allume la prise et retourne son état (pour debugger)
- prise-off qui éteint la prise et retourne son état (pour debugger)



### Échanges HTTP 

L’échange HTTP se fait en 2 trames :

- La trame qui contient la requête "GET", envoyée par le client
- Une réponse en **json**, envoyée par le prise.





### Tag sur le dernier commit

``` shell
$ git tag "partie_1" 0152ee2361985c648bbcb0da815364ce6a7bdc57
```

On peut voir le tag avec `git log` :

``` shell
$ git log
commit 0152ee2361985c648bbcb0da815364ce6a7bdc57 (HEAD -> master, tag: partie_1, origin/master)
Author: wyqhael <turashusama@gmail.com>
Date:   Tue Nov 24 13:27:30 2020 +0100

    compte-rendu a jour
```

On push le tag :

``` shell
$ git push --tag 
```







## MQTT

Le broker est a l'adresse `10.202.0.107`.

``` shell
$ mosquitto_pub -h 10.202.0.107 -t shellies/shellyplug-s-6A6534/relay/0/command -l
off
on
```

La prise s’éteint et s'allume waho.

Quand on se s'abonne au topic on voit :

``` shell
$ mosquitto_sub -h 10.202.0.107 -t shellies/#
off
0.00
236
29.29
84.73
0
on
off
off
```



### Scripts

On crée 3 scripts :

- on qui allume la prise
- off qui éteint la prise
- toggle qui bascule la prise dans L’état inverse
- status qui qui s'abonne au topic pour voir les états de la prise





### Tag sur le dernier commit

``` shell
$ git tag "partie_2"
```

On push le tag :

``` shell
$ git push --tag
```







## Serveur REST

``` shell
$ apt install ngnix
$ systemctl start ngnix
```

Le fichier de configuration se trouve dans : `/etc/nginx/nginx.conf`.

On modifie la config pour avoir la page php quand on arrive sur **NGINX**.
**GERARDIN Clemence**

---

# REST : TP 2



## Sommaire

[TOC]





## Exercice 1.

> Il existe beaucoup de façons différentes de faire du web dynamique: mod_php ( sous apache ), cgi, fastcgi, fpm-php ... Expliquez en une page maximum comment tout ceci s’organise et fonctionne. Quels sont les avantages et inconvénients de chaque méthode?

- Un client envoie des requêtes HTTP au serveur, qui peut être sous **Apache2**, **NGINX**, ...

- Pour répondre à la requête du client le serveur fait appel à la page web demandée par le client

  - Avec mod_php le serveur fait appel à la page directement dans son processus.

  - Avec CGI le serveur fait appel à CGI qui appelle une page dans un langage (C, C++, python, ...) réinterprétée par un interpréteur (spawnfcgi) en php. 
  - FCGI fonctionne comme CGI, avec quelques petites différences expliquées plus bas.
  - FPM-PHP est une version de FCGI mais en php.



Sous apache il y a **mod_php**, qui sert à interpréter les fichiers php avec apache. **mod_php** tourne "dans" le processus apache. **mod_php** est toutefois moins sécurisé que **CGI**, le processus tourne dans apache.

**CGI** de séparer le process php du processus d'apache ce qui en fait une solution plus sécurisée. Mais **CGI** est bien plus lent, à chaque fois que le php est sollicité (assez souvent sur des sites dynamiques), une nouvelle instance de **CGI** est lancée. 

**FastCGI** est une version améliorée de **CGI**, elle est plus rapide.

**PHP-FPM** est une version de **FastCGI** exclusivement en php. Les process sont séparés mais pas réinterprétés ils sont directement écrits en php. L'inconvénient de cette méthode est qu'il faut tout faire en php...







## Exercice 2.

> Mettez en place un serveur Nginx, et configurez le support du php via fpm-php. Vous détaillerez les configurations en expliquant chaque paramètre nécessaire au fast-cgi.

### Installation de : nginx, php, php-fpm

``` shell
$ sudo apt install nginx php php-fpm
```

### Test

``` shell
$ curl http://localhost/index.nginx-debian.html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
</body>
</html>
```

### Configuration

Dans le fichier **/etc/nginx/sites-enabled/default** on décommente les lignes suivantes :

``` bash
location ~ \.php$ {
		include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
		fastcgi_pass unix:/run/php/php7.3-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	}
```







## Exercice 3.

> Expliquez le rôle de spawn-fcgi. Montrez comment le mettre en place avec Nginx pour un serveur affichant «hello world» via le langage C

**Spawn-fcgi** sert à séparer les processus **fcgi** des processus du serveur web. Il les fait tourner en daemon. Redémarrer le serveur web n'arrête pas le processus **fcgi**.



### Code C

``` c
#include "fcgi_stdio.h"
#include <stdlib.h>

int main(void)
{

 while(FCGI_Accept() >= 0)
 {
  printf("Content-type: text/html\r\nStatus: 200 OK\r\n\r\n<html>\n<body>\nhello world !!!</body>\n</html>\n");

 }

  return 0;
}
```



### Spawn-fcgi

Installation des paquets :

``` shell
$ sudo apt install libfcgi-dev spawn-fcgi
```

**Spawn-fcgi** sur le code :

``` shell
$ gcc -lfcgi hello.c -o hello
$ spawn-fcgi -a127.0.0.1 -p9000 -n ./hello
```

Dans **/etc/nginx/sites-enabled/default** on rajoute les lignes suivantes :

``` shell
location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		fastcgi_pass 127.0.0.1:9000;
		try_files $uri $uri/ =404;
	}
```

On curl **localhost** :

``` shell
$ curl localhost
<html>
<body>
hello world !!!</body>
</html>
```

On a bien la page qui affiche **hello world** !







## Exercice 4.

> Comment sont transmis les arguments du serveur web au programme C ? 
>
> Réalisez un exemple permettant de multiplier deux entiers transmis dans un formulaire via les méthodes GET et POST.

Les arguments du serveur web sont transmis au programme C via un flux



### POST

``` shell
		int i, ch, arg, add1, add2;
	    char nbr1[256];
	    char nbr2[256];
	    arg=0;
	    add1=0;
	    add2=0;
	    int equal=0;
	    printf("Standard input:<br>\n<pre>\n");
            for (i = 0; i < len; i++) {
                if ((ch  = getchar()) < 0) {
                    printf("Error: Not enough bytes received on standard input<p>\n");
                    break;
		}
		if(ch=='&'){
			arg++;
			equal=0;
		}
		else{
			if(equal==1 && arg==0){
                        nbr1[add1]=ch;
                        add1++;
			}
			if(arg==1 && equal==1){
                        nbr2[add2]=ch;
                        add2++;
			}
			if(ch=='='){
                        equal=1;
                        } 
		}
            }
	    nbr1[add1]='\0';
	    nbr2[add2]='\0';
	    int val1=atoi(nbr1);
	    int val2=atoi(nbr2);
            printf("valeur variables : %s  %s \n</pre><p>\n",nbr1, nbr2);
	    printf("val1 = %d * val2= %d === %d\n", val1, val2, val1*val2);
        }
```

On poste deux nombre avec `curl` :

``` shell
curl --data "nbr1=2&nbr2=3" localhost
<title>FastCGI echo</title><h1>FastCGI echo</h1>
Request number 1,  Process ID: 1104<p>
Standard input:<br>
<pre>
valeur variables : 2  3 
</pre><p>
val1 = 2 * val2= 3 === 6
```







## Exercice 5.

> En vous basant sur la question précédente, mettez en place une API REST permettant de piloter la lampe connectée de la salle en MQTT et d’en relever les paramètres.
>
> ATTENTION: il s’agit d’une **API**, pas d’un formulaire ...


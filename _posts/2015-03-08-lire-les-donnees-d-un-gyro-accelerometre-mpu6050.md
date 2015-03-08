---
layout: post
title: "Lire les données d'un gyro/accéléromètre MPU6050"
date: 2015-03-08 00:00:00
---

J'ai eu le temps à la fin des vacances de faire quelques tests avec un gyro/accéléromètre MPU6050 retrouvé au fond des tiroirs. En fouillant un peu sur les Internets, j'ai pu lire les données disponibles et faire une petite visualisation 3D en bonus.

## Le capteur

![Le capteur](/img/blog/2015-lire-les-donnees-d-un-gyro-accelerometre-mpu6050/sensor.jpg)

Le capteur possède 8 connecteurs (appelés _pins_), mais nous ne nous servirons que de 5 d'entre eux. Il est alimenté par 3.3 V via les pins _VCC_ et _GND_. Ce composant communique les données via les pins _SDA_ et _SCL_ en utilisant un protocole appelé [I2C](https://fr.wikipedia.org/wiki/I2C).

![Schéma simplifié du bus I2C](/img/blog/2015-lire-les-donnees-d-un-gyro-accelerometre-mpu6050/i2c-diagram.png)

L'intérêt est que I2C permet de brancher plusieurs composants à ces deux pins. Par exemple, sur le schéma ci-dessus, trois composants (_slaves_) sont reliés au Raspberry Pi (_master_). Les deux fils _SDA_ et _SCL_ sont donc appelés le _bus I2C_. Chaque périphérique branché sur le bus I2C a sa propre adresse qui permet de l'identifier (comme des numéros de maisons dans une rue). Le dernier pin _ADO_ sert à choisir cette adresse.

## Liaison avec le Raspberry Pi

Il nous faudra configurer le Raspberry Pi pour pouvoir utiliser le bus I2C. Je laisse à ceux qui voudraient le faire la lecture du [tutoriel d'Adafruit](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-4-gpio-setup/configuring-i2c).

Reste ensuite à connecter le MPU6050 au Raspberry Pi. Pour cela, il faut consulter le schéma des pins du Raspberry Pi :

![GPIO disponibles sur un Raspberry Pi modèle B](/img/blog/2015-lire-les-donnees-d-un-gyro-accelerometre-mpu6050/gpio.png)
<p class="text-center">_NB: les pins 1 et 2 sont vers l'extérieur du Raspberry Pi._</p>

En essayant de ne pas se gourrer de pin comme moi, il faut donc relier :

* Le pin 1 du Raspbeery Pi (3.3 V) au pin _VCC_ du MPU6050
* Le pin 3 à _SDA_
* Le pin 5 à _SCL_
* Le pin 6 à _GND_

Pour pouvoir tester les branchements, on lance la commande `i2cdetect` sur le Raspberry Pi :

```
$ sudo i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- 68 -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --
```

On peut voir apparaître le capteur avec l'adresse hexadécimale 0x68.

## Lecture des données via le bus I2C

Le bus I2C permet d'envoyer et recevoir des données du capteur. On parle d'écriture et de lecture. Pour chaque opération (écriture ou lecture), il faut préciser l'adresse du capteur et un registre. Ce registre est un numéro (on le notera en hexadécimal) qui permettra de signifier _ce que l'on veut lire ou écrire_, par exemple lire la coordonnée X de l'accélération (c'est une lecture) ou mettre le capteur en veille (c'est une écriture).

J'ai réalisé la lecture des données en codant un petit programme en [Node.js](https://nodejs.org). On pourra se référer à [la _datasheet_ du MPU6050](http://www.invensense.com/mems/gyro/documents/RM-MPU-6000A-00v4.2.pdf) pour se documenter sur les registres offerts par le capteur.

Il faut tout d'abord réveiller le capteur en écrivant un bit nul au registre 0x6b (le capteur est en veille par défaut lors de la mise sous tension). Ensuite, il suffit de lire une série de registres pour récupérer les différentes composantes : les données du gyromètre et de l'accéléromètre selon les axes X, Y et Z. Les données récupérées sont des entiers relatifs codés sur deux octets représentés avec le [complément à deux](https://fr.wikipedia.org/wiki/Compl%C3%A9ment_%C3%A0_deux).

J'ai donc créé un petit module Node.js pour lire les données du capteur : https://github.com/emersion/node-i2c-mpu6050

## Résultat

Après quelques heures de codage et déboguage, voici le résultat :

<iframe width="420" height="315" src="https://www.youtube.com/embed/_WRySGOwtGc" frameborder="0" allowfullscreen></iframe>

Avec l'aide de mon module Node.js et de la librarie [three.js](http://threejs.org/), on peut créer un petit serveur HTTP qui permet de visualiser l'orientation du capteur en temps réel. Vous pouvez voir le code source ici : https://github.com/emersion/node-mpu6050-server
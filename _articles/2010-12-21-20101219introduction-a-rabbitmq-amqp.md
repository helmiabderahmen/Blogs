---
ID: 94
post_title: 'Introduction à RabbitMQ &#8211; AMQP Partie I'
author: chebert
post_date: 2010-12-21 14:15:00
post_excerpt: |
  <p><a href="http://www.rabbitmq.com/" hreflang="en">RabbitMQ</a> est un broker de messages se basant sur le standard <a href="http://www.amqp.org/" hreflang="en">AMQP</a> afin d'échanger avec différents clients.</p> <p>Cette suite d'article présente AMQP, RabbitMQ ainsi que l'API de Spring pour établir des connections vers un broker AMQP.</p>
layout: post
permalink: http://blog.zenika-offres.com/?p=94
published: true
---
<p><a href="http://www.rabbitmq.com/" hreflang="en">RabbitMQ</a> est un broker de messages se basant sur le standard <a href="http://www.amqp.org/" hreflang="en">AMQP</a> afin d'échanger avec différents clients.</p> <p>Cette suite d'article présente AMQP, RabbitMQ ainsi que l'API de Spring pour établir des connections vers un broker AMQP.</p>
<!--more-->
<h2>AMQP</h2> <p>Le protocole AMQP sur lequel est basé RabbitMQ a pour but d'offrir un système d'échange totalement interopérable entre les différents acteurs. Contrairement à JMS, AMQP n'est pas une API, et de fait ne consiste pas une librairie pour une plateforme en particulier, mais est un protocole d'échange tel que HTTP ou SMTP.</p> <p>AMQP tire son origine du besoin de normer un système d'échanges de messages totalement asynchrone qui de plus s'abstrait totalement de l'implémentation du broker ou du client. En d'autres termes, il est tout à fait possible d'utiliser un broker AMQP en Erlang, sur lequel un client java va envoyer des messages qui seront consommés par un second client en .Net&nbsp;:</p> <p><img src="/wp-content/uploads/2015/07/Global.png" alt="Global.png" style="display:block; margin:0 auto;" /></p> <h3>Etablir la connection</h3> <p>AMQP table sur certains concepts pour gérer toute la communication entre les clients et le broker:</p> <p>Un client va établir une <strong>connection</strong> (<em>connection</em>) vers le broker.<br />
Au travers de cette connection, il pourra ouvrir un ou plusieurs <strong>channels</strong> (<em>canaux</em>) de communication grâce auxquels les requêtes pourront être transmises au broker, en parallèle.</p> <p><img src="/wp-content/uploads/2015/07/Connection.png" alt="Connection.png" style="display:block; margin:0 auto;" /></p> <p>Généralement, une application ne nécessite que l'ouverture d'une unique connexion; toute les communications sont ensuite effectuées au travers de plusieurs channels différents pendant la durée de vie de l'application.</p> <h3>Les bases d'un échange</h3> <p>Une fois connecté, un client va pouvoir accéder aux <strong>exchanges</strong> (<em>échanges</em>) et aux <strong>queues</strong> (<em>files</em>) pour transmettre et recevoir les messages.</p> <p>L'<strong>exchange</strong> est le point où un message peut être déposé&nbsp;; de là, le message suivra ensuite un cheminement pour enfin être placé dans la ou les queues de destination, où il sera entreposé jusqu'à consommation. Plusieurs client différents pourront envoyer des messages sur un même exchange, ainsi plusieurs messages envoyés pourront suivre le même mapping.</p> <p>Le chemin pris par le message sera défini grâce aux <strong>bindings</strong> (<em>liaisons</em>) que l'on pourra appliquer entre un exchange et une queue. Ainsi il sera possible de définir la destination d'un message directement depuis le broker.<br />
Ce système permet d'avoir un client producteur de messages indépendant du consommateur&nbsp;; ainsi, en cas de changement d'architecture, il suffira de modifier les bindings adéquats sur le broker, les deux clients (producteur et consommateur) ne nécessitent alors pas de modification dans le code.</p> <p>La <strong>queue</strong> est le point de consomation des messages; tout comme pour les exchanges, il est possible d'avoir plusieurs consommateurs sur une même queue.<br />
Chaque consommateur peut demander au broker de lui envoyer le prochain message stocké dans la queue. Dès lors le message est envoyé au client et supprimé de la queue. Un message consommé n'est plus disponible, et de fait ne peut pas être consommé à nouveau.</p> <p>Afin de simplifier les schémas, un client producteur de message sera symbolisé par <img src="/wp-content/uploads/2015/07/Producer.png" alt="Producer.png" />; un client consommateur de messages sera signalé par <img src="/wp-content/uploads/2015/07/Consumer.png" alt="Consumer.png" />; un exchange par <img src="/wp-content/uploads/2015/07/Exchange.png" alt="Exchange.png" /> et une queue grâce à <img src="/wp-content/uploads/2015/07/Queue.png" alt="Queue.png" /></p> <h3>Premiers échanges (mode Fanout)</h3> <p>Le cas le plus basique est l'envoi d'un message sur un exchange désservant une queue particulière, et la consommation sur cette queue par un unique client.</p> <p><img src="/wp-content/uploads/2015/07/Classic.png" alt="Classic.png" style="display:block; margin:0 auto;" /></p> <p>Il est aussi possible d'envoyer le même message sur deux queues différentes afin que deux consommateurs puissent le recevoir et le traiter. A ce moment là le message est dupliqué, et chaque queue contient sa propre copie.</p> <p><img src="/wp-content/uploads/2015/07/Fanout.png" alt="Fanout.png" style="display:block; margin:0 auto;" /></p> <p>Une autre variante consiste à déclarer plusieurs exchanges qui envoient des messages sur une queue commune.</p> <p><img src="/wp-content/uploads/2015/07/Fanout2.png" alt="Fanout2.png" style="display:block; margin:0 auto;" /></p> <p>Ici les messages reçus sur Exchange1 ou Exchange2 seront tous disponibles sur Queue2</p> <p>Ce type d'échange est appelé <strong>fanout</strong> et représente une communication 1:N. Pour un exchange donné, les N queues liées pourront recevoir le message.</p> <h3>Utilisation des routing keys (mode Direct)</h3> <p>Afin de personnaliser les directions prises par un message, un système de binding/routing key est disponible. Chaque binding pourra définir sa propre binding key qui sera une chaîne de caractère identifiant un mapping précis. Les messages pourront de leur côté définir une routing key qui servira à définir au travers de quel binding le message devra passer. Une routing key est simplement un identifiant écrit sous la forme de noms séparés par des points. Par exemple si l'on devait représenter une key pour l'indice VMW, elle pourrait être sous la forme <em>stock.us.vmw</em>.</p> <p>La routing key sera comparée aux binding keys de chaque binding disponible; si l'un correspond alors le message sera transféré uniquement sur la queue représentée dans ce binding.<br />
Dans le cas ou la routing key ne correspondrait à aucune des possibilités, alors le message serait simplement ignoré.</p> <p><img src="/wp-content/uploads/2015/07/Direct.png" alt="Direct.png" style="display:block; margin:0 auto;" /></p> <p>Un message ayant la Routing key <em>stock.us.vmw</em> serait disponible sur Queue1 uniquement.</p> <p>Comme auparavant il est possible d'ajouter d'autres acteurs pour augmenter les possibilités de la structure.</p> <p>Les échanges de ce type sont appelés <strong>direct</strong>, ils permettent de mettre en place une communication 1:1. Pour un exchange donné, uniquement la queue dont le binding correspond à la routing key du message pourra recevoir le message.</p> <h3>Les wildcards (mode Topic)</h3> <p>Ce même système de routing/binding key est poussé au point de permettre d'utiliser des wildcards dans les noms des clefs.</p> <p>Le premier type de wildcard, l’astérisque, permet de représenter un seul et unique élément dans la routing-key.<br />
Par exemple une binding key <em>stock.us.*</em> pourrait représenter toutes les routing-key commençant par <em>stock.us.</em> et se terminant par un unique élément. Par exemple <em>stock.us.vmw</em> et <em>stock.us.orcl</em> seraient valides, <em>stock.us.orcl.java</em> ne le serait pas.</p> <p>Un second caractère de wildcard, le dièse, est utilisable pour représenter une suite de plusieurs éléments.<br />
Par exemple la binding key <em>stock.#</em> peut représenter <em>stock.us.vmw</em>, <em>stock.us.orcl.java</em> ou <em>stock.fr.ead</em>.</p> <p><img src="/wp-content/uploads/2015/07/Topic.png" alt="Topic.png" style="display:block; margin:0 auto;" /></p> <p>Dans cette configuration un message dont la routing key serait <em>stock.us.orcl.java</em> serait disponible dans Queue1 et Queue2.<br />
Un message dont la routing key serait <em>stock.fr.ead</em> ne serait disponible que dans la Queue2.<br />
Si le message avait une routing key non représentée dans les bindings, tel que <em>fr.cac.ead</em>, alors le message serait ignoré et ne figurerait dans aucune queue.</p> <p>Les différents wildcards peuvent figurer n'importe ou dans la binding key; <em>stock.#.orcl</em> est donc elle aussi valide.</p> <p>Ce système d'exchange est appelé le mode <strong>topic</strong>, il permet de mettre en place un échange en 1:N basé sur des règles définies dans les binding keys.</p> <h2>Conclusion</h2> <p>AMQP est un système très souple qui en plus de sa capacité à être utilisé dans n'importe quel environnement, gère la communication avec simplicité grâce aux différents systèmes d'échanges applicables.</p> <p>Dans un prochain post, nous pourrons voir plus de détails le fonctionnement de AMQP: Comment définir le système d'échange utilisé, quelles sont les options disponibles pour les exchanges et queues, qui crée tous ces composants, etc.</p>
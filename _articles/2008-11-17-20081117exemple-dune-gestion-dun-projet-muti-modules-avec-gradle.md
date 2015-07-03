---
ID: 156
post_title: >
  Exemple d’une gestion d’un projet
  muti-modules avec Gradle
author: Grégory Boissinot
post_date: 2008-11-17 17:14:00
post_excerpt: '<p><a href="http://www.gradle.org/">Gradle</a> est un système de build complet offrant une gestion avancé d’un projet multi-modules. Prenons la structure de projet suivante&nbsp;:</p> <pre> resanet   +-- settings.gradle   +-- build.gradle shared   +-- src/main/java/com/zenika/formation/resanet/Operator.java services   +-- src/main/java/com/zenika/formation/resanet/Authenticate.java </pre> <p>Nous avons un projet nommée “resanet” constitué de deux modules (ou sous-projets) “shared” et “services”. Dans notre example, le module  “services” va dépendre du module “shared” à la compilation. La première constatation est qu’il n’est pas obligatoire d’avoir un script de build par module. Dans le cas de projet multi-modules, seul le descripteur Gradle racine (build.gradle) est requis. La philosophie de Gradle est d’avoir un script de build uniquement si cela est nécessaire; un répertoire sous la racine du projet définit un module.</p>'
layout: post
permalink: http://blog.zenika-offres.com/?p=156
published: true
---
<p><a href="http://www.gradle.org/">Gradle</a> est un système de build complet offrant une gestion avancé d’un projet multi-modules. Prenons la structure de projet suivante&nbsp;:</p> <pre> resanet   +-- settings.gradle   +-- build.gradle shared   +-- src/main/java/com/zenika/formation/resanet/Operator.java services   +-- src/main/java/com/zenika/formation/resanet/Authenticate.java </pre> <p>Nous avons un projet nommée “resanet” constitué de deux modules (ou sous-projets) “shared” et “services”. Dans notre example, le module  “services” va dépendre du module “shared” à la compilation. La première constatation est qu’il n’est pas obligatoire d’avoir un script de build par module. Dans le cas de projet multi-modules, seul le descripteur Gradle racine (build.gradle) est requis. La philosophie de Gradle est d’avoir un script de build uniquement si cela est nécessaire; un répertoire sous la racine du projet définit un module.</p>
<!--more-->
<p>Un build Gradle est constitué de trois phases&nbsp;: une phase d’initialisation, une phase de configuration et une phase d’exécution. C’est la phase d’initialisation de Gradle qui permet de déterminer quels projets vont intervenir durant le build. Pour chaque module ou projet de build, un objet Project sera crée. Cette phase d’initialisation s’appuie sur le fichier “settings.gradle” avec le contenu suivant&nbsp;:</p> <pre class="groovy code groovy" style="font-family:inherit"><ol><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">include <span style="color: #ff0000;">&quot;shared&quot;</span>, <span style="color: #ff0000;">&quot;services&quot;</span></div></li></ol></pre> <p>Explorons maintenant la définition du fichier de build "build.gradle"&nbsp;:</p> <pre class="groovy code groovy" style="font-family:inherit"><ol><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">project.<span style="color: #006600;">group</span> <span style="color: #66cc66;">=</span> <span style="color: #ff0000;">'com.zenika.formation'</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">project.<span style="color: #006600;">version</span> <span style="color: #66cc66;">=</span> <span style="color: #ff0000;">'1.0'</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">allprojects <span style="color: #66cc66;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	usePlugin<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">'java'</span><span style="color: #66cc66;">&#41;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	manifest.<span style="color: #006600;">mainAttributes</span><span style="color: #66cc66;">&#40;</span>provider: <span style="color: #ff0000;">'gradle'</span><span style="color: #66cc66;">&#41;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	sourceCompatibility <span style="color: #66cc66;">=</span> <span style="color: #cc66cc;">1.5</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	targetCompatibility <span style="color: #66cc66;">=</span> <span style="color: #cc66cc;">1.5</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #66cc66;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">project<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">':services'</span><span style="color: #66cc66;">&#41;</span><span style="color: #66cc66;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	dependencies<span style="color: #66cc66;">&#123;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">		compile project<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">':shared'</span><span style="color: #66cc66;">&#41;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #66cc66;">&#125;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #66cc66;">&#125;</span></div></li></ol></pre> <p>Vous remarquez que Gradle permet depuis un script de build d’accéder à n’importe autre script de build grâce à son mécanique d’injection de configuration. Cette fonctionnalité est très puissante comparé à Maven qui ne supporte que l’héritage.</p> <p>Si vous invoquez la commande de compilation au niveau du projet racine&nbsp;:</p> <pre> /usr/local/projects/gradle/resanet&gt; gradle compile </pre> <p>vous constaterez que le sous-projet “shared” puis le sous-projet “services” sont compilés</p> <p>Mais ce qui est important, c’est que si on se place dans le sous-projet “services”, et on invoque la commande de compilation</p> <pre> /usr/local/projects/gradle/resanet/services&gt; gradle compile </pre> <p>vous constaterez que le projet “shared” puis le projet “services” sont compilés. Pour ceux qui viennent du monde Maven, Gradle révolutionne le build d’un projet multi-modules. Avec Maven, vous auriez l’aborescence suivante&nbsp;:</p> <pre> resanet   +-- pom.xml shared   +-- src/main/java/com/zenika/formation/resanet/Operator.java   +-- pom.xml services   +-- src/main/java/com/zenika/formation/resanet/Authenticate.java   +-- pom.xml </pre> <p>Et si vous placez dans le module "services", seul celui-ci est compilé. La librairie shared aurait du être pré-construire et déployé dans un repository Maven (au moins local). En Maven, la compilation simultané du module "shared" et "services" n’est possible que depuis la racine du projet</p> <pre> /usr/local/projects/maven/resanet&gt; mvn compile </pre> <p>Néanmoins, Gradle est encore un produit jeune et toutes les fonctionnalités ne sont pas encore présentes. En particularité la fonctionnalité de build incrémental. Par défaut, avec Maven ou avec Gradle, si on modifie le projet “shared” et qu’on invoque de nouveau la commande de compilation au niveau du projet racine, seule le projet “shared” est recompilé.<br />
Le module “services” n’est pas recompilé car son code source n’a pas changé. Vous avez alors un projet potentiellement incohérent car le module “services” (qui dépend de “shared”) peut ne pas recompiler.<br />
En pratique,  on force une recompilation complète avec la commande qui consiste a détruitre les précédents résultats de compilation, puis à compiler tout le projet multi-modules.</p> <pre> /usr/local/projects/maven/resanet&gt; mvn clean compile /usr/local/projects/gradle/resanet&gt; gradle clean compile </pre> <p>Dans le cadre de Maven, l’autre possibilité est d’utiliser le plugin Maven <a href="https://maven-incremental-build.dev.java.net/">maven-incremental-plugin</a> permettant une suppression du répertoire de compilation pour les modules ayant subit une modication et les sous-projets ayant pour dépendances les modules modifiés. Pour Gradle, cette fonctionnalité sera prise en compte <a href="http://jira.codehaus.org/browse/GRADLE-66">très prochainement</a>.</p>
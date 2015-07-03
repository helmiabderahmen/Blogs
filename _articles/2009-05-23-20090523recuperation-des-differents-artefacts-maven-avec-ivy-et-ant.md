---
ID: 166
post_title: >
  Récupération des différents artefacts
  Maven avec Ivy et Ant
author: Grégory Boissinot
post_date: 2009-05-23 13:34:00
post_excerpt: |
  <p><a href="http://ant.apache.org/ivy/">Ivy</a> est outil de gestion de dépendances très utilisé et très flexible. Parmis ses fonctionnalités, nous allons illustrer son utilisation pour récupérer des artefacts dans un repository distant de type <a href="http://maven.apache.org/">Maven</a>. Dans l'exemple de ce billet, nous utiliserons Ivy avec le builder <a href="http://ant.apache.org/">Ant</a>.</p> <p>Dans le cadre d'un repository distant de type Maven, il est très simple de récupérer l'artefact binaire avec ses dépendances transitives, grâce à la lecture par Ivy du descripteur Maven. Néanmoins, il est souvent méconnu de savoir comment récupérer depuis Ivy les artefacts des sources, les artefacts de la javadoc ou un artefact d'un type ou d'un classifier particulier.</p>
layout: post
permalink: http://blog.zenika-offres.com/?p=166
published: true
---
<p><a href="http://ant.apache.org/ivy/">Ivy</a> est outil de gestion de dépendances très utilisé et très flexible. Parmis ses fonctionnalités, nous allons illustrer son utilisation pour récupérer des artefacts dans un repository distant de type <a href="http://maven.apache.org/">Maven</a>. Dans l'exemple de ce billet, nous utiliserons Ivy avec le builder <a href="http://ant.apache.org/">Ant</a>.</p> <p>Dans le cadre d'un repository distant de type Maven, il est très simple de récupérer l'artefact binaire avec ses dépendances transitives, grâce à la lecture par Ivy du descripteur Maven. Néanmoins, il est souvent méconnu de savoir comment récupérer depuis Ivy les artefacts des sources, les artefacts de la javadoc ou un artefact d'un type ou d'un classifier particulier.</p>
<!--more-->
<p>Nous allons utiliser pour notre exemple les artefacts du plugin gradle de Hudson, localisés dans le repository Maven <a href="http://download.java.net/maven/2/">java.net</a>. Nous récupérerons <a href="http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/">la version 1.2</a>. Dans le cadre du builder Maven, chaque projet produit un seul artefact. Des notions supplémentaires de types et de classifiers (ou de natures) permettent de produire plusieurs livrables par projet. En Ivy, c'est légèrement différent avec pour un module, la production possible de plusieurs artefacts.</p> <p>La liste des artefacts à récupérer sont aux adresses suivantes</p> <ul> <li><a href="http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2-javadoc.jar">http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2-javadoc.jar</a></li> <li><a href="http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2-sources.jar">http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2-sources.jar</a></li> <li><a href="http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2.hpi">http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2.hpi</a></li> <li><a href="http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2.jar">http://download.java.net/maven/2/org/jvnet/hudson/plugins/gradle/1.2/gradle-1.2.jar</a></li> </ul> <p>Nous avons l'artefact des sources, l'artefact de la documentation javadoc, un binaire avec une extension "jar" et un binaire avec une extension "hpi". Nous avons donc en Ivy la valeur "org.jvnet.hudson.plugins" pour l'organisation, la valeur "gradle" pour le nom du module et la valeur "1.2" pour la version.</p> <p>Pour notre besoin, nous devons réaliser les étapes suivantes</p> <ol> <li>Écriture du fichier de configuration globale de Ivy  spécifiant le repository java.net</li> <li>Écriture du descripteur Ivy décrivant les module avec les différents artefacts à récupérer.</li> <li>Écriture du script de build Ant pour mettre en oeuvre Ivy</li> <li>Invocation du script Ant pour la récupération des artefacts</li> </ol> <h4>Écriture du fichier de configurations globale de Ivy</h4> <pre class="xml code xml" style="font-family:inherit"><ol><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;?xml</span> <span style="color: #000066;">version</span>=<span style="color: #ff0000;">&quot;1.0&quot;</span><span style="color: #000000; font-weight: bold;">?&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;ivysettings<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;settings</span> <span style="color: #000066;">defaultResolver</span>=<span style="color: #ff0000;">&quot;java.net2&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;resolvers<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;ibiblio</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;java.net2&quot;</span> <span style="color: #000066;">root</span>=<span style="color: #ff0000;">&quot;http://download.java.net/maven/2/&quot;</span> <span style="color: #000066;">m2compatible</span>=<span style="color: #ff0000;">&quot;true&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/resolvers<span style="color: #000000; font-weight: bold;">&gt;</span></span></span> </div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/ivysettings<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li></ol></pre> <p>Nous spécifions que Ivy va rechercher et récupérer par défaut les dépendances dans le repository java.net de type Maven2.</p> <h4>Écriture du descripteur Ivy décrivant le module avec ses artefacts à récupérer.</h4> <pre class="xml code xml" style="font-family:inherit"><ol><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #808080; font-style: italic;">&lt;!-- fichier ivysettings.xml --&gt;</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;ivy-module</span> <span style="color: #000066;">version</span>=<span style="color: #ff0000;">&quot;2.0&quot;</span>  <span style="color: #000066;">xmlns:m</span>=<span style="color: #ff0000;">&quot;http://ant.apache.org/ivy/maven&quot;</span><span style="color: #000000; font-weight: bold;">&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"> <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;info</span> <span style="color: #000066;">organisation</span>=<span style="color: #ff0000;">&quot;net.boissinot&quot;</span> <span style="color: #000066;">module</span>=<span style="color: #ff0000;">&quot;artifact-example&quot;</span> <span style="color: #000000; font-weight: bold;">&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;description<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	Complex Artifact Example</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/description<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"> <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/info<span style="color: #000000; font-weight: bold;">&gt;</span>
</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"> <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;configurations<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;conf</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;bin&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;conf</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;source&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;conf</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;javadoc&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"> <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/configurations<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"> <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;dependencies<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;dependency</span> <span style="color: #000066;">org</span>=<span style="color: #ff0000;">&quot;org.jvnet.hudson.plugins&quot;</span>  <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;gradle&quot;</span> <span style="color: #000066;">rev</span>=<span style="color: #ff0000;">&quot;1.0&quot;</span> 		</span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">		        <span style="color: #000066;">conf</span>=<span style="color: #ff0000;">&quot;bin,sources,javadoc-&gt;</span></span>master&quot;&gt; </div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;artifact</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;gradle&quot;</span> <span style="color: #000066;">conf</span>=<span style="color: #ff0000;">&quot;bin&quot;</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;artifact</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;gradle&quot;</span> <span style="color: #000066;">type</span>=<span style="color: #ff0000;">&quot;hpi&quot;</span> <span style="color: #000066;">conf</span>=<span style="color: #ff0000;">&quot;bin&quot;</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;artifact</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;gradle&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">                      <span style="color: #000066;">type</span>=<span style="color: #ff0000;">&quot;source&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">		      <span style="color: #000066;">ext</span>=<span style="color: #ff0000;">&quot;jar&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">                      <span style="color: #000066;">m:classifier</span>=<span style="color: #ff0000;">&quot;sources&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">		      <span style="color: #000066;">conf</span>=<span style="color: #ff0000;">&quot;source&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;artifact</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;gradle&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">                     <span style="color: #000066;">type</span>=<span style="color: #ff0000;">&quot;javadoc&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">		     <span style="color: #000066;">ext</span>=<span style="color: #ff0000;">&quot;jar&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">                     <span style="color: #000066;">m:classifier</span>=<span style="color: #ff0000;">&quot;javadoc&quot;</span> </span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;">	             <span style="color: #000066;">conf</span>=<span style="color: #ff0000;">&quot;javadoc&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal;
margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/dependency<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/dependencies<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/ivy-module<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li></ol></pre> <p>Premièrement, nous déclarons 3 configurations nommées "bin", "source" et "javadoc". Chaque configuration va contenir un ensemble de fichiers définissant un classpath. Nous souhaitons que la configuration "bin" contienne les artefacts binaires, la configuration "source" contienne les artefacts de sources et la configuration "javadoc" contienne les artefacts de javadoc. En terme de post traitement, il est recommandé de faire des espaces différents.</p> <p>Puis nous déclarons notre dépendance vers le module Gradle avec explicitement les artefacts du module.</p> <h5>Détail des dépendances</h5> <p>Au niveau du module en dépendance, nous avons la configuration suivante</p> <pre> conf=&quot;bin,sources,javadoc-&gt;master&quot; </pre> <p>Côté gauche de la "→", nous spécifions les configurations de notre module courant. Ce sont les configurations qui vont contenir les artefacts récupérés. Côté droit, nous spécifions la configuration du module cible. Dans le cadre de Maven, la clé logique "master" spécifie que nous souhaitons récupérer uniquement l'artefact.</p> <p><ins>Remarque</ins>: pour récupérer l'artefact et ses dépendances transitives de compilation et de d'exécution, nous aurions utiliser la configuration "default".</p> <h5>Détail des artefacts</h5> <p>Pour chaque élément &lt;artifact&gt;, nous devons préciser</p> <ul> <li>Un attribut "name"  décrivant le nom de l'artefact Ivy. En maven, ce nom est toujours le même pour un module donné.</li> <li>Un attribut "conf" sélectionnant la configuration dans la quel sera stockée l’artefact. L'élément est optionnel si nous avions spécifié une configuration par défaut.</li> <li>Un attribut "type". La valeur par défaut est "jar".</li> <li>Un attribut d'extension "ext" qui a pour défaut la valeur de l’attribut "type".</li> <li>Et un classifier si besoin. Il est importé depuis l’espace de noms Maven. Il permet de sélectionner notre artefact Maven avec le bon classifier.</li> </ul> <h4>Écriture du script de build Ant</h4> <pre class="xml code xml" style="font-family:inherit"><ol><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;project</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;example&quot;</span> <span style="color: #000066;">default</span>=<span style="color: #ff0000;">&quot;ivy-retrieve&quot;</span> <span style="color: #000066;">xmlns:ivy</span>=<span style="color: #ff0000;">&quot;antlib:org.apache.ivy.ant&quot;</span><span style="color: #000000; font-weight: bold;">&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;settings.dir&quot;</span> <span style="color: #000066;">location</span>=<span style="color: #ff0000;">&quot;settings&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;ivy.lib.dir&quot;</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">&quot;lib&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;target</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;ivy-init&quot;</span><span style="color: #000000; font-weight: bold;">&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;ivy:settings</span> <span style="color: #000066;">file</span>=<span style="color: #ff0000;">&quot;${settings.dir}/ivysettings.xml&quot;</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/target<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;target</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;ivy-resolve&quot;</span> <span style="color: #000066;">depends</span>=<span style="color: #ff0000;">&quot;ivy-init&quot;</span><span style="color: #000000; font-weight: bold;">&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;ivy:resolve</span> <span style="color: #000066;">file</span>=<span style="color: #ff0000;">&quot;${basedir}/ivy.xml&quot;</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/target<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;target</span> <span
style="color: #000066;">name</span>=<span style="color: #ff0000;">&quot;ivy-retrieve&quot;</span> <span style="color: #000066;">depends</span>=<span style="color: #ff0000;">&quot;ivy-resolve&quot;</span><span style="color: #000000; font-weight: bold;">&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;ivy:retrieve</span> <span style="color: #000066;">pattern</span>=<span style="color: #ff0000;">&quot;${ivy.lib.dir}/[type]s/[artifact]-[revision](-[classifier]).[ext]&quot;</span> <span style="color: #000066;">sync</span>=<span style="color: #ff0000;">&quot;true&quot;</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/target<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;">&nbsp;</div></li><li style="font-weight: normal;"><div style="font-family: monospace; font-weight: normal; font-style: normal; margin:0; padding:0; background:inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/project<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></div></li></ol></pre> <p>Nous avons trois cibles Ant.</p> <ul> <li>La cible Ant "ivy-init" va initialiser et charger notre fichier Ivy de configuration globale déclarant le repository java.net.</li> <li>La cible Ant "ivy-resolve" va analyser le fichier descripteur Ivy "ivy.xml" et résoudre toutes les dépendances. Ces dépendances seront téléchargées dans le cache local de Ivy. L'emplacement du cache est spécifié à travers la propriété "ivy.cache.dir"; sa valeur par défaut est "USER_HOME/.ivy2".</li> <li>La cible Ant "ivy-retrieve" va copier les artefacts dans un réperoire définit par la propriété "ivy.lib.dir" avec pour organisation le pattern "[type]s/[artifact]-[revision](-[classifier]).[ext]". Nous allons donc avoir par exemple "lib/javadocs/gradle-1.2-javadoc.jar"</li> </ul> <p>Et nous spécifions que la cible "ivy-retrieve" sera invoquée par défaut.</p> <h4>Invocation de la récupération des artefacts</h4> <p>Le simple appel de la commande ant invoquera la cible par défaut "ivy-retrieve", déclenchant l'initialisation de la configuration, la récupération des artefacts dans le cache Ivy, puis la copie de ces artefacts dans le répertoire définit par la propriété "ivy.lib.dir".</p>
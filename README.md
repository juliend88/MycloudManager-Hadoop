# Instrumentaliser un cluster Hadoop avec MyCloudManager

![mycloudmanagerimg](img/mycloudmanager.png) ![hadooplogo](img/hadoop-logo.png)

## Lancement des Stacks

Ce tutoriel à pour but de vous montrer comment instrumentaliser votre cluster hadoop avec la stack MyCloudManager.

Pour commencer assurez vous d'avoir lancé les stacks dans votre tenant. Si ce n'est pas le cas, je vous invite à regarder les articles suivant :

* Démarrer MyCloudManager : http://dev.cloudwatt.com/fr/blog/innovation-beta-mycloudmanager-v2.html

* Démarrer un cluster Hadoop : http://dev.cloudwatt.com/fr/blog/deployer-votre-service-hadoop-as-a-service-au-travers-d-un-stack-heat.html


## Démarrer l'instrumentalisation

Pour pouvoir déployer les applications de MyCloudManager, il faut que celui-ci puisse communiquer avec les instances et donc qu'elles soientt attachées au même réseau que votre stack MyCloudManager.

Dans le cadre du déploiement du cluster hadoop, il nous est possible de choisir le réseau sur lequel celui-ci sera déployé. Voici la procédure :

* Connectez-vous à l'interface web de votre stack hadoop via la flotting-ip donnée en sortie de votre stack ainsi que vos indentifiants Cloudwatt.

![sortiehadoop](img/Sortiestackhadoop.png)
![hadoopauth](img/hadoopauth.png)

* Vérifiez que vous vous trouvez bien sur le tenant où vous avez créé MyCloudManager ainsi que la région. Dans cet exemple : tenant = `0750177966_@_1449053663`, region= `FR1`.

![tenantregion](img/tenantregion.png)

* Commencez la création de votre cluster, rendez-vous dans la partie `Data Processing` puis `Clusters` de votre console Hadoop ensuite cliquez sur `Launch Cluster` et enfin choisissez le type de cluster ainsi que la version. Dans cet exemple : plugin `Vanilla` version 2.7.1.

![launchcluster](img/launchcluster.png)

* Sur le deuxième volet, renseignez les paramètres que vous souhaitez donner à votre cluster. Surtout dans la partie `Neutron Management Network` renseignez ici le réseau de votre MyCloudManager puis démarrez la création.

![paramcluster](img/paramscluster.png)

* Une fois votre cluster créé, vous pouvez maintenant vous connecter en *ssh* sur l'ensemble des instances qui le compose et copier la commande `curl` qui vous est donnée lors de l'ajout d'une instance dans MyCloudManager.

![addsrv](img/addsrv.png)

* Apres avoir attaché les instances, vous pouvez à présent lancer le déployement des applications comme expliqué dans l'article MyCloudManager.

  http://dev.cloudwatt.com/fr/blog/innovation-beta-mycloudmanager-v2.html

* Cliquez sur les icônes se situant à droite de chaque instance afin d'installer les services correspondants comme Zabbix (pour la supervision, le monitoring et l'alerting), Ntp, GrayLog (pour le log management)... Si l'application n'est pas bien installée sur les nœuds, pensez à copier le contenu du fichier ~/.ssh/authorized_keys2 à ~/.ssh/authorized_keys et réinstallez-la.

* Les instances sont maintenant bien monitorées par le MyCloudManager.

![instanceok](img/instanceok.png)

## Monitoring des métriques des nœuds du cluster hadoop par Zabbix

Zabbix est bien configuré sur les nœuds sans avoir besoins de modifications. Les métriques des nœuds comme la charge CPU, l'utlisation de la RAM ou encore des disques ... sont visibles sur l'interface.

![Zabbix](img/Zabbix.png)

## Monitoring des logs des nœuds du cluster hadoop par Graylog

L'application Log Management (Graylog) est configurée par défaut pour monitorer les logs de syslogs des noeuds en utilisant des agents rsyslog. Cependant, les noeuds de cluster Hadoop ont d'autres logs à gérer comme les logs de Datanodes, les logs de Namenode, les logs de Yarn, ... Afin de surveiller proprement les logs d'un cluster Hadoop, nous avons besoin d'ajouter ces logs aux configurations rsyslog pour qu'ils puissent les envoyer à Graylog.

### Mise en place de la configuration

* Se connecter aux noeuds avec les droits root de votre user
* Vérifier la version de rsyslog

~~~bash
rsyslog -version
~~~

* Mettre à jour la version de rsyslog si elle est inférieure à 8.15.0

~~~bash
wget http://rpms.adiscon.com/v8-stable/rsyslog.repo -O /etc/yum.repos.d/rsyslog.repo
yum update rsyslog
/etc/init.d/rsyslogd restart
~~~

* Modifier la configuration de rsyslog: Les fichiers .conf de rsyslog se trouvent dans /etc/rsyslog.d/. Pour chaque log de hadoop à surveiller, nous ajoutons un fichier .conf en précisant le chemin de logs. Ci-dessous un exemple de configuration pour les logs de Yarn.

~~~bash
module(load="imfile" mode="inotify")
$InputFileName /var/log/hadoop-yarn/hadoop-*.out
$InputFileTag yarn:
$InputRunFileMonitor
~~~

* Visualiser les logs via l'interface de Graylog accessible depuis votre MyCloudManager

![graylog](img/Graylog.png)

---
Have Fun and Hack in Peace

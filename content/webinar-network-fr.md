Title: Gérer des équipments IOS avec Ansible
Category: Ansible
Date: 2020-05-15
Lang: fr
Slug: webinar-05-2020

Ci dessous un petit résumé de la demo que j'ai effecuté lors de notre dernier webinar Red Hat "Automatiser au dela de Linux".   

Le code de la demo réseau est à:  
[https://github.com/automaticdavid/webinar_052020](https://github.com/automaticdavid/webinar_052020)

Le workshop sécurité est decrit à:   
[https://ansible.github.io/workshops/](https://ansible.github.io/workshops/)

Le replay du webinar:  
[L’automatisation au delà des systèmes Linux](https://www.redhat.com/en/events/webinar/red-hat-ansible-automation-platform-cloud-reseau-iot-lautomatisation-au-dela-des-systemes-linux)


## 1- Utiliser les facts & ios_command

![webinar1_demo_1]({static}/images/webinar1_demo_1.gif)

Dans cette première partie on regarde l'inventaire utilisé. Dans le playbook `gather_ios_data.yml` j'utilise `gather_facts: yes` pour les équipements réseau, ce qui est nouveau avec Ansible 2.9. Cela me permet d'utiliser les variables `ansible_net_*` directement. 

Ensuite j'utilise `ios_command` pour éxecuter une command IOS et récupérer le résultat, que j'utilise pour afficher la conf snmp des 4 routeurs. 

Le playbook `config_routers.yml` permet ensuite de changer cette config snmp. Notez que lors de la 2ème éxecution, Ansible affiche `changed=0` et toutes les machines sont en "OK vert". C'est l'indempotence: les equipements sont déjà dans l'état que l'on veut et Ansible ne fait rien quand il n'y a rien à faire.

Ensuite je me connecte directement à un switch et je fais un changement manuel. Quand j'execute à nouveau mon playbook de config, seul ce changement là est corrigé. 


## 2- Resource modules & Source of truth. 

![webinar1_demo_2]({static}/images/webinar1_demo_2.gif)

Pour aller un peu plus loin j'utilise ensuite le module `ios_facts` et avec Ansible 2.9 cela permet d'extraire la configuration des swtiches existants sous la forme de données structurées. Quand j'execute le playbook `gather_ios_resource.yml` on voit que la configuration des interfaces L3 est affichée sous la forme d'une liste d'interfaces en YAML: 

![webinar1_demo_2]({static}/images/webinar1_demo2_list.png)

Chaque élément de la liste (marqué par un tiret) est un dictionnaire qui contient des clés: dans cet exemple "ipv4" et "name". La clé "ipv4" contient une liste, qui ici n'a qu'un seul élément. Cet élément est aussi un dictionaire, qui n'a qu'une seule clé: "address". 
Avec un peu d'habitude cette façon de représenter la configuration permet de lire et modifier les valeurs de façon simple et robuste.

Le playbook écrit ces valeurs dans le repertoire `host_vars`. Dans la demo je change une valeur de configuration dans le fichier qui décrit le routeur rtr1 et j'execute le playbook `config_l3.yml` en mode "check". Ce mode fait qu'Ansible dit quels changements il ferait sans vraiment les faire. J'execute ensuite le changement et je vérifie qu'il a bien eu lieu avec le playbook `gather_ios_resource.yml`. Ce dernier ne modifie pas la source de verité car elle est deja à jour!

Ce que cela montre c'est que avec le module `ios_facts`, Ansible est capable de produire des données structurées à partir de la configuration existante sur les switchs. Et qu'avec le module  `ios_l3_interfaces` on est capable de ré-utiliser ces données directement, pour modifier la configuration des équipements. Comme la configuration est en YAML, on peut la gérer comme du code dans git. Cela signifie qu'on a une traçabilité totale des changements, qu'on peut ré-appliquer n'importe quelle version, ou encore qu'on peut manager l'infrastructure réseau avec un gitflow. 


Plus d'info sur les resources modules dans cette conférence donnée par [@IPvSean](https://twitter.com/ipvsean?lang=en):  
[https://www.ansible.com/2019-ansible-network-automation-resource-modules](https://www.ansible.com/2019-ansible-network-automation-resource-modules) 

   





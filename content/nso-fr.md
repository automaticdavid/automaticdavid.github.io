Title: Interop Ansible & Cisco NSO
Category: Ansible
Date: 2020-06-05
Lang: fr
Slug: nso-06-2020

Ansible est très performant pour automatiser les équipements réseau. Mais parfois on a déjà une infra [Cisco NSO](https://www.cisco.com/c/en/us/solutions/service-provider/solutions-cloud-providers/network-services-orchestrator-solutions.html) en place. Les modules Ansible pour NSO permettent d'utiliser les deux solutions en cote à cote, de passer de l'une à l'autre de façon simple et finalement de rendre NSO plus interessant.

Le code utilisé est à:  
[https://github.com/automaticdavid/ansible-nso](https://github.com/automaticdavid/ansible-nso)

Vous pouvez facilement tester en utilisant le lab NSO proposé par Cisco:  
[https://devnetsandbox.cisco.com/RM/Topology](https://devnetsandbox.cisco.com/RM/Topology)


## 1- Piloter le sync NSO

![NSO]({static}/images/nso1.gif)

Dans cette première partie on commence par executer le playbook `check_sync.yml` qui permet de vérifier que les switches présents dans l'infra sont synchronisés avec la base NSO. C'est un exemple d'utilisation du module ansible `nso_action` qui permet de déclencher des actions NSO. On l'execute d'abord pour un switch nommé et ensuite on check le sync pour tous les equipements. 

Ensuite j'exécute le playbook `show_address.yml`: c'est un exemple d'utilisation du module `nso_query` qui permet de faire une requête XPATH contre NSO, et que j'utilise pour afficher les IP des mes équipements. 

Je me connecte ensuite à un switch et je modifie la configuration en direct (le lab Cisco est configuré avec seulement telnet). Exécuter de nouveau `check_sync.yml` montre alors que l'équipement en question est "out-of-sync". Il devient facile de gérer cette exception dans un playbook, en ouvrant un tiquet dans un ITSM par exemple ou en déclenchant une action de remédiation. 

Cette action peut être par exemple le playbook `sync_to.yml` que j'exécute ensuite, qui effectue un sync-to NSO: il pousse la configuration NSO vers le switch. J'exécute ensuite le `check_sync.yml` de nouveau pour vérifier que tout est synchrone. 


## 2- Source Of Truth avec NSO 

![NSO]({static}/images/nso2.gif)

Dans cette deuxième étape on commence par utiliser le module `nso_verify` pour effectuer un backup de la conf NSO. Ce qu'on obtient c'est une copie local des config des équipements, sous forme de YAML. On peut donc utiliser ces fichier comme "source de vérité" de l'infrastructure, en la stockant dans git par exemple. 

Je continue en modifiant une information de configuration pour un des équipements dans son fichier de description. J'exécute ensuite le playbook `apply_sot.yml`. La première tache lit les infos de la "source of truth" et les applique à la configuration NSO en utilisant le module `nso_config`. On fait ensuite un `nso_verify` qui montre qu'il y a bien une violation correspondant à la modification que nous avons effectué, c'est à dire que l'équipement n'est pas synchro avec la config NSO:

![NSO]({static}/images/nso3.png)

On effectue donc un sync-to dans la tache "Apply SOT" et un dernier "Check sync" confirme que l'équipement a été synchronisé avec la source de verité. 

On peut aussi vérifier un élément de configuration en particulier, c'est ce que montre le playbook `verify_config.yml`: on vérifie juste un sous ensemble de la config NSO. Cela peut être interessant par exemple si on veut vérifier un même élément de config mais sur un très grand nombre d'équipements. 

Enfin j'exécute le playbook `apply_sot.yml` une seconde fois: il ne se passe rien et "changed" est à 0. En effet, le playbook est indempotent et Ansible ne fait rien s'il n'y a rien à faire. Avec `--check-mode` cela permet de vérifier facilement la conformité de la config NSO. 

Pour finir je me connecte à l'équipement pour vérifier que la modification faite dans la source of truth a bien été appliquée.


## 3- Resource Modules Ansible & NSO

![NSO]({static}/images/nso4.gif)
   
Dans cette dernière partie, on utilise un des modules réseau d'Ansible pour changer la config NXOS directement sur le switch. Pour utiliser Ansible, j'ai activé ssh sur un des équipement et créé un fichier inventory. Le module utilisé est `nxos_vlan`: 

```
- name: Direct device change
  nxos_vlans:
    config:
      - vlan_id: 501
        name: ansible
    state:  merged
  register: r_vlan
```

On voit que le module remonte les états "before" et "after" pour le changement. 

Manipuler les conf des équipements réseau avec Ansible peut être plus simple qu'avec NSO. 

Après le changement, on effectue une vérification de la synchro NSO: évidemment il y a une violation. Le playbook effectue alors un sync_from pour resynchroniser NSO depuis la configuration de l'équipement. On est donc capable d'utiliser NSO depuis Ansible, mais aussi Ansible et NSO en "cote à cote".

 



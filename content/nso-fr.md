Title: Interop Ansible & Cisco NSO
Category: Ansible
Date: 2020-06-05
Lang: fr
Slug: nso-06-2020

Ansible est très performant pour automatiser les equipement réseau. Mais parfois les clients ont deja une infra [Cisco NSO](https://www.cisco.com/c/en/us/solutions/service-provider/solutions-cloud-providers/network-services-orchestrator-solutions.html) en place. Les modules Ansible pour NSO permettent d'utiliser les deux solutions en cote à cote, de passer de l'une à l'autre de façon simple et finalement de rendre NSO plus interessant.

Le code utilisé est à:  
[https://github.com/automaticdavid/ansible-nso](https://github.com/automaticdavid/ansible-nso)

Vous pouvez facilement tester en utilisant, comme moi, le lab NSO proposé par Cisco:
[https://devnetsandbox.cisco.com/RM/Topology](https://devnetsandbox.cisco.com/RM/Topology)


## 1- Piloter le sync NSO

![NSO]({static}/images/nso1.gif)

Dans cette première partie on commence par executer le playbook `check_sync.yml` qui permet de verifier que les switches presents dans l'infra sont synchronisés avec la base NSO. C'est un exemple d'utilisation du module ansible `nso_action` qui permet de declencher des actions NSO. On l'execute d'abord pour un switch nommé et ensuite on check le sync pour tous les equipements. 

Ensuite j'execute le playbook `show_address.yml`: c'est un exemple d'utilisation du module `nso_query` qui permet de faire une requete XPATH contre NSO, et que j'utilise pour afficher les IP des mes equipements. Je me connecte ensuite à un switch et je modifie la configuration en direct. Executer de nouveau `check_sync.yml` montre alors que l'equipement en question est "out-of-sync". Il devient facile de gerer cette exception dans un playbook, en ouvrant un tiquet dans un ITSM par exemple ou en declenchant une action de remediation. 

Cette action peut etre par exemple le playbook que j'execute ensute, qui effectue un sync-to NSO: il pousse la configuration NSO vers le switch. J'execute ensuite le `check_sync.yml`de nouveau pour verifier que tout est synchrone. 


## 2- Source Of Truth avec NSO 

![NSO]({static}/images/nso2.gif)

Dans cette deuxieme etape on commence par utiliser le module `nso_verify` pour effectuter un backup de la conf NSO. Ce qu'on obtient c'est une copie local des config des equipements, sous forme de YAML. On peut donc utiliser ces fichier comme "source de verité" de l'infrastructure, en la stockant dans git par exemple. 

Je continue en modifiant une information de configuration pour un des equipements dans son fichier de description. J'excecute ensuite le playbook `apply_sot.yml`. La premiere tache lit les infos de la "source of truth" et les applique à la configuration NSO en utilisant le module `nso_config`. On fait ensuite un `nso_verify` qui montre qu'il y a bien une violation correspondant à la modification que nous avons effectuée, c'est à dire que l'equipement n'est pas synchro avec la config NSO:

![NSO]({static}/images/nso3.png)

On effectue donc un sync-to dans la tache "Apply SOT" et un dernier "Check sync" confirme que l'equipement a a été synchronisé avec la source de verité. 

On peut aussi verifier un element de configuration particulier, c'est ce que montre le playbook `verify_config.yml`: on verifie juste un sous ensemble de la config NSO. Cela peut être interessant par exemple si on veut verifier un meme element de config mais sur un très grand nombre d'equipement. 

Enfin j'execute le playbook `apply_sot.yml` une seconde fois: il ne se passe rien et "changed" est à 0. en effet, le playbook est indempotent est Ansible ne fait rien s'il n'y a rien a faire. Avec `--check-mode` cela permet de verifier facilement la conformité de la config NSO. 

Pour finir je me connecte à l'equipement pour verifier que la modification faite dans la source of truth a bien été appliquée.


## 3- Resource Modules Ansible & NSO

![NSO]({static}/images/nso4.gif)
   
Dans cette derniere partie, on utilise un des modules reseau d'Ansible pour changer la config NXOS directement sur le switch. Le module utilisé est `nxos_vlan`: 

```
- name: Direct device change
  nxos_vlans:
    config:
      - vlan_id: 501
        name: ansible
    state:  merged
  register: r_vlan
```

On voit que le module remonte les etats "before" et "after" pour le changement. 

Manipuler les conf des equipements réseau avec Ansible peut être plus simple qu'avec NSO. 

Après le changement, on effectue une verification de la synchro NSO: evidement il y a une violation. Le playbook effectue alors un sync_from pour resynchroniser NSO depuis la configuration de l'equipement. On est donc capable d'utiliser NSO depuis Ansible, mais aussi Ansible et NSO en "cote à cote".

 



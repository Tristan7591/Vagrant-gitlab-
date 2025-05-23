# gitlab-https Vagrant setup

Ce projet fournit un **Vagrantfile** pour déployer automatiquement une instance GitLab CE sur Ubuntu 20.04 (focal) avec HTTPS (certificat auto-signé).

---

## Prérequis

- [Vagrant](https://www.vagrantup.com/) (>= 2.2)
- [VirtualBox](https://www.virtualbox.org/) (>= 6.0)
- (Optionnel) Un couple de clés SSH `gitlab_id_ed25519`/`gitlab_id_ed25519.pub` placé à la racine du projet, pour injecter automatiquement votre clé publique dans l’utilisateur `root` de GitLab.

---

## Installation et démarrage

**Cloner ce dépôt**  
   ```bash
   git clone <URL_DU_DÉPÔT>
   cd <NOM_DU_RÉPERTOIRE>

Rappel des commandes vagrant:
 - vagrant validate : pour valider la syntaxe vagrant / ruby
 - vagrant up : pour lancer la vm 
 - vagrant ssh : pour ce connecter a l'instance        suivie de sudo -i pour ce connecter en root
 - vagrant destroy -f : pour kill la vm

 !! Ceci et pour un lab le serveur n'est pas sécuriser !!
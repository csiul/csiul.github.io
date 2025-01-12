# CSIUL Website

Bienvenue sur le site officiel du Club de Sécurité Informatique de l’Université Laval (CSIUL). Ce site, construit avec [Hugo](https://gohugo.io/), sert de plateforme d'accueil pour le club et contient divers articles destinés à introduire les bases de la cybersécurité.

Le site utilise le thème [Hugo Theme Console](https://github.com/mrmierzejewski/hugo-theme-console), avec plusieurs modifications pour répondre aux besoins du club.

## Fonctionnalités
- **Page d'accueil** : Informations sur le club et ses activités.
- **Articles** : Ressources pour apprendre les concepts fondamentaux de la cybersécurité.

## Contribuer au site

### Prérequis
- [Hugo](https://gohugo.io/getting-started/installing/) doit être installé sur votre machine.
- [Git](https://git-scm.com/) pour cloner le dépôt.

### Étapes

1. Forkez le dépôt :

   Créez une copie du dépôt dans votre compte GitHub.

3. Clonez le dépôt :

   ```bash
   git clone https://github.com/csiul/csiul.github.io
   ```

4. Naviguez dans le répertoire du projet :

   ```bash
   cd csiul.github.io
   ```

6. Créez une branche pour vos modifications :

   ```bash
   git switch -c ajout-article-x
   ```

7. Installez les dépendances du thème :

   ```bash
   git submodule update --init
   ```
   
8. Créer un articles et modifiez le :

   ```bash
   hugo new posts/nom-de-votre-article.md
   ```
   
9. Lancez le serveur de développement Hugo :

   ```bash
   hugo server -D
   ```
   
10. Accédez au site via votre navigateur à l'adresse suivante : http://localhost:1313

13. Validez vos changements :
  
   ```bash
   git add .
   git commit -m "Description de vos modifications et ajouts"
   ```

11. Poussez votre branche :
   
   ```bash
   git push -u origin ajout-article-x
   ```

11. Créez une Pull Request (PR) :
   
   - Rendez-vous sur la page GitHub de votre fork.
   - Cliquez sur "Compare & pull request".
   - Remplissez la description de votre PR et soumettez-la.

Merci de contribuer au site du CSIUL ! Ensemble, faisons grandir la communauté de la cybersécurité.

author: Gabin Darras
summary: Git Intermediate
id: codelabs-gitIntermediate
categories: git
environments: javascript
status: draft
feedback link: github.com/gabindarras
analytics account: 0

# Git - Intermédiaire

## Présentation

Ce CodeLabs a pour but de vous présenter la grande majorité des cas qu'on peut rencontrer en travaillant avec git.

Une question ? Une difficulté ? : **gabin.darras@ineat-conseil.fr**

Pré-requis:

- Avoir **git** installé sur votre machine.
- Avoir un compte Github/Gitlab pour pouvoir forker le projet
- Être à l'aise avec les bases de git, n'hésitez pas à suivre le CodeLabs débutant dans le cas contraire.

## Récupération du projet

Forker le projet git intermediate puis le récupérer avec la commande

```
git clone le-nom-de-votre-projet-après-le-fork
```

## Rebase sans conflit

Dans le guide débutant nous avons vu comment créer une nouvelle branche, commit et push cette dernière. C'est le cas le plus simple que vous pourrez rencontrer au cours de vos différents projets. Néanmoins il existe d'autres configurations un peu moins évidente sur lesquelles nous allons nous attarder.
La première reste relativement simple, nous avons tiré une branche et nous travaillons dessus normalement (vous pouvez checkout la branch **rebase**). Entre temps la **develop** a été mise à jour par une autre personne donc nous ne sommes plus à jour avec cette dernière.

(Si vous n'avez pas suivi le cours débutant et que vous n'avez crée d'alias je vous conseille vivement d'utiliser celui ci pour un historique de commit plus clair :

```
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

Pour s'en rendre compte il suffit simplement de lancer la commande

```
git lg
```

l'information qui nous intéresse ici se situe juste avant le message de commit, il s'agit de l'état de la branche courante par rapport aux autres branches. Pour rappel nous avons une **master** qui sert à la production, une **develop** qui à la base a été tirée de **master**, et nos branches actives de développement. Dans notre cas il n'y a aucune trace de la branche **develop**, pourtant nous l'avions bien tiré de cette dernière. Il va donc falloir mettre à jour notre branche avec la nouvelle version de la **develop**

### Fetch

Le 'fetch' va permettre de synchroniser votre remote avec le répo distant pour qu'ils soient tous les 2 iso. Et oui, pour le moment votre remote ne contient pas encore la modification, voici la commande :

```
git fetch -p
```

> -p : Avant de mettre à jour les branches, git fait le tri des branches qui potentiellement n'existent plus sur la remote

### Rebase

Notre remote est bien à jour, nous avons maintenant pouvoir également mettre à jour notre branche, dans le jargons git on appelle cette opération le "rebase". Voici la commande :

```
git rebase origin/develop
```

Nous pouvons maintenant voir dans l'historique (git lg) que la branche rebase a bien été mise à jour avec le nouveau contenu de la branche **develop**. Si vous êtes nombreux à travailler sur votre projet je vous conseille de faire le rebase au moins une fois par jour, ça vous évitera de très gros conflits ! Conflits ? C'est quoi ? Ça tombe bien c'est la prochaine section :)

## Rebase avec conflit

Ce cas devient très courant avec l'augmentation de l'effectif au sein d'un projet, mieux vaut donc bien le maitriser :)
Pour faire très simple, nous sommes dans le même cas qu'avant (c'est à dire effectuer un rebase de la **develop** sur notre branche courante) sauf que cette fois-ci nous touchons à un fichier (et pour être plus précis à une ligne) qui a également été modifiée dans la **develop**. Prenons un petit exemple pour illustrer, allez sur la branche **conflict** et faites un rebase de la **develop**

Et la c'est un peu le bordel ! Avant d'essayer de resoudre le conflit nous allons d'abord installer un outil de gestion de conflit.

Nous allons donc annuler le rebase qu'on vient de faire, pour cela nous allons lancer la commande suivante :

```
git rebase --abort
```

> Si vous avez bien prêté attention aux informations remontées lors du conflit, cette commande nous a été proposée

### kdiff3

Afin de faciliter les rebase avec conflit, nous vous proposons de configurer un **mergetool** Git.
La liste des outils supportés par défaut par Git est [ici](https://git-scm.com/docs/git-config#git-config-mergetool)

[kdiff3] est l'un des outils de merge les plus populaire. Nous le prendrons en exemple pour la configuration.

> Adaptez la configuration ci-après à votre outil de gestion de merge favori

- Télécharger et installer [kdiff3]
- Indiquer à Git quel outil de merge vous allez utiliser :

```
git config --global merge.tool kdiff3
```

- Ne pas conserver les fichiers originaux aprés résolution des conflits

```
git config --global mergetool.keepbackup false
git config --global mergetool.keeptemporaries false
```

[kdiff3]: http://kdiff3.sourceforge.net/

### Résolution des conflits

Maintenant que notre outil de merge est bien installé, nous sommes prêts à résoudre tous les conflits !

Effectuer de nouveau la commande pour rebase la develop sur la branche **conflict** pour avoir le message de conflit, puis exécutez la commande suivante :

```
git mergetool
```

> Cette commande va ouvrir l'outil de merge et nous laisser la possibilité de choisir la version souhaitée

Une fois le code choisi, vous pouvez enregistrer et fermer l'outil de merge. Si il y a d'autres conflits, ils seront automatiquement détectés et vous seront proposés de la même façon.
Puis exécutez la commande :

```
git rebase --continue
```

> Cela va comme son nom l'indique continuer le rebase. Notre cas est simple car nous n'avons qu'un commit de retard mais il n'est pas rare du tout d'en avoir plusieurs, cette commande permet de passer au commit suivant.

## Merge

Le merge est utilisé, comme son nom l'indique, pour fusionner une branche avec une autre. Dans notre cas, il s'agirat de merger notre branche locale à la branche **develop**.

### Rebase vs Merge

Normalement la vous devriez vous dire "mais je ne comprends pas, pour récupérer les modifs d'une branche on utilise rebase, et la tu nous parles de merge..." et vous avez parfaitement raison ! Il y a néanmoins une petite subtilité entre les 2 (sinon il n'y aura qu'une seule commande !).

Le rebase a pour but premier de réécrire l'historique du projet (cette partie sera abordée dans le cours avancée). Lorsque vous faites un rebase de la **develop** sur votre branche local, les commits de cette dernière seront réécrit. Pour vérifier cela il suffit de regarder l'id des commits en question, après rebase ils ont changé. C'est la que le rebase pose problème pour la **develop**, en effet cette dernière est une branche protégée donc son historique ne peut à aucun moment être réécrit. C'est la que le merge entre en action !

Le merge va simplement ajouter les commits de notre branche locale à la suite des commits de la **develop**. Il y a une condition toutefois à respecter pour que le merge se passe bien, que les 2 branches soient à jour ou dit en langage git...que la branche **develop** ait bien été rebase de la branche locale ! (Attention aux noeuds au cerveau :))

### 1e merge

C'est à vous de jouer maintenant, le but étant de merger la branche "merge" sur la **develop**. Pour cela placez vous sur cette dernière et exécutez la commande suivante :

```
git merge origin/merge --no-ff
```

> De base la commande merge va rajouter un commit disant que la branche a bien été merge. Mais cette information n'est pas utile et pollue l'historique, c'est pour cela qu'on a rajouté l'option --no-ff

Nous allons maintenant modifier notre fichier de configuration globale pour éviter de devoir ajouter cette option à chaque fois

```
git config --global merge.ff only
```

## Rappel sur le pull

Par défaut quand on fait un

```
git pull
```

2 opérations sont effectuées :

```
git fetch
```

puis

```
git merge
```

Nous allons donc une nouvelle fois modifier notre fichier de configuration globale

```
git config --global pull.rebase preserve
```

> Effectuer un rebase lors des pull afin d'éviter les messages de commit pulluant de types "merged".

## Réecriture de commit

### Réécriture du dernier commit

De temps en temps nous écrivons des commits avec des messages pas toujours pertinants, nous allons voir ici comment modifier le message du dernier commit effectué.
Pour cela récupérer le contenu de la branche **amend** et exécutez la commande git lg pour regarder le dernier commit en question. Ce dernier n'est pas réglementaire d'un point de vu convention git. Pour le modifier :

```
git commit --amend
```

Il n'y a plus qu'à push la correction, une fois cette dernière effectuée.

Et la git est complétement perdu ! Ba oui vous poussez un historique qui a changé, il ne voit plus l'identifiant du dernier commit donc il n'est pas très content

> /!\ ATTENTION /!\ Dans ces cas là git vous propose de pull la branche, il ne faut ABSOLUMENT PAS le faire :)

En fait il suffit simplement de préciser que la version qu'on a est la bonne et qu'elle doit tout bonnement remplacer la version présente sur le repo distant. Voici la commande

```
git push -f
```

> -f ou --force pour "forcer" git à écraser le contenu

Il y a une règle d'or à ABSOLUMENT respecter, il ne faut jamais force push la branche **develop** ou **master** car ces dernières sont protégées et doivent avoir un historique propre. Par contre il y a aucun soucis à force push votre branche locale, c'est tout à fait commun ! Attention toutefois si vous êtes plusieurs à travailler sur la même branche :)

### Rebase intéractif

Le cas précédent n'est pas le plus fréquent quand il s'agit de réécrire notre historique de commit, en effet il ne touche qu'au dernier commit effectué. Or dans une grande majorité des cas nous allons également toucher aux autres commit. Par exemple c'est la fin de journée mais notre travail n'est pas encore fini, c'est pourquoi nous décidons de faire un commit intermédiaire pour éviter de perdre ce qu'on vient de faire. Le lendemain nous terminons ce qu'il nous restait à faire, donc nous ajoutons un 2e commit. Le soucis ici c'est qu'on se retrouve avec 2 commits qui sont censés faire exactement la même chose. Tirez la branche **rebase-interactif** et regardez son historique pour bien voir les 2 commits en question.
Exécutez la commande suivante :

```
git rebase -i e409c8f
```

> e409c8f est le SHA-1 du parent du commit à partir duquel on veut commencer à réécrire l'historique.

Plusieurs options s'offrent alors à nous, Git vous propose d'ailleurs tout un tas de commandes, mais nous n'allons n'en utiliser que 2 pour le moment: le squash et le fixup

- le squash permet de fusionner 2 commits et de modifier le message de commit
- le fixup fusionner également 2 commits mais ne permet de modifier le message.

Dans notre cas nous allons utiliser le squash, pour cela placez vous en dessous du 2e commit et remplacez "pick" par "s" ou "squash"

Modifiez le message de commit, sauvegardez et quittez. Vous devriez voir un message du style "Successfully rebased and updated refs/heads/rebase-interactif."
Pour s'assurer que le rebase s'est déroulé correctement il n'y a qu'à regarder l'historique de la branche, les 2 commits ont été remplacés par un autre commit. C'est pourquoi vous devez effectuer un force push pour écraser le veille historique du répo distant.

## À vous de jouer !

Le but de cet exercice est de simuler un projet avec 3 personnes travaillant en parallèle.
Pour cela tirez 3 nouvelles branches EN MÊME TEMPS :

- Dans la 1e vous modifierez le fichier README.md en ajoutant ce que vous avez aimé dans ce CodeLabs.
- Dans la 2e vous ajouterez une image de votre humeur actuellement.
- Dans la 3e vous modifierez le fichier README.md en ajoutant ce qui pourrait être amélioré dans ce CodeLabs.

La branche **develop** devra contenir donc 3 commits à la fin :)

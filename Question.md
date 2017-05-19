# Création d'un buildpack *markdown* custom

Un buildpack est utilisé par Cloud Foundry afin de préparer l'environnement d'exécution de l'application qui est poussée.

Nous allons automatiser la création d'un site web écrit en [markdown](https://fr.wikipedia.org/wiki/Markdown).

- Les fichiers markdown seront impérativement stockés dans un répertoire *dist* qui sera positionné à la racine de votre site 
- Au déploiement, l'ensemble des fichiers `dist/*.md` seront transformés en html en utilisant l'utilitaire [pandoc](http://pandoc.org)

Pour cela il nous faut développer un buildpack *markdown* spécifique. 2 options possibles :

- s'inspirer très fortement du buildpack staticfile standard (exercice 1)
- le faire à partir de rien (exercice 2)

# Exercice 1 : le faire à partir du buildpack staticfile standard

Téléchargez le zip du buildpack standard [StaticFile V1.3.9](https://github.com/cloudfoundry/staticfile-buildpack). (_Branch->Tag->v1.3.9_)

> Attention: Assurez-vous d'avoir téléchargé le répertoire `compile-extension` 

Après avoir lu la documentation de ce buildpack, déployez l'application de test qui est mise à votre disposition dans `AppTest`. Vérifiez que vous obtenez bien le résultat prévu par ce buildpack.

### Préambule

- modifiez le fichier `VERSION`

- Installez [buildpack-packer](https://github.com/cloudfoundry/buildpack-packager)

```sh
cd staticfile-buildpack-1.3.9
BUNDLE_GEMFILE=cf.Gemfile bundle
```

- packagez le buildpack

```sh
BUNDLE_GEMFILE=cf.Gemfile bundle exec buildpack-packager --cached
```

- On pousse le zip en lui associant un nom logique et un ordre de priorité

```sh
cf create-buildpack md_static_buildpack markdown-staticfile_buildpack-cached-v1.0.0.zip 1
```

- NB: le fichier zip a été généré par l'étape précédente, assurez-vous que le nom est le bon !

- Si le buildpack existe déja, il faut le mettre à jour

```sh
cf  update-buildpack md_static_buildpack -p ./markdown-staticfile_buildpack-cached-v1.0.0.zip  -i 0
```

- Ou le supprimer

```sh
cf delete-buildpack -f md_static_buildpack
```

- Vérifiez que le buildpack a bien été installé dans Cloud Foundry

```sh
cf buildpacks
```

### Détection

Lorqu'une application est déployée, Cloud Foundry va successivement appeler le script `bin/detect` de chaque buildpack. Le premier qui répondra le code `0` sera choisi comme buildpack pour cette application.

- Modifiez le code de `bin/detect` afin d'être adapté au cas d'utilisation décrit en préambule

- Vérifiez que ce buildpack est maintenant choisi par Cloud Foundry

### Compilation

Une fois le buildpack identifié, c'est la phase de compilation qui est lancée. Analysez et modifiez le script `bin/compile` afin que ce dernier copie le contenu de votre répertoire `dist` dans le répertoire `$build_dir/public`

Afin de vérifier que l'opération de *staging* s'est bien passée, connectez vous par ssh au conteneur herbergeant votre application :

```sh
cf ssh StaticWebSite
cd app
cd public
ls
```

> Vous n'avez pas besoin de modifier le code existant du script `compile`, rajoutez votre code juste avant l'installation de nginx

> utilisez la commande `status` afin de générer des traces dans la sortie standard lors d'un déploiement

### Packaging de pandoc

L'utilitaire `pandoc` vous est fourni. Modifiez votre buildpack afin de le copier dans le répertoire cible:

```sh
cp -f $compile_buildpack_dir/pandoc pandoc
```

- redéployez votre buildpack

- redéployez l'application

Afin de vérifier que pandoc est bien pris en compte, connectez vous par ssh au conteneur herbergeant votre application:

```sh
cf ssh StaticWebSite
cd app
ls pandoc
```

### Utilisation de pandoc

Utilisez `pandoc` pour transformer les fichiers `script/*.md` en html avant de les stocker dans le répertoire `public`

Connectez vous à cette [adresse](http://static.local.pcfdev.io) afin de tester que tout c'est bien passé.

### Pour faire joli

Afin d'avoir un rendu plus joli, rajoutez l'option suivante à l'exécution de pandoc:

```sh
-c "https://cdn.rawgit.com/dashed/6714393/raw/ae966d9d0806eb1e24462d88082a0264438adc50/github-pandoc.css"
```

# Exercice 2 : le faire à partir de rien

- créer un répertoire pour le buildpack
- créer un sous répertoire `dependencies`
- y copier les fichiers `pandoc` (à trouver dans les labs) et [ngnix](https://buildpacks.cloudfoundry.org/concourse-binaries/nginx/nginx-1.11.5-linux-x64.tgz)
- créer un sous répertoire `bin`
- créer les scripts bash `detect`, `compile` et `release` simples en vous inspirant des scripts du buildpack staticfile standard (voir le premier exercice)
- vérifier qu'ils sont bien exécutables
- tester ces scripts en local
- enregistrer le buildpack dans Cloud Foundry avec les commandes `cf` adéquates

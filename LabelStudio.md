Tutoriel pour Linux/Mac, vous pouvez suivre sur windows en installant gitbash ou WSL et un environnement Anaconda
- https://labelstud.io/guide/install.html#Install-with-Anaconda
- git-scm.com/install/windows

Nous allons d'abord récupérer un manuscrit astronomique avec l'aide du code que nous avons développé avec le tutoriel IIIF : 
```
python -m venv venv/
```
```
source venv/bin/activate
```
```
python aikoncli.py https://eida.obspm.fr/eida/iiif/wit404_pdf553/manifest.json test/
```
```
deactivate
sudo rm -rf venv/
```
Nous récupérons dans un dossier test/wit404_pdf553 un petit jeu de données au format YOLO.

# Tutoriel : Installation et importation de projets dans Label Studio

[Documentation officielle de Label Studio](https://labelstud.io/guide/quick_start)

Développé par HumanSignal, Label Studio est une application open source d'annotation de données très populaire. Modulaire et programmable, elle est idéale pour le prototypage de toute tâche liée à la création de jeux de données pour les humanités numériques, tout en restant suffisamment simple pour une utilisation individuelle par des chercheurs, ingénieurs ou étudiants ne disposant pas d'une infrastructure technique complexe.

Label Studio prend en charge les environnements d'annotation pour un large éventail de tâches d'apprentissage automatique :

- **Vision par ordinateur** (classification, détection, suivi, segmentation)

- **Traitement automatique du langage naturel** (OCR/HTR, reconnaissance d'entités nommées, analyse des sentiments, réponse contextuelle)

- **Traitement audio et vocal** (transcription, reconnaissance vocale, reconnaissance des émotions)

- **Traitement multimodal** (audio-texte ou image-texte)

- **Évaluation et optimisation** pour tous types de modèles.

## 1 : Installation de Label Studio avec Python

Il existe plusieurs façons d’installer et de configurer un environnement Label Studio ; nous utiliserons ici la version pour Python disponible via PIP.

1. Dans un terminal, créez un répertoire `labelstudio` :

```
mkdir -p labelstudio/subdirectory
cd labelstudio/
```
2. Créez un environnement virtuel Python et activez-le :

```
python -m venv labelstudioenv
source labelstudioenv/bin/activate
```
3. Installez Label Studio dans l'environnement virtuel :

```
pip install label-studio
```
4. Configurez l'accès au stockage (cette opération doit être répétée à chaque réactivation de l'environnement virtuel) :

```
export LABEL_STUDIO_LOCAL_FILES_SERVING_ENABLED=true
```
```
export LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT=/home/user/labelstudio/
```

5. Lancez Label Studio :

```
label-studio
```
L'application devrait s'ouvrir automatiquement dans votre navigateur web par défaut sur le port 8080. [http://localhost:8080/](http://localhost:8080/)

Vous devez créer un utilisateur Label Studio, même si vous utilisez un système local.

## 2 : Premiers pas avec Label Studio

1. Cliquez sur le menu `Create Project` et donnez un nom à votre projet. Ignorez le menu `Data Import` et accédez directement au menu `Labeling Setup`.

2. Dans la catégorie `Computer Vision`, sélectionnez `Object Detection with Bounding Boxes`.

3. Vous serez redirigé vers une interface graphique (GUI) pour configurer l’environnement d’annotation. Passez en mode `Code` et copiez-collez l'extrait de pseudo-code suivant :

```
<View>

<Image name="image" value="$image"/>

<RectangleLabels name="label" toName="image">

<Label value="Illustration" background="#FF0052"/><Label value="Finger" background="#00C68D"/><Label value="Table" background="#0055DA"/></RectangleLabels>

</View>

```

Enfin, n'oubliez pas de cliquer sur `Save`.

## 3 : Connexion d'un stockage externe

Nous avons maintenant un projet avec une interface d'annotation, mais aucune donnée n'a encore été importée. Bien que Label Studio permette d'importer des éléments un par un, il est plus pratique d'effectuer des importations groupées par dossier.

1. Pour ce faire, cliquez sur le bouton bleu `Connect Cloud Storage` et sélectionnez `Local Files`.

2. Saisissez un nom pour le stockage (par exemple, « Données d'entraînement ») et indiquez le chemin d'accès local absolu au dossier `subdirectory`.

```
/home/user/labelstudio/subdirectory
```
Cliquez ensuite sur  `Verify Connection`, puis sur `Next`.

3. Dans le menu `Import Settings & Preview`, sous `Import Method`, sélectionnez l'option `Files - Automatically creates a task for each storage object`. Vous pouvez ignorer l'option `File Name Filter` ; puis `Next` et `Save & Sync`.

Si vous modifiez les fichiers du dossier, vous pouvez retourner dans le menu `Cloud Storage` et utiliser `Sync Storage` pour importer les nouvelles images.

4. Ajoutez des images à votre sous-répertoire et cliquez sur `Sync Storage` pour qu'elles apparaissent comme des « tâches » dans votre environnement d'annotation.

## 4 : Environnement d’annotation Label Studio

Le pseudo-code copié-collé définit un environnement simple avec trois classes :

- Illustration
- Doigt
- Table

Dans l’interface du projet Label Studio, vous pouvez ajuster les paramètres du projet d’annotation, organiser vos tâches et accéder à l’environnement d’annotation en cliquant sur le bouton `Label All Tasks`.

L’environnement d’annotation affiche votre image. Vous pouvez y dessiner des cadres de délimitation et les assigner à des classes spécifiques à l’aide des touches numériques. Une fois les cadres dessinés (ou si aucun objet n’est présent à annoter), cliquez sur le bouton `Submit` pour enregistrer votre travail dans Label Studio. Si une image est invalide et doit être exclue de l’ensemble de données, vous pouvez utiliser le bouton `Skip`.

Vous pouvez zoomer sur les images à l’aide des raccourcis `CTRL +` et `CTRL -`. La barre latérale gauche propose plusieurs outils, notamment un historique des actions et des informations sur les classes et leurs relations (pour les ontologies hiérarchiques).
## 5 : Importer un jeu de données pré-annoté pour modification et correction

Voici la structure de notre répertoire `labelstudio/`, nécessaire pour importer un jeu de données pré-annoté au format YOLO dans Label Studio.

```
labelstudio
└── subdirectory
    ├── classes.txt
    ├── images
        ├── 1.jpg
        ├── 2.jpg
        └── n.jpg
    └── labels
        ├── 1.txt
        ├── 2.txt
        └── n.txt
```

### 5.1 Le fichier classes.txt

Une annotation au format YOLO est structurée comme suit : `class center_x center_y width height`, avec des coordonnées normalisées entre 0 et 1 et limitées à 6 décimales.

```
0 0.41880 0.28368 0.34066 0.32192
```

La notation YOLO représente les classes par un nombre compris entre 0 et n. Le fichier `classes.txt` contient la liste des classes, organisées selon l'ordre numérique utilisé dans le jeu de données à traiter.

```
Illustration
Doigt
Tableau
```
Les images et leurs étiquettes correspondantes doivent avoir exactement le même nom de fichier ; lors de l'annotation, toute image sans fichier `.txt` correspondant sera considérée comme non annotée pendant l'entraînement.

### 5.2 Conversion d'une tâche YOLO en JSON

En fermant Label Studio avec `CTRL+C` sans désactiver l'environnement virtuel, vous pouvez utiliser l'utilitaire de conversion de tâches YOLO vers JSON intégré à Label Studio avec la commande suivante :
```
label-studio-converter import yolo -i /home/user/labelstudio/subdirectory -o output.json --image-root-url "/data/local-files/?d=subdirectory/images"
```

Si la conversion réussit, deux nouveaux fichiers devraient apparaître dans l'arborescence des répertoires.
```
labelstudio
├── output.json
├── output.label_config.xml
└── subdirectory
    ├── classes.txt
    ├── images
        ├── 1.jpg
        ├── 2.jpg
        └── n.jpg
    └── labels
        ├── 1.txt
        ├── 2.txt
        └── n.txt
```
Vous pouvez ouvrir le fichier JSON dans votre navigateur pour vérifier que les annotations ont été correctement converties au format Label Studio. Pour que vos annotations soient fonctionnelles, assurez-vous qu'elles contiennent une section « annotations » incluant les paramètres x, y, largeur et hauteur, ainsi qu'un élément « rectanglelabels ».
```json
{
   "data":{
      "image":"/data/local-files/?d=one/images/4283.jpg"
   },
   "annotations":[
      {
         "result":[
            {
               "id":"306500c114",
               "type":"rectanglelabels",
               "value":{
                  "x":30.419921875,
                  "y":36.8326334733053,
                  "width":58.49609375,
                  "height":33.9532093581284,
                  "rotation":0,
                  "rectanglelabels":[
                     "Illustration"
                  ]
               }
            }
         ]
      }
   ]
}
```

### 5.3 Importation des images et annotations converties

Relancez Label Studio, accédez à votre projet, puis sélectionnez le menu `Import` et le bouton `Upload Files` .

Sélectionnez le fichier « output.json ». En théorie, tous les fichiers, ainsi que leurs annotations, devraient être importés et disponibles pour sélection et modification dans Label Studio.

## 6 : Exportation d'un jeu de données depuis Label Studio

Pour exporter un projet depuis Label Studio, utilisez simplement le bouton `Export` et choisissez l'un des formats d'exportation proposés par le logiciel (COCO, YOLO ou Pascal VOC XML). Le fichier exporté sera compressé au format .zip. Vous pouvez également exporter directement au format JSON TASK pour alimenter un pipeline de conversion si Label Studio ne prend pas en charge votre besoin.

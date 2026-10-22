# Tutoriel installation et import d'un projet sur Label Studio

[Documentation officielle de Label Studio](https://labelstud.io/guide/quick_start)

Développé par l'entreprise HumanSignal, Label Studio est une application open source d'annotation de données très populaire. Elle a l'avantage d'être modulaire et programmable pour prototyper n'importe quelle tâche de constitution de jeux de données en humanités numériques, mais aussi d'être assez simple pour être utilisé individuellement par des chercheurs, des ingénieurs ou des étudiants sans infrastructure. 

Label Studio supporte des environnements d'annotation pour toutes les tâches d'apprentissage automatique : 

- **Vision artificielle** (classification, détection, tracking, segmentation)

- **Traitement automatique du langage** (OCR/HTR, reconnaissance d'entités nommées, analyse des sentiments, réponse en fonction du contexte)

- **Traitement de l'audio et de la parole** (transcription, reconnaissance de voix, reconnaissance des émotions)

- **Traitement multimodal** (Audio-Text ou Image-Text)

- **Évaluation et fine-tuning** pour tous les types de modèles. 

## 1 : Installation de Label Studio avec Python

Il existe plusieurs manières d'installer et de mettre en place un environnement Label Studio mais nous utiliserons ici la version disponible sur PIP avec Python. 

1. Dans un terminal, créez un dossier labelstudio
```
mkdir -p labelstudio/subdirectory
cd labelstudio/
```
2. Avec Python, creez un environnement Python et activez le
```
python -m venv labelstudioenv
source labelstudioenv/bin/activate
```
3. Installez Label Studio dans l'environnement virtuel
```
pip install label-studio
```
4. Configurer l'accès au stockage (à refaire à chaque réactivation de l'environnement virtuel)
```
export LABEL_STUDIO_LOCAL_FILES_SERVING_ENABLED=true
export LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT=/home/user/labelstudio/
```

5. Lancez Label Studio
```
label-studio
```
L'application normalement automatiquement en local dans votre navigateur par défaut sur le port 8080 [http://localhost:8080/](http://localhost:8080/)

Vous devez créer un utilisateur Label Studio même dans un système local. 

## 2 : Prendre en main Label Studio

1. Nous allons cliquer sur le menu `Create Project` où nous allons choisir un nom, nous allons ignorer le menu `Data Import` pour nous concentrer sur le menu `Labeling Setup`. 

2. Dans le menu `Computer Vision`, nous allons sélectionner `Object Detection with Bounding Boxes`

3. Nous arrivons dans une interface graphique utilisateur pour paramétrer l'environnement d'annotation. Nous allons basculer sur l'interface `Code` pour copier-coller le pseudo-code ci-dessous. 

```
<View>
  <Image name="image" value="$image"/>
  <RectangleLabels name="label" toName="image">
    
    
  <Label value="Illustration" background="#FF0052"/><Label value="Finger" background="#00C68D"/><Label value="Table" background="#0055DA"/></RectangleLabels>
</View>
```

Puis n'oublions pas de cliquer sur `Save`

## 3 : Connecter un stockage externe

Nous avons maintenant un projet avec une interface d'annotation, mais nous n'avons pas encore de données importées. L'import pièce à pièce existe dans Label Studio, mais il est plus pratique de pouvoir faire de l'import de masse par dossier. 

1. Pour cela, nous allons cliquer sur le bouton bleu `Connect Cloud Storage` pour sélectionner `Local Files`. 

2. Nous devons entrer un nom pour le stockage, comme `Training Data`, et un chemin local absolu du dossier `subdirectory`. 

```
/home/user/labelstudio/subdirectory
```
Puis, utilisons le bouton `Verify Connection` et enfin `Next`. 

3. Dans le menu `Import Settings & Preview`, `import Method`, nous allons sélectionner l'option `Files - Automatically creates a task for each storage object` ; nous ignorons l'option `File Name Filter` puis `Next` et `Save & Sync`. 

Si nous modifions les fichiers à l'intérieur du dossier, nous pouvons retourner sur le menu `Cloud Storage` pour utiliser `Sync Storage` et importer toutes les nouvelles images. 

4. Nous allons ajouter des images dans notre sous-dossier et cliquer sur `Sync Storage` pour les voir apparaîtres comme "tâches" dans notre environnement d'annotation.

## 4 : L'environnement d'annotation Label Studio

Le pseudo-code que nous avions copié-collé est un environnement simple avec 3 classes : 

- Illustration
- Finger
- Table

Dans l'interface de projet de Label Studio, nous pouvons changer les paramètres du projet d'annotation, organiser nos tasks, et entrer dans l'environnement d'annotation avec le bouton `Label All Task`. 

L'environnement d'annotation vous présente votre image, sur laquelle vous pouvez tracer vos "bounding boxes" en sélectionnant leurs classes à l'aide des touches numériques. Une fois les boîtes tracées (ou non en cas d'absence d'objet à annoter), il faut cliquer sur le bouton `Submit` pour que Label Studio les enregistre. Si l'image n'est pas valide et doit être exclue du corpus, alors le bouton `Skip` est disponible. 

Il est possible de zoomer dans les images à l'aide des raccourcis `CTRL +` et `CTRL -`, dans la barre de gauche, plusieurs outils sont disponibles comme un historique, des informations sur les classes et les relations entre les classes (pour les ontologies hiérarchiques). 

## 5 : Importer un jeu de données préannoté pour le modifier et le corriger

Voici l'organisation de notre dossier labelstudio/ dont nous avons besoin pour importer un jeu de données préannoté au format YOLO dans Label Studio.
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

Une annotation au format YOLO est construite ainsi : `classe centre_x centre_y largeur hauteur` avec des coordonées normalisées de 0 à 1 limitées à 6 décimales.  
```
0 0.418801 0.283681 0.34066 0.32192
```

La notation YOLO indique les classes en fonction d'un chiffre entre 0 et n. À l'intérieur du fichier `classes.txt` se trouvent les classes organisées dans l'ordre du numéro du dataset que vous voulez corriger. 

```
Illustration
Finger
Table
```
Les images et les étiquettes doivent porter exactement le même nom ; dans le cadre de la procédure d'annotation, les images dépourvues de fichier .txt seront considérées comme non annotées.

### 5.2 Convertir le YOLO en JSON TASK

En éteignant Label Studio avec `CTRL C` sans désactiver l'environnement virtuel, nous pouvons utiliser l'utilitaire interne de conversion yolo -> json task de Label Studio avec cette commande :
```
label-studio-converter import yolo -i /home/user/labelstudio/subdirectory -o output.json --image-root-url "/data/local-files/?d=subdirectory/images"
```
Si la conversion fonctionne, alors deux nouveaux fichiers doivent apparaître dans l'arborescence. 
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
Vous pouvez ouvrir le fichier JSON dans votre navigateur pour vérifier si les annotations ont été correctement converties au format Label Studio. Pour que vos annotations soient fonctionnelles, assurez-vous qu'elles contiennent bien une section « annotations » incluant les paramètres x, y, width (largeur) et height (hauteur), ainsi qu'un élément « rectanglelabels ».
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

### 5.3 Importer les images et les annotations converties

Relancez Label Studio, accédez à votre projet, puis sélectionnez le menu `import` et le bouton `Upload Files`.

Sélectionnez le fichier `output.json`. En principe, tous les fichiers ainsi que les annotations devraient être importés, sélectionnables et modifiables dans Label Studio. 

## 6 : Exporter un jeu de données hors de Label Studio

Pour exporter un projet hors de Label Studio, il suffit d'utiliser le bouton `Export` et de choisir le format d'export proposé par le logiciel (COCO, YOLO, Pascal VOC XML) avec un fichier .zip. 


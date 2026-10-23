# Tutorial: Installation and Project Import in Label Studio

[Official Label Studio Documentation](https://labelstud.io/guide/quick_start)

Developed by the company HumanSignal, Label Studio is a highly popular open-source data annotation application. It is both modular and programmable ; making it ideal for prototyping any task related to building datasets for the digital humanities ; while remaining simple enough for individual use by researchers, engineers, or students without a complex technical infrastructure.

Label Studio supports annotation environments for a wide range of machine learning tasks:

- **Computer Vision** (classification, detection, tracking, segmentation)

- **Natural Language Processing** (OCR/HTR, named entity recognition, sentiment analysis, context-based response)

- **Audio and Speech Processing** (transcription, voice recognition, emotion recognition)

- **Multimodal Processing** (Audio-Text or Image-Text)

- **Evaluation and fine-tuning** for all types of models.

## 1: Installing Label Studio with Python

There are several ways to install and set up a Label Studio environment; here, we will use the version for Python available via PIP.

1. In a terminal, create a `labelstudio` directory:
```
mkdir -p labelstudio/subdirectory
cd labelstudio/
```
2. Create a Python virtual environment and activate it:
```
python -m venv labelstudioenv
source labelstudioenv/bin/activate
```
3. Install Label Studio within the virtual environment:
```
pip install label-studio
```
4. Configure storage access (this must be repeated each time the virtual environment is reactivated):
```
export LABEL_STUDIO_LOCAL_FILES_SERVING_ENABLED=true
export LABEL_STUDIO_LOCAL_FILES_DOCUMENT_ROOT=/home/user/labelstudio/
```

5. Launch Label Studio:
```
label-studio
```
The application should automatically open in your default web browser at port 8080: [http://localhost:8080/](http://localhost:8080/)

You must create a Label Studio user, even when using a local system.

## 2: Getting Started with Label Studio

1. Click on the `Create Project` menu and enter a name for your project. Skip the `Data Import` menu and proceed directly to the `Labeling Setup` menu.

2. Under the `Computer Vision` category, select `Object Detection with Bounding Boxes`.

3. You will be taken to a graphical user interface (GUI) for configuring the annotation environment. Switch to the `Code` view and copy-paste the following pseudo-code snippet:

```
<View>
<Image name="image" value="$image"/>
<RectangleLabels name="label" toName="image">


<Label value="Illustration" background="#FF0052"/><Label value="Finger" background="#00C68D"/><Label value="Table" background="#0055DA"/></RectangleLabels>
</View>
```

Finally, be sure to click `Save`.

## 3: Connecting External Storage

We now have a project with an annotation interface, but no data has been imported yet. While Label Studio supports importing items one by one, it is more convenient to perform bulk imports by folder.

1. To do this, click the blue `Connect Cloud Storage` button and select `Local Files`.

2. Enter a name for the storage—for example, `Training Data`—and provide the absolute local path to the `subdirectory` folder.

```
/home/user/labelstudio/subdirectory
```
Then, click `Verify Connection`, followed by `Next`.

3. In the `Import Settings & Preview` menu, under `Import Method`, select the option `Files - Automatically creates a task for each storage object`. You can skip the `File Name Filter` option; then click `Next` and `Save & Sync`.

If you modify the files within the folder, you can return to the `Cloud Storage` menu and use `Sync Storage` to import any new images.

4. Add images to your subdirectory and click `Sync Storage` to have them appear as "tasks" within your annotation environment.

## 4: The Label Studio Annotation Environment

The pseudo-code we copied and pasted defines a simple environment with three classes:

- Illustration
- Finger
- Table

Within the Label Studio project interface, you can adjust annotation project settings, organize your tasks, and enter the annotation environment by clicking the `Label All Tasks` button.

The annotation environment displays your image, where you can draw bounding boxes and assign them to specific classes using the number keys. Once the boxes have been drawn (or if no objects are present to be annotated), click the `Submit` button to save your work in Label Studio. If an image is invalid and should be excluded from the dataset, you can use the `Skip` button.

You can zoom in on images using the `CTRL +` and `CTRL -` shortcuts. The left-hand sidebar provides several tools, including an action history and information regarding classes and the relationships between them (for hierarchical ontologies).

## 5: Importing a pre-annotated dataset for modification and correction

Below is the structure of our `labelstudio/` directory, which is required to import a pre-annotated dataset in YOLO format into Label Studio.
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

### 5.1 The classes.txt file

An annotation in YOLO format is structured as follows: `class center_x center_y width height`, with coordinates normalized from 0 to 1 and limited to 6 decimal places.
```
0 0.41880 0.28368 0.34066 0.32192
```

The YOLO notation represents classes using a number between 0 and *n*. The `classes.txt` file contains the list of classes, organized according to the numerical order used in the dataset you wish to process.

```
Illustration
Finger
Table
```
Images and their corresponding labels must have the exact same filename; during the annotation process, any images that lack a corresponding `.txt` file will be considered unannotated during training. 

### 5.2 Converting YOLO to JSON TASK

By shutting down Label Studio using `CTRL+C` and without deactivating the virtual environment, we can use Label Studio’s built-in YOLO-to-JSON-task conversion utility with the following command:
```
label-studio-converter import yolo -i /home/user/labelstudio/subdirectory -o output.json --image-root-url "/data/local-files/?d=subdirectory/images"
```
If the conversion is successful, two new files should appear in the directory structure.
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
You can open the JSON file in your browser to verify that the annotations have been correctly converted to the Label Studio format. For your annotations to be functional, ensure that they contain an "annotations" section that includes the parameters x, y, width, and height, as well as a "rectanglelabels" element.
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

### 5.3 Importing the converted images and annotations

Relaunch Label Studio, navigate to your project, then select the `Import` menu and the `Upload Files` button.

Select the `output.json` file. In theory, all the files, along with their annotations, should be imported and become available for selection and editing within Label Studio.

## 6: Exporting a dataset from Label Studio

To export a project from Label Studio, simply use the `Export` button and choose one of the export formats offered by the software (COCO, YOLO, or Pascal VOC XML), which will be provided as a .zip file. You can also directly output JSON TASK to feed a conversion pipeline if Label Studio does not support your need. 


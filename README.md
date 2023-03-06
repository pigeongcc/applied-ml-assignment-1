Colab model.ipynb: [link](https://colab.research.google.com/drive/1ZzxEX77AhZlFWkP3MBYkI4Tkn0fkF94a?usp=sharing).

# 1. Dataset

I collected a dataset of 38 photos with books (186 bboxes) and cars (88 bboxes). 

Examples of annotated photos:

<img src="https://user-images.githubusercontent.com/48735488/223205547-3b35b7c3-30d1-44a3-b7b7-e83e9758a9fd.png" width=35% height=35%>
<img src="https://user-images.githubusercontent.com/48735488/223206541-5c1a4bdc-edb8-41ff-8e40-41a783437eb8.png" width=35% height=35%>
<img src="https://user-images.githubusercontent.com/48735488/223205674-bd6edd20-3bc7-47f0-954d-86c9a47c6f0c.png" width=35% height=35%>
<img src="https://user-images.githubusercontent.com/48735488/223206623-9aaec676-1116-45ca-9cdf-615c1ccffefa.png" width=35% height=35%>

# 2. Dataset Annotation and Augmentations

I annotated the dataset on Roboflow.

At the dataset generation step, I applied the following preprocessing:
* Auto-orientation of pixel data (with EXIF-orientation stripping)
* Resize to 640x640 (Stretch)

And the following augmentations:

* Randomly crop between 0 and 25 percent of the image
* Random rotation of between -10 and +10 degrees
* Random brigthness adjustment of between -20 and +20 percent
* Random exposure adjustment of between -15 and +15 percent
* Random Gaussian blur of between 0 and 1 pixels

Examples of annotated augmented samples:

<img src="https://user-images.githubusercontent.com/48735488/223210290-9dd89df1-07af-45b3-8e44-268ffb85878f.png" width=35% height=35%>
<img src="https://user-images.githubusercontent.com/48735488/223210377-28e1d42b-0235-4d4c-b0c1-ddcdf953b636.png" width=35% height=35%>


# 3. Faster R-CNN

I trained an R-CNN model from Detectron2. In particular, its "faster_rcnn_R_101_FPN_3x" version.

I ran 3000 iterations and obtained interesting results. See them below at point 5.

Note: the inference detections for Faster R-CNN in Colab looks poorly because it was ran with low IoU threshold of 0,05. But the metrics were computed for an adequate threshold of 0,2.

# 4. YOLOv5

I also trained a YOLOv5 model. I chose v5 architecture because it's lightweight and converges quicker than some other YOLO versions. 

I ran 1500 epochs - it's an adequate number for object detection tasks. However, the early stopping was triggered after 427 epochs.

# 5. Results

Both models were evaluated based on mAP, speed of inference, and size.

## Faster R-CNN

- **mAP among all classes:**

```
    mAP50 = 0.469
    mAP50-95 = 0.365
```

- **Speed of inference:** 116.3 ms

- **Model Size:** 330.1 MB

Evaluation results (averaged between classes):
|   AP   |  AP50  |  AP75  |  APs   |  APm   |  APl   |
|:------:|:------:|:------:|:------:|:------:|:------:|
| 36.536 | 46.857 | 43.728 | 31.191 | 36.892 | 36.502 |

Evaluation results (per-class):
| class      | AP   | class      | AP     | class      | AP    |
|:-----------|:-----|:-----------|:-------|:-----------|:------|
| books-cars | nan  | book       | 73.073 | car        | 0.000 |

As you can see, there is no progress with cars detection.

Possible reasons are:

1. Class imbalance:

|  category  | #instances   |  category  | #instances   |  category  | #instances   |
|:----------:|:-------------|:----------:|:-------------|:----------:|:-------------|
| books-cars | 0            |    book    | 442          |    car     | 183          |
|   total    | 625          |            |              |            |              |

2. Poor Dataset Quality:

In the original dataset, there are 88 bboxes with cars (which were augmented to 183 samples, as you see in the table above). Among these 88 examples, there are plenty of ways a car was captured at the photo: side, back, front, diagonally, from afar, from near. Such variability together with lack of data could lead to poor performance within the limited number of epochs.

The books case is much easier: all of them were captured from approximately same distance and angle. Moreover, there are more books samples in the dataset. So, the model has managed to converge well during the training process. The resulted `mAP50:95` is 0.73.


## YOLOv5

- **mAP among all classes:**

```
    mAP50 = 0.568
    mAP50-95 = 0.211
```

- **Speed of inference:** 8.2 ms

- **Model Size:** 14.4 MB (7015519 parameters)

Details:

|Class     |Images    |Instances   | Precision  | Recall    |   mAP50    | mAP50-95 |
|:--------:|:---------|:----------:|:-----------|:---------:|:-----------|:--------:|
|all       |   5      |     28     |  0.599     |  0.61     |   0.568    |  0.211   |
|book      |   5      |     15     |  0.719     | 0.682     |   0.675    |   0.24   |
|car       |   5      |     13     |   0.48     | 0.538     |   0.461    |  0.182   |


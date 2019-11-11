# Using ultralyitcs/yolov3 to create transfer-learned Yolo model

- clone git repository https://github.com/ultralytics/yolov3
- yolov3.weights is located in \Applications\Tracking and inventory\Resources\Models
- Install dependencies (torch, numpy, etc.) 

## Data preparation

```
data
+-- my_data
|	+-- images
|		+-- 00000000.jpg
|		+-- 00000001.jpg
| +-- labels
|		+-- 00000000.txt
|		+-- 00000001.txt
|   +-- my_data.names
|   +-- my_data.data
|	+-- train.txt
|	+-- validation.txt
|	+-- yolov3_custom.cfg
```

*my_data.data*: 

> classes=n
> train=data/my_data/train.txt
> valid=data/my_data/validation.txt
> names=data/my_data/my_data.names
> backup=backup/

*my_data.names*:

> name_1
> name_2
> name_3
> ...
> name_n

**Note:** When referring  to *name_1* via index, use index=0

*train.txt*:

> data/images/00000013.jpg
> data/images/00002021.jpg
> ...

*validation.txt*:

> data/images/00000043.jpg
> data/images/00000121.jpg
> ...

*labels/xxxxxxxx.txt*: 

> class_idx_of_instance x_center y_center width height

**Note:** Coordinates are normalized => x_center = x_center_pixels/width, ..., width = width_pixels/width, ...

*yolov3_custom.cfg*: See below





## Modifying and transfer-training a yolo model on custom data

Obtain .cfg file and .weights file from original model to which you want to apply transfer learning.  (*yolov3.cfg* in yolov3/cfg along with *yolov3.weights* in *weights* represents the common yolo model which was trained on COCO)

### Create custom Yolo configuration

- Copy an appropriate yolo.cfg file. Rename it (e.g. *yolov3_custom.cfg*)
- Edit training parameters (top of file) **Note:** Think some of these are ignored, e.g. batch_size
- Edit fields related to number of classes in .cfg file:
  - https://github.com/ultralytics/yolov3/wiki/Example:-Transfer-Learning
- **Note:** Not sure how *yolov3.weights* are applied to the modified *yolov3_custom*. Probably a good idea not to touch anything else if you want to do transfer learning successfully.

### Train the model

Ensure that **ONNX_EXPORT = False** in models.py

```python train.py --data data/my_data/my_data.data --cfg cfg/yolov3_custom.cfg --weights weights/yolov3.weights --transfer --epochs=100 --batch-size=16```

This will create and store a PyTorch weights file *best.pt* in the *weights* directory.

You will also see results.png in yolov3 directory with training curves, etc.

- *--transfer* flag freezes all weights but those of the last layers
- To resume training with all weights unlocked, run training again without *--transfer* and *--weights* flags, but with *--resume* flag. (Initializes weights with *weights/last.pt*)
- To use the GPU, add the flag *--device=0*

### Verify model

Ensure that **ONNX_EXPORT = False** in models.py

```python detect.py --cfg cfg/yolov3_custom.cfg --data data/my_data/my_data.data --weights weights/best.pt```

Check directory *outputs* for results on images stored in *data/samples*.









## Export the model to ONNX

Ensure that **ONNX_EXPORT = True** in models.py

```python detect.py --cfg cfg/yolov3_custom.cfg --data data/my_data/my_data.data --weights weights/best.pt```

This will create and store an ONNX model *export.onnx* in the *weights* directory.
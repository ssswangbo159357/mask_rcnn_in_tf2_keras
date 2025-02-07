## Mask RCNN in tf2-keras

### bilibili讲解视频

[![Watch the video](https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/b站视频页.png)](https://www.bilibili.com/video/BV1qA411w7Zg?share_source=copy_web)

### requirements

- tensorflow-gpu >= 2.1.0
- xmltodict
- Pillow
- opencv-python
- matplotlib

### 检测效果

- VOC2012

<img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample1.png" width="350" height="230"/>  <img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample2.png" width="350" height="230"/>

<img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample3.png" width="350" height="230"/>  <img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample4.png" width="350" height="230"/>

<img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample5.png" width="350" height="230"/>  <img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample6.png" width="350" height="230"/>

<img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample7.png" width="350" height="230"/>  <img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/sample8.png" width="350" height="230"/>


### 训练`Voc2012`数据

1. 构建模型
```python
from mrcnn.mask_rcnn import MaskRCNN
mrcnn = MaskRCNN(classes=['_background_', 'aeroplane', 'bicycle', 'bird', 'boat', 'bottle', 'bus',
                              'car', 'cat', 'chair', 'cow', 'diningtable', 'dog', 'horse', 'motorbike',
                              'person', 'pottedplant', 'sheep', 'sofa', 'train', 'tvmonitor'],
                 is_training=True,
                 batch_size=2)
```

2. 训练
```python
mrcnn.train(epochs=300, log_dir='./logs', data_path='../data/voc2012_46_samples')```
```

3. tensorboard查看效果
```python
tensorboard --host 0.0.0.0 --logdir ./logs/ --port 9013 --samples_per_plugin=images=40
```

4. 浏览器打开: `http://127.0.0.1:9013`

<img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/scalar.png" width="800" height="437"/>  

<img src="https://raw.githubusercontent.com/yyccR/Pictures/master/mask_rcnn/images.png" width="800" height="437"/>


### 测试`Voc2012`
1. 构建模型
```python
from mrcnn.mask_rcnn import MaskRCNN
mrcnn = MaskRCNN(classes=['_background_', 'aeroplane', 'bicycle', 'bird', 'boat', 'bottle', 'bus',
                              'car', 'cat', 'chair', 'cow', 'diningtable', 'dog', 'horse', 'motorbike',
                              'person', 'pottedplant', 'sheep', 'sofa', 'train', 'tvmonitor'],
                 is_training=False,
                 batch_size=1,
                 image_shape=[640,640,3])
```

2. 加载权重
```python
model_path = '.h5 file path'
mrcnn.load_weights(model_path, by_name=True)
```

3. 测试, 在`/tmp`目录下可以看到检测结果保存的图片
```python
import cv2
import numpy as np
from mrcnn.anchors_ops import get_anchors
anchors = get_anchors(image_shape=mrcnn.image_shape,
                      scales=selmrcnnf.scales,
                      ratios=mrcnn.ratios,
                      feature_strides=mrcnn.feature_strides,
                      anchor_stride=mrcnn.anchor_stride)
all_anchors = np.stack([anchors], axis=0)

image = cv2.imread("image_path")
image = np.stack([image], axis=0)

boxes,class_ids,scores,masks = mrcnn.predict(image=image, anchors=anchors, draw_detect_res_figure=True)
```

### 训练自己的数据
1. 自定义自己的generator, 参考 `./data/generate_voc_segment_data.py`, 其`self.next_batch`方法输出每个batch数据, 格式为：
- imgs - self.image_mean, shape = [batch, h, w, 3]
- masks, shape = [batch, h, w, max_instances]
- gt_boxes: shape = [batch, max_instances, 4]
- labels: shape = [batch, max_instances]

2. 修改`generate_tfrecord_files.py`, 替换 `voc_seg = VocSegmentDataGenerator(*)` 为上面你定义的generator类.
3. 模型启动和训练流程和上文`Voc2012`训练一样
4. 模型测试只需要修改对应类别即可.

### 代码细节

- 源码训练voc2012数据: /official_mask_rcnn/samples/voc/voc2012.py
- 复现代码: /mrcnn
- 数据生成: /data/generate_voc_segment_data.py
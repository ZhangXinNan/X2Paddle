### Warning
> **TensorFlow2Paddle is not stable and lots of tensorflow operations are not supported yet**

> **Only tested on vgg_16/resnet_v1_50/inception_v3 with is_training=False**

### Dependency
> 1. python = 2.7
> 2. PaddlePaddle >= 1.2.0
> 3. TensorFlow >= 1.12.0

**Notice：You can install PaddlePaddle and Tensorflow in different virtual environment since there's dependency conflict between PaddlePaddle and TensorFlow**

### Usage
> **1. Model file: Tensorflow checkpoint directory**

> **2. input tensors' and output tensors' name**

### Demo: How to transform tensorflow resnet_v1_50 pretrained model to PaddlePaddle model for inference
#### 1. Get pretrained_model

```
git clone https://github.com/PaddlePaddle/X2Paddle.git
cd X2Paddle/TensorFlow2Paddle
wget http://download.tensorflow.org/models/resnet_v1_50_2016_08_28.tar.gz
tar xzvf resnet_v2_50_2017_04_14.tar.gz
```

#### 2. Change model to ckpt model with meta file


```
python demo/save_resnet_ckpt_model.py resnet_v1_50.ckpt ./new_ckpt_model
```

#### 3. Export PaddlePaddle model

```
python demo/export_resnet_to_paddle_model.py new_ckpt_model/resnet.meta new_ckpt_model fluid_model
```

#### 4. Test PaddlePaddle model
```python 
from fluid_model.mymodel import KitModel
import paddle.fluid as fluid
import numpy
import os

result = KitModel()
exe = fluid.Executor(fluid.CPUPlace())
exe.run(fluid.default_startup_program())

var_list = list()
for f in os.listdir('./fluid_model'):
    f = f.split('/fluid')[-1]
    if f.startswith("param_"):
        var_list.append(fluid.default_main_program().global_block().var(f))
fluid.io.load_vars(exe, './fluid_model', vars=var_list)

test_data = numpy.random.rand(1, 3, 224, 224)
test_data = numpy.array(test_data, dtype='float32')
result = exe.run(fluid.default_main_program(), 
      feed={'input_0':numpy.array(img_data, dtype='float32')}, 
      fetch_list=[result])
print(result)
```

### Link
[MMdnn-Tensorflow](https://github.com/Microsoft/MMdnn/tree/master/mmdnn/conversion/tensorflow)
Good reference: https://github.com/PRBonn/bonnet/issues/15


### REQUIREMENTS:
* Docker
* Nvidia-docker


1. `sudo dockerd`


[Only need to do this once] In bonnet directory: 

1. `docker pull tano297/bonnet:cuda9-cudnn7-tf17-trt304`

2. `nvidia-docker build -t bonnet .`

Put your model + image into another directory, e.g. /Documents

1. `nvidia-docker run -ti --rm -e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v $HOME/.Xauthority:/home/developer/.Xauthority -v /home/$USER/Documents:/test --net=host --pid=host --ipc=host bonnet /bin/bash`

Model and image are now put in `/test`

At this point, docker container should be running.
1. Go to `~/bonnet_wrkdir/deploy_cpp/src/ros/config` (or something like that).

2. Edit the cnn_config.yaml file: model path points to “/test/frozen” [a different path, here you’re going to freeze your trained model into this directory]

3. [Only need to do this once] Setup dependencies for training as specified here:
`https://github.com/PRBonn/bonnet/tree/master/train_py`

When you’re ready to freeze your graph: run the following under `~/bonnet_wrkdir/train_py`

1. `python ./cnn_freeze.py -p /test/<name of pretrained model> -l /test/frozen`

> For Example:
> 
> `python ./cnn_freeze.py -p ~/datasets/bonnet_data/city_512 -l /tmp/frozen_model/city_512`
> In docker: `./cnn_freeze.py -p /shared/city_512 -l /tmp/frozen_model/city_512`

2. Run cnn_use.py or cnn_use_pb_tensorRT.py (for speed up) to to analyze the image.
   1. `python ./cnn_use.py -l /test/log -p /test/<name of pretrained model> -i /test/<name of image>`
   
 > For example: 
 >  
 > `python ./cnn_use.py -l /tmp/log -p ~/datasets/bonnet_data/city_512/ -i ~/datasets/bonnet_data/leftImg8bit/train/zurich/zurich_000040_000019_leftImg8bit.png`

   2. `python ./cnn_use_pb_tensorRT.py -l /test/log -p /test/frozen -i /test/<name of image>`


!!!!!
Results are in the `/test/log`

Done

FAQ:
- If you encounter this issue:
```
Traceback (most recent call last):
  File "./cnn_use.py", line 37, in <module>
    import tensorflow as tf
  File "/usr/local/lib/python3.6/dist-packages/tensorflow/__init__.py", line 24, in <module>
    from tensorflow.python import *  # pylint: disable=redefined-builtin
  File "/usr/local/lib/python3.6/dist-packages/tensorflow/python/__init__.py", line 63, in <module>
    from tensorflow.python.framework.framework_lib import *  # pylint: disable=redefined-builtin
  File "/usr/local/lib/python3.6/dist-packages/tensorflow/python/framework/framework_lib.py", line 76, in <module>
    from tensorflow.python.framework.ops import Graph
  File "/usr/local/lib/python3.6/dist-packages/tensorflow/python/framework/ops.py", line 53, in <module>
    from tensorflow.python.platform import app
  File "/usr/local/lib/python3.6/dist-packages/tensorflow/python/platform/app.py", line 24, in <module>
    from tensorflow.python.platform import flags
  File "/usr/local/lib/python3.6/dist-packages/tensorflow/python/platform/flags.py", line 25, in <module>
    from absl.flags import *  # pylint: disable=wildcard-import
ModuleNotFoundError: No module named 'absl'
```
Make sure you specify `python` in front of the command you are running!

- If you encounter missing 'arch.abstract_net':
```
Copying files to /tmp/frozen_model/city_512 for further reference.
Traceback (most recent call last):
  File "./cnn_freeze.py", line 165, in <module>
    "architecture", os.getcwd() + '/arch/' + NET["name"] + '.py')
  File "/home/tonirv/Code/bonnet/train_py/arch/bonnet.py", line 28, in <module>
    from arch.abstract_net import AbstractNetwork
ImportError: No module named arch.abstract_net
```

Do
`pip install arch`


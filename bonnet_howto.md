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

1. `./cnn_freeze.py -p /test/<name of pretrained model> -l /test/frozen`

> For Example:
> 
> `./cnn_freeze.py -p ~/datasets/bonnet_data/city_512 -l /tmp/frozen_model/city_512`

2. Run cnn_use.py or cnn_use_pb_tensorRT.py (for speed up) to to analyze the image.
   1. `./cnn_use.py -l /test/log -p /test/<name of pretrained model> -i /test/<name of image>`
   
 > For example: 
 >  
 > `./cnn_use.py -l /tmp/log -p ~/datasets/bonnet_data/city_512/ -i ~/datasets/bonnet_data/leftImg8bit/train/zurich/zurich_000040_000019_leftImg8bit.png`

   2. `./cnn_use_pb_tensorRT.py -l /test/log -p /test/frozen -i /test/<name of image>`


!!!!!
Results are in the `/test/log`

Done

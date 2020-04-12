# Retinanet-Tutorial
This is a tutorial created for the sole purpose of helping you quickly and easily train an object detector for your own dataset. It is an improvement over my [previous tutorial](https://github.com/jaspereb/FasterRCNNTutorial) which used the now outdated FasterRCNN network and tensorflow. This tutorial makes use of keras and tensorboard.

It is based on the excellent [keras-retinanet](https://github.com/fizyr/keras-retinanet) implementation by fizyr which you should definitely read if you have time. This includes a sample dataset of images of plums but is intended to help you train your on your own dataset. This is a step which is often not well documented and can easily trip up new developers with specific data formatting requirements that aren't at all obvious. The idea of this tutorial is that you can use the instructions here, and when you get stuck refer to the screen recording of the entire process at YOUTUBE LINK HERE.

The fizyr implementation we will be using has several data formatting options (and you could even write your own). The CSV input format is probably the easiest to understand and generate, BUT it's non standard and if you ever want to use a different network you would need to convert it. Because of that, we will be using the Pascal VOC 2007 format for our data. Then if you need it in the future, there are tools to easily convert to other formats (like COCO).

#ROS?
ROS is the Robot Operating System. Included in this repository is a ROS node to run the detector as part of a robot perception system. Even if you don't have a robot, ROS drivers exist for most types of cameras so this is an easy way to get live data streams and inference results set up. If you only care about inferencing a single image just use the testDetector.py script. If you would like to use the ROS node, see USING_ROS.md in the ROS-Node folder. 

#Should I use this tutorial?
This tutorial is aimed at people who don't have a lot of experience with linux or machine learning. If you only have a passing acquaintance with bash, or care more about getting something to work than understanding the mechanisms behind it, then this is the tutorial for you. You should be able to follow the steps in the video exactly, and get a result that works. 

This is only for linux users, everything will be in Ubuntu using an Nvidia GPU. 

#Installation
The first issues that most deep learning practicioners run into are installation errors, so here's some brief advice on installing stuff. 

Most errors come from version mismatches, so make sure you get this right the first time, it's much faster to just downgrade a component than to assume a higher version will work. The key components you need to install are Nvidia drivers, CUDA, CUDNN, tensorflow, keras and a bunch of python libraries. While you can run this without a GPU, it's so slow that it's not worth it.

* NVIDIA Drivers. These are a massive pain in Ubuntu and can easily leave you with a broken OS if you do it wrong. The most reliable method I have found is to download the most recent driver version from https://www.nvidia.com/Download/index.aspx. This will give you a file like `NVIDIA-Linux-x86_64-440.82.run`. You want to close x server in order to install it (this command will drop you into terminal only mode, no graphics, so have these instructions on a separate computer!), to do this press `CTRL+ALT+F1` or `CTRL+ALT+F2`


#Creating the dataset
The first step to creating your dataset is to pick the format you want to use, we will go with Pascal VOC 2007 here, but you could also use the CSV format or COCO. 



#FAQ
##How big should my images be?
The TLDR is, images of most reasonable sizes and aspect ratios will work. If your images are super small (less than 300px a side) or super big (more than 2000px a side) you may have issues with the default anchor sizes. You could either resize your training and inference images to something like 800*800 or adjust the default anchors to suit your objects. But train the network and see if it works on your native images first. For very large images you may also run out of GPU memory.

Retinanet will automatically resize images based on the settings in the generator.py file for your dataset. The defaults are to make the minimum image side 800px and the max side 1333px. This function looks like:

    def compute_resize_scale(image_shape, min_side=800, max_side=1333):
        """ Compute an image scale such that the image size is constrained to min_side and max_side.

        Args
            min_side: The image's min side will be equal to min_side after resizing.
            max_side: If after resizing the image's max side is above max_side, resize until the max side is equal to max_side.

        Returns
            A resizing scale.
        """
        (rows, cols, _) = image_shape

        smallest_side = min(rows, cols)

        # rescale the image so the smallest side is min_side
        scale = min_side / smallest_side

        # check if the largest side is now greater than max_side, which can happen
        # when images have a large aspect ratio
        largest_side = max(rows, cols)
        if largest_side * scale > max_side:
            scale = max_side / largest_side

        return scale

If you have very large images (\> 3MP which you would get off a DSLR) you may also run out of GPU memory, the best approach is to resize these to X by 800. If you have tiny objects in large images you many need to crop them into tiles using imagemagick and train on each tile. Retinanet's default anchor sizes are in the keras-retinanet/utils/anchors.py file and range from 32px to 512px with additional scaling. So objects smaller than 30px will not be well detected. If your objects are less than this after resizing the images to be X by 800 you will need to use more tiles.

So for a 6000x4000 image with objects that are originally 100x100 pix, it would get resized to 1200*800 and the objects would be 20x20px, which are too small.

To crop the above image into 4 tiles run the following command (if not given a crop location than imagemagick will tile them). Then delete any offcuts which are created.

    for file in $PWD/.*jpg
    do
    convert $file -crop 3000x2000 $file
    done

Rather than relying on the resizing step in the code, I prefer to just resize the images to begin with. It will also allow you to keep an eye on the minimum object size during labelling. So change to the JPEGImages dir and run.

    for file in $PWD/*.jpg
    do
    convert $file -resize (Your X value)x800 $file
    done

The anchor ratios will multiply the x dimension and divide the y dimension, so if you have an aspect ratio of 0.5 your 256x256 anchor becomes 128x512.

##How long should I train for?
That depends a lot on how hard your problem is and how big the dataset is. Using a 1080Ti I could get a reasonable detector within an hour of training but would continue to improve for about 12hrs. Generally you want to look at your test set loss and that should decrease rapidly then flatten out. You want to stop training just as the curve flattens out (before you start over fitting too much). This is where tensorboard comes in useful.

##How many images do I need?
Again, this is mostly dependent on how hard your problem is. I've trained weed detectors using 20 images (~15 weeds per image) and gotten something usable, more data will obviously improve the accuracy. I would recommend starting small, maybe 1 hour of labelling effort to begin with. And you should be able to tell if the detector will work based on the results of that, before you scale up the dataset to improve accuracy.

The COCO dataset has 1.5 million object instances! Pascal VOC 2007 has 24,640 object instances over 20 classes, so roughly 1000 objects per class. It's actually not that hard to label that many instances, if each one takes 5 seconds that's about 2 hours of labelling for a 1 class detector. 










NOTES:
-change pascal_voc.py classes to be only plum


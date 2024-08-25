---
title: "[LiP] Simple real-time Human Pose Estimation in Python with OpenCV, MoveNet, and UDP Sockets"
# date: 2020-09-15T11:30:03+00:00
# description: "Brief guide to integrate Rust code into Unity"
weight: 2
# aliases: ["/p22"]
tags: ["Unity", "Python", "OpenCV", "MoveNet", "UDP"]
# author: "Me"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://mayonesso.com/blog/2024/pose-estimation"
disableHLJS: true # to disable highlightjs
disableShare: true
hideSummary: false
summary: "Human Pose Estimation and transmition through UDP"
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: false
UseHugoToc: true
cover:
    image: "cover.webp" # image path/url
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---

## • Introduction

In another episode of [learn in public](https://www.swyx.io/learn-in-public) I am here to share a little experiment that uses MoveNet to estimate a human pose of a video or the input from a webcam. Then we’ll send that information through our old and loved _User Datagram Protocol_ (yes, these are the words behind [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol)).

## • Basic Structure

We are going to use Python and the program can be logically split into three parts:

i.- **Inference Hub**: [MoveNet](https://blog.tensorflow.org/2021/05/next-generation-pose-detection-with-movenet-and-tensorflowjs.html) is a model offered by TensorFlow.
Following the instructions/idea from this [tutorial](https://www.tensorflow.org/hub/tutorials/movenet), in the _Inference Hub,_ we initialize the desired flavor of the model and set up the details to invoke the inference method.

ii.- **Frame Analysis**: Since we are analyzing a video we will extract frame by frame using [OpenCV](https://opencv.org/) to calculate the inference in a loop.

iii.- **Pose Keypoints Transmission**: The last part is all about transmitting the serialized results of the inference over the network using UDP.

### Inference Hub

This part of the is pretty straightforward, following the [tutorial](https://www.tensorflow.org/hub/tutorials/movenet) from the TensorFlow site, we extract the functionality to [download the desired model](https://www.tensorflow.org/hub/tutorials/movenet#load_model_from_tf_hub) (we can choose from _lightning_ or _thunder_, and its FP16 and INT8 quantized versions) with its inference invocation, the logic to [crop a relevant section](https://www.tensorflow.org/hub/tutorials/movenet#cropping_algorithm) of the frame (this will help us to make our inference more efficient), and the logic to convert the inference results to key points and edges.

The code is inside `pose_transmitter/inference_hub` with `hub.py` as the main entry point.

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/1_main_entry_point.webp"
>}}

### Frame Analysis

The frame extraction is made in the _VideoPoseProcessor_ object where after the initialization we are ready to run a loop processing our video.

The code is in `pose_transmitter/video_pose_processor.py`

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/2_video_processor.webp"
>}}

The parameters are the _inference_hub_, the _source,_ and a _debug_ flag.

We use [OpenCV](https://docs.opencv.org/4.x/d8/dfe/classcv_1_1VideoCapture.html) to work with the video file or with the camera input.  
The processing loop is as follows:

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/3_open_cv.webp"
>}}

Note the line 23 of the snapshot. Since OpenCV represents the colors of the image in the [Blue-Green-Red (BGR)](https://learnopencv.com/why-does-opencv-use-bgr-color-format/) format we need to convert it to the Red-Green-Blue (RGB) format, which is the format that TensorFlow uses.

After running the inference we invoke the callback (in case of being defined) and _display_results_ if specified.

The entry point of this logic is the method called `start_processing` with the only parameter the callback to process the results.

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/4_start_proc.webp"
>}}

Note how if the `debug` flag is present, we cannot use threads.  
The logic behind this is that all the operations that display UI elements have to run in the main thread. And since the `debug` flag implies showing the results on the screen (`display_results=True`) we are limited in this regard.

### Pose Keypoints Transmission

The data to be transmitted is serialized and then placed in a Queue. A worker thread is responsible for retrieving and dispatching these messages.

Our selection in this case is a simple UDP transmission, serializing the data using [MsgPack](https://msgpack.org/index.html) which is a simpler approach compared to other serialization methods (like [Procolol Buffers](https://protobuf.dev/) or [Thrift](https://thrift.apache.org/)), with support in different languages, and also more efficient than simple JSON.

The code is in `pose_transmitter/qudp_transmitter.py`
We can see the constructor for QUDPTransmitter here:

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/5_transmitter.webp"
>}}

The code that receives the data (and what is used as the callback in the `VideoPoseProcessor` is this `put_message` method:

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/6_put_msge.webp"
>}}

We activate the worker thread with `_activate_transmission` and while is active, we will be sending the data through `_send`:

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/7_send.webp"
>}}

The data consists of a list of pairs of floating-point values (`List<float[]>`):

```string
[  
    [989.79443359375, 635.7391967773438],
    [1091.6107177734375, 512.1315307617188],
    [889.1296997070312, 520.48828125],
    [1204.3812255859375, 533.4179077148438],
    ...  
]
```

> 📝 Note:  
> It is worth noting that not all points may be inferred and included in the list.  
> For example, if the camera captures half of a person’s body, it will send the information of the points found on that particular half.

## • Finally, _all together..._

The last part (ok, this time _is_ the actual ‘last part’) is putting it all together.

The `main.py` file in our project is in charge of that.

We use [argparse](https://docs.python.org/3/library/argparse.html) to being able to specify parameters from the terminal:

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/8_arg_parse.webp"
>}}

And from there, we wrap up the rest:

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/9_run_proc.webp"
>}}

So now we are ready to use the program.  
On the Terminal, we can execute:

```bash
python ./pose_transmitter/main.py --video_source 0 --host_ip 127.0.0.1 --host_port 4900
```

or

```bash
python ./pose_transmitter/main.py --video_source some_video.mp4
```

or just:

```bash
python ./pose_transmitter/main.py --debug
```

The results are something like:

{{< figure
    alt="Code screenshot"
    align=center
    caption=""
    src="imgs/10_results.webp"
>}}

## • Notes

**Performance:** It is **not** really good. I am getting around 30FPS using my webcam (less than that as you can see in the example) which for this experiment [is enough](https://www.neogaf.com/threads/the-human-eye-cant-see-more-than-30fps-when-there-are-100-frames-or-more.1471501/).  
In the [TensorFlow JS Demo](https://storage.googleapis.com/tfjs-models/demos/pose-detection/index.html?model=movenet), I am getting 3x times that. Same machine, on the browser, Javascript… You get the idea...

Even in Python, I am pretty sure that there is space for improvement. Maybe the fact that I am extracting the frame, then doing the inference, and then extracting the next frame.  
Couldn’t be parallel tasks? Not sure how the Global Interpreter Lock will limit this, but is an interesting exercise that is pending.

—-

Finally, the code is in [this repo](https://github.com/mayo-nesso/pose-transmitter), all ideas, suggestions, and comments are welcome!

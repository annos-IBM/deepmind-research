--------------------------------------------------------------------------------

# CMTouch Dataset

<!-- ![downloads](https://img.shields.io/github/downloads/atom/atom/total.svg)
![build](https://img.shields.io/appveyor/ci/:user/:repo.svg)
![chat](https://img.shields.io/discord/:serverId.svg)
 -->

This repository contains datasets for cross-modal representation learning, used
in developing rich touch representations in "Learning rich touch representations
through cross-modal self-supervision" [1].

The datasets we provide are:

1.  CMTouch-Props
2.  CMTouch-YCB

The datasets consist of episodes collected by running a reinforcement learning
agent on a simulated Shadow Dexterous Hand [2] interacting with different
objects. From this interactions, observations from different sensory modalities
are collected at each time step, including vision, proprioception (joint
positions and velocities), touch, actions, object IDs. We used these data to
learn rich touch representations using cross-modal self-supervision.

## Bibtex

If you use one of these datasets in your work, please cite the reference paper
as follows:

```
@InProceedings{zambelli20learning,
author = "Zambelli, Martina and Aytar, Yusuf and Visin, Francesco and Zhou, Yuxiang and Hadsell, Raia",
title = "Learning rich touch representations through cross-modal self-supervision",
year = "2020",
}
```

<!--
@misc{cmtouchdatasets}, title={CMTouch Datasets}, author={Zambelli, Martina and
Aytar, Yusuf and Visin, Francesco and Zhou, Yuxiang and Hadsell, Raia},
howpublished={https://github.com/deepmind/deepmind-research/tree/master/cmtouch},
year={2020} }
-->

## Descriptions

### Experimental setup

We run experiments in simulation with MuJoCo [3] and we use the simulated Shadow
Dexterous Hand [2], with five fingers and 24 degrees of freedom, actuated by 20
motors. In simulation, each fingertip has a spatial touch sensor attached with a
spatial resolution of 4×4 and three channels: one for normal force and two for
tangential forces. We simplify this by summing across the spatial dimensions,
to obtain a single force vector for each fingertip representing one normal force
and two tangential forces. The state consists of proprioception (joint positions
and joint velocities) and touch.

Visual inputs are collected with a 64×64 resolution and are only used for
representation learning, but are not provided as observations to control the
robot’s actions. The action space is 20-dimensional. We use velocity control and
a control rate of 30 Hz. Each episode has 200 time steps, which correspond to
about 6 seconds. The environment consists of the Shadow Hand, facing down, and
interacting with different objects. These objects have different shapes, sizes
and physical properties (e.g. rigid or soft). We develop two versions of the
task, the first using simple props and the second using YCB objects. In both
cases, objects are fixed to their frame of reference, while their position and
orientation are randomized.

### CMTouch-Props

This is a dataset based on simple geometric 3D shapes (referred to as "props").
Props are simple 3D shaped objects that include cubes, spheres, cylinders and
ellipsoid of different sizes. We also generated the soft version of each prop,
which can deform under the pressure of the touching fingers.

Soft deformable objects are complex entities to simulate: they are defined
through a composition of multiple bodies (capsules) that are tied together to
form a shape, such as a cube or a sphere. The main characteristic of these
objects is their elastic behaviour, that is they change shape when touched. The
most difficult thing to simulate in this context is contacts, which grow
exponentially with the increased number of colliding bodies.

Forty-eight different objects are generated by sampling from 6 different sizes,
4 different shapes (i.e. sphere, cylinder, cube, ellipsoid), and they can either
be rigid or soft.

![](https://i.imgur.com/Hps38z5.jpg)

### CMTouch-YCB

This is a dataset based on YCB objects. The YCB objects dataset [4] consists of
everyday objects with different shapes, sizes, textures, weight and rigidity.

We chose a set of ten objects: cracker box, sugar box, mustard bottle, potted
meat can, banana, pitcher base, bleach cleanser, mug, power drill, scissors.
These are generated in simulation at their standard size, which is also
proportionate to the default dimension of the simulated Shadow Hand.

The pose of each object is randomly selected among a set of 60 different poses,
where we vary the orientation of the object. These variations make the
identification of each object more complex than the CMTouch-Props and require a
higher generalization capability from the learning method applied.

![](https://i.imgur.com/Mf3KYbn.jpg)

## Download

The datasets can be downloaded from
[Google Cloud Storage](https://console.cloud.google.com/storage/browser/dm_cmtouch).
Each dataset is a single
[TFRecord](https://www.tensorflow.org/tutorials/load_data/tfrecord) file.

On Linux, to download a particular dataset, use the web interface, or run `wget`
with the appropriate filename as follows:

```
wget https://storage.googleapis.com/dm_cmtouch/datasets/cmtouch_props_all_test.tfrecords
wget https://storage.googleapis.com/dm_cmtouch/datasets/cmtouch_props_all_train.tfrecords
wget https://storage.googleapis.com/dm_cmtouch/datasets/cmtouch_props_all_val.tfrecords
wget https://storage.googleapis.com/dm_cmtouch/datasets/cmtouch_ycb_all_test.tfrecords
wget https://storage.googleapis.com/dm_cmtouch/datasets/cmtouch_ycb_all_train.tfrecords
wget https://storage.googleapis.com/dm_cmtouch/datasets/cmtouch_ycb_all_val.tfrecords
```


## Usage

After downloading the dataset files, you can read them as `tf.data.Dataset`
instances with the readers provided. The example below shows how to read the
cmtouch-props dataset:

```
record_file = 'test.tfrecords'
dataset = tf.data.TFRecordDataset(record_file)
parsed_dataset = dataset.map(_parse_tf_example)
```

(a complete example is provided in the Colab).

All dataset readers return the following set of observations:

'camera': tf.io.FixedLenFeature([], tf.string),
    'camera/height': tf.io.FixedLenFeature([], tf.int64),
    'camera/width': tf.io.FixedLenFeature([], tf.int64),
    'camera/channel': tf.io.FixedLenFeature([], tf.int64),
    'object_id': tf.io.FixedLenFeature([], tf.string),  # for both
    'object_id/dim': tf.io.FixedLenFeature([], tf.int64),
    'orientation_id': tf.io.FixedLenFeature([], tf.string),  # only for ycb
    'orientation_id/dim': tf.io.FixedLenFeature([], tf.int64),
    'shadowhand_motor/joints_vel': tf.io.FixedLenFeature([], tf.string),
    'shadowhand_motor/joints_vel/dim': tf.io.FixedLenFeature([], tf.int64),
    'shadowhand_motor/joints_pos': tf.io.FixedLenFeature([], tf.string),
    'shadowhand_motor/joints_pos/dim': tf.io.FixedLenFeature([], tf.int64),
    'shadowhand_motor/spatial_touch': tf.io.FixedLenFeature([], tf.string),
    'shadowhand_motor/spatial_touch/dim': tf.io.FixedLenFeature([], tf.int64),
    'actions'

*   'camera': `Tensor` of shape [sequence_length, height, width, channels] and type
    uint8

*   'shadowhand_motor/spatial_touch': `Tensor` of shape [sequence_length, num_fingers x 3] and type float32

*   'shadowhand_motor/joints_pos': `Tensor` of shape [sequence_length, num_joint_positions] and type float32

*   'shadowhand_motor/joints_vel': `Tensor` of shape [sequence_length,
    num_joint_velocities] and type float32

*   'actions': `Tensor` of shape [sequence_length, num_actuated_joints] and type
    float32

*   'object_id': `Scalar` indicating an object identification number

*   'orientation_id': `Scalar` indicating a YCB object pose identification number
    (CMTouch-YCB only)

Few-shot evaluations can be made by creating subsets of data to train and
evaluate the models.

<!--
```diff=
- TODO
```
-->

## References

[1] M. Zambelli, Y. Aytar, F. Visin, Y. Zhou, R. Hadsell. Learning rich touch
representations through cross-modal self-supervision. Conference on Robot
Learning (CoRL), 2020.

[2] ShadowRobot, Shadow Dexterous Hand.
https://www.shadowrobot.com/products/dexterous-hand/.

[3] E. Todorov, T. Erez, and Y. Tassa. MuJoCo: A physics engine for model-based
control. In Proceedings of the International Conference on Intelligent Robots
and Systems (IROS), 2012.

[4] B. Calli, A. Singh, J. Bruce, A. Walsman, K. Konolige, S. Srinivasa, P.
Abbeel, and A. M. Dollar. Yale-cmu-berkeley dataset for robotic manipulation
research. The International Journal of RoboticsResearch, 36(3):261–268, 2017.

## Disclaimers

This is not an official Google product.

## Appendix and FAQ

**Find this document incomplete?** Leave a comment!
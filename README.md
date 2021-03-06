# Table Tennis Ball Prediction

[![DOI](https://zenodo.org/badge/85822225.svg)](https://zenodo.org/badge/latestdoi/85822225)

## LICENSE
MIT License

Please cite this github repository with the DOI above if it helps your research.

---

Implementation of **Highway network + LSTM + MDN** to perform prediction of flying table tennis ball


A brief report can be found in `report/table_tennis_prediciton.pdf`

Please refer to [this branch](https://github.com/taochenshh/HLM-Pred/tree/better_gazebo_model) to see the **detailed neural network structure**


---

**Performance**

[![Palying Table Tennis with DNN](https://j.gifs.com/nZA9Vl.gif)](https://youtu.be/FS1Ry3mMRWM)


---

## IO

**Input**:

Intial several camera views of flying table tennis

**Output**:

Ball's position in a single future frame or several future frames

---
## Mission

**Given** a user-specified number of initial frames of camera **images** of flying table tennis ball, the algorithm here can **predict** the cartesian coordinate of **table tennis ball** in the future frames in the world coordinate system. 

For example, after training, the algorithm can predict the position of table tennis ball 34 frames later after it received the initial 14 frames of table tennis ball flying camera views. That's to say, after a table tennis ball is served, a camera will be recording visual images of this scene periodically 30 Hz). And the algorithm can predict the ball position in the 38th (14 + 24) frames, i.e., the ball's position after 1.6s since it's launched. Based on the prediction, the robot will be able to kick back the ball.

The algorithm is able to accurately predict the future (like 24 frames later) ball's position based on the initial 14 consecutive camera views. Plus, it can also **predict ball's position in several future frames** (like future 6 frames) **simultaneously** based on these 14 initial frames, i.e., ball's positions in 19th to 24th frames.


---

## Neural Network Structure

**Highway Networks**
![](figures/highway.png)

**Neural Network for Single Frame Prediction**
![](figures/s_nn.png)

**Neural Network for Multiple Frame Predictions**
![](figures/m_nn.png)

---

## Prediction Animation

**Single-Frame Prediction**:

Input starting frame=1:

![Input starting frame=1](figures/test_single_1.png)

Input starting frame=2:

![Input starting frame=2](figures/test_single_2.png)

Input starting frame=3:

![Input starting frame=3](figures/test_single_3.png)


**Multi-Frames Prediction**:

Input starting frame=1:

![Input starting frame=1](figures/test_multi_1.png)

Input starting frame=2:

![Input starting frame=2](figures/test_multi_2.png)

Input starting frame=3:

![Input starting frame=3](figures/test_multi_3.png)


---

## Package Requirements:
* [TensorFlow 1.0](https://www.tensorflow.org/install/install_linux)
```bash
pip install tensorflow-gpu
```

* [TensorLayer](http://tensorlayer.readthedocs.io/en/latest/)
```bash
pip install git+https://github.com/zsdonghao/tensorlayer.git
```
* [sklearn](http://scikit-learn.org/)
```bash
pip install scikit-learn
```

---

## Run
```bash
cd scripts
python main.py
```

**Single frame prediction training**:
```bash
python main.py -fd 4 --num_cells=2 -wd=0.00003 -sl=14 --keep_prob=0.75
```

**Testing**:
```bash
python main.py -fd 4 --num_cells=2 -wd=0.00003 -sl=14 --keep_prob=0.75 --test
```

**Multiple frames prediction training**:
```bash
python main.py --train_condition=multiframe_pred -fd=4 --num_cells=2 -wd=0.00003 -sl=14 --keep_prob=0.75
```

**Testing**:
```bash
python main.py --train_condition=multiframe_pred -fd=4 --num_cells=2 -wd=0.00003 -sl=14 --keep_prob=0.75 --test
```

### Control UR robot

First, compile UR ROS package:
```bash
cd <path_to_HLM-Pred>/pingpong-simulation
catkin_make
```
Put `ping_pong_ball` and `table` folders in `model` folder into `~/.gazebo/models`.

Also, change the computer name in `pingpong-simulation/src/pingpang_gazebo/worlds/table_tennis.world`: `[<uri>file:///home/...]`

```bash
gedit ~/.bashrc
```
Add `source <path_to_HLM-Pred>/pingpong-simulation/devel/setup.bash` to the end of `bashrc`

To launch the table tennis environment, run:
```bash
roslaunch pingpang_gazebo pingpang_AI.launch
```

Then, run the prediction script in `<path_to_HLM-Pred>/scripts`

**Single frame prediction**:
```bash
cd <path_to_HLM-Pred>/scripts/
python main.py --train_condition=real_time_play -fd 4 --num_cells=2 -wd=0.00003 -sl=14 --keep_prob=0.75
```

**Multiple frame prediction**:
```bash
python main.py --train_condition=real_time_play -fd=4 --num_cells=2 -wd=0.00003 -sl=14 --keep_prob=0.75 -md
```

These are just example usages, modify the arguments according to your own needs.

After this, make UR robot play table tennis by running:
```bash
cd <path_to_HLM-Pred>/pingpong-simulation/src/pingpang_control/scripts
python play_table_tennis.py
```


## Optional arguments to pass in when run main.py
```bash
usage: main.py [-h]
               [--train_condition {toy_test,coordinate_input,center_pixel_input,real_time_play,multiframe_pred}]
               [--train_dir TRAIN_DIR] [--restore_training]
               [--restore_step RESTORE_STEP] [--test] [--rnn_cell {lstm,gru}]
               [--num_cells NUM_CELLS] [--num_mixtures NUM_MIXTURES]
               [--seq_length SEQ_LENGTH] [--gaussian_dim GAUSSIAN_DIM]
               [--features_dim FEATURES_DIM] [--weight_decay WEIGHT_DECAY]
               [--learning_rate LEARNING_RATE] [--batch_size BATCH_SIZE]
               [--num_epochs NUM_EPOCHS] [--keep_prob KEEP_PROB]
               [--grad_clip GRAD_CLIP] [--random_seed RANDOM_SEED]
               [--ckpt_interval CKPT_INTERVAL]
               [--predicted_step PREDICTED_STEP]
               [--pred_frames_num PRED_FRAMES_NUM] [--multi_pred]

Run the Table Tennis Ball Prediction algorithm.

optional arguments:
  -h, --help            show this help message and exit
  --train_condition {toy_test,coordinate_input,center_pixel_input,real_time_play,multiframe_pred}, 
  -tc {toy_test,coordinate_input,center_pixel_input,real_time_play,multiframe_pred}
                        train_condition: 'toy_test', 'coordinate_input',
                        'center_pixel_input','multiframe_pred' or
                        'real_time_play'
  --train_dir TRAIN_DIR
                        path to the training directory.
  --restore_training, -rt
                        restore training
  --restore_step RESTORE_STEP, -rs RESTORE_STEP
                        checkpoint file to restore
  --test                run algorithm in testing mode
  --rnn_cell {lstm,gru}
                        type of rnn cell: 'lstm' or 'gru'
  --num_cells NUM_CELLS
                        number of rnn cells
  --num_mixtures NUM_MIXTURES
                        number of mixtures for MDN
  --seq_length SEQ_LENGTH, -sl SEQ_LENGTH
                        sequence length
  --gaussian_dim GAUSSIAN_DIM, -gd GAUSSIAN_DIM
                        dimentionality of gaussian distribution
  --features_dim FEATURES_DIM, -fd FEATURES_DIM
                        dimentionality of input features
  --weight_decay WEIGHT_DECAY, -wd WEIGHT_DECAY
                        scale factor of regularization of neural network
  --learning_rate LEARNING_RATE, -lr LEARNING_RATE
                        learning rate
  --batch_size BATCH_SIZE
                        minibatch size
  --num_epochs NUM_EPOCHS
                        number of epochs
  --keep_prob KEEP_PROB
                        dropout keep probability
  --grad_clip GRAD_CLIP
                        clip gradients at this value
  --random_seed RANDOM_SEED
                        random seed
  --ckpt_interval CKPT_INTERVAL
                        checkpoint file save interval
  --predicted_step PREDICTED_STEP, -ps PREDICTED_STEP
                        which time step after the last sequence input to make
                        prediction
  --pred_frames_num PRED_FRAMES_NUM, -pfn PRED_FRAMES_NUM
                        if predicting multiple frames, how many frames to
                        predict
  --multi_pred, -md     if in real_time_play mode, choose whether to predict
                        single frame or multiple frames

```


## Training Result:

**Single frame prediction**: given 14 initial camera frames, predict the position of table tennis ball in the 24th frame starting from the last frame (the 14th frame input), i.e., the ball's position in the 47th (14 + 33) frame
```bash
Percentage of testing with distance less than 0.010m is: 57.22 %
Percentage of testing with distance less than 0.020m is: 82.62 %
Percentage of testing with distance less than 0.030m is: 91.17 %
```

**Multiple frames prediction**: given 14 initial camera frames, predict the position of table tennis ball in the 19th to the 24th frames starting from the last frame (the 14th frame input), i.e., the ball's position in the 33th to 38th frame

19th frame：
```bash
Time step 0 Distances statistics:
Percentage with distance less than 0.010m is: 68.72 %
Percentage with distance less than 0.020m is: 89.21 %
Percentage with distance less than 0.030m is: 96.33 %
```

20th frame：
```bash
Time step 1 Distances statistics:
Percentage with distance less than 0.010m is: 65.21 %
Percentage with distance less than 0.020m is: 87.85 %
Percentage with distance less than 0.030m is: 96.33 %
```

21th frame：
```bash
Percentage with distance less than 0.010m is: 64.35 %
Percentage with distance less than 0.020m is: 87.36 %
Percentage with distance less than 0.030m is: 95.64 %
```

22th frame：
```bash
Time step 3 Distances statistics:
Percentage with distance less than 0.010m is: 61.85 %
Percentage with distance less than 0.020m is: 86.14 %
Percentage with distance less than 0.030m is: 94.64 %
```

23th frame：
```bash
Time step 4 Distances statistics:
Percentage with distance less than 0.010m is: 60.36 %
Percentage with distance less than 0.020m is: 85.56 %
Percentage with distance less than 0.030m is: 94.32 %
```

24th frame：
```bash
Time step 5 Distances statistics:
Percentage with distance less than 0.010m is: 56.74 %
Percentage with distance less than 0.020m is: 83.32 %
Percentage with distance less than 0.030m is: 93.57 %
```

# SRTP_Alpha_CC_Zero

- Alpha CC Zero: A Deep Reinforcement Learning Model for Chinese Chess
- Environment:

```
Python 3.6.13
cuda 11.2
tensorflow-gpu 2.6.2
pillow 8.4.0
scipy
uvloop
```

- How to run:
- Just play

```
python main.py --mode play --processor gpu --num_gpus 1 --ai_function mcts --ai_count 1 
```

- Multiple processes train

```
python main.py --mode distributed_train --processor gpu --train_playout 400 --res_block_nums 9 --train_epoch 100 --batch_size 256 --mcts_num 8
```

- Evaluate (Compute elo)

```
python main.py --mode eval --processor gpu --play_playout 40 --res_block_nums 9 --eval_num 1 --game_num 10
```

- Self-Play & Network Training

![1_reverse](https://user-images.githubusercontent.com/67775090/187147719-3edd4e5e-a76e-465d-99a7-a694bfb6710d.png)

- Real Play (Net + MCTS)

![2_reverse](https://user-images.githubusercontent.com/67775090/187147789-cd494e7f-7508-44de-b28f-fc2c80d71886.png)

- Network Structure

![image](https://user-images.githubusercontent.com/67775090/188292248-1cc34df8-7430-4c9e-8b81-e9e8585bfcca.png)

- Reference

[1] Silver, David, Schrittwieser, Julian, Simonyan, Karen, Antonoglou, Ioannis, Huang, Aja, Guez, Arthur, Hubert, Thomas, Baker, Lucas, Lai, Matthew, Bolton, Adrian, Chen, Yutian, Lillicrap, Timothy, Hui, Fan, Sifre, Laurent, van den Driessche, George, Graepel, Thore and Hassabis, Demis. "Mastering the game of Go without human knowledge." Nature 550 (2017): 354--.

# DRLND Project 3 Report (Collaboration and Competition)

![Trained Agent][image1]

In this project, two agents control rackets to bounce a ball over a net. If an agent hits the ball over the net, it receives a reward of +0.1. If an agent lets a ball hit the ground or hits the ball out of bounds, it receives a reward of -0.01. Thus, the goal of each agent is to keep the ball in play.

## Learning Algorithm

The [MAPPDG algorithm](https://arxiv.org/pdf/1706.02275.pdf) was introduced as an extension of DDPG for multi-agent environments. One simple way to think of MADDPG is as a wrapper for handling multiple DDPG agents. But the power of this algorithm is that it adopts a framework of centralized training and decentralized execution. This means that there is extra information used during training that is not used during testing. More specifically, the training process makes use of both actors and critics, just like DDPG. The difference is that the input to each agent's critic consists of all the observations and actions for all the agents combined. However, since only the actor is present during testing, that extra information used during training effectively goes away. This framework makes MADDPG flexible enough to handle competitive, collaborative, and mixed environments.

Because of the similarities with DDPG, the hyperparamters used in MADDPG are very similar, and are listed below:

```
BUFFER_SIZE                 # replay buffer size
BATCH_SIZE                  # minibatch size
GAMMA                       # discount factor
TAU                         # for soft update of target parameters
LR_ACTOR                    # learning rate of the actor 
LR_CRITIC                   # learning rate of the critic
WEIGHT_DECAY                # L2 weight decay
UPDATE_EVERY                # weight update frequency
NOISE_AMPLIFICATION         # exploration noise amplification
NOISE_AMPLIFICATION_DECAY   # noise amplification decay
```

The last two hyperparameters were added to amplify the exploration noise used by the agent in an effort to learn the optimal policy faster.

In MADDPG, each agent has its own actor local, actor target, critic local and critic target networks. The model architecture for MADDPG resembles DDPG very closely. The difference is that I found better results when concatenating the states and actions as the input to the critic's first hidden layer. DDPG, on the other hand, concatenates the actions to the input of the second hidden layer, after the states have already gone through in the first hidden layer.

### Code implementation

The codes consist of 2 files:

- `model.py` : Implement the **Actor** and the **Critic** class
    - Both Actor and Critic class implement a *Target* and a *Local* Neural Network for training.
    
- `Tennis.ipynb` : 
    - Import the necessary packages 
    - Check the state and action spaces
    - The Actor's *Local* and *Target* neural networks, and the Critic's *Local* and *Target* neural networks, the Noise process and the Replay Buffer are instanciated by the DDPG Agent.
    - The Noise uses Ornstein-Uhlenbeck process.
    - The Replay Buffer is a fixed-size buffer to store experience tuples.
    - Create MADDPG class
    - Train 2 agents using MADDPG
    - Plot the scores/rewards
  
### Hyperparameters

The MADDPG agent uses the following hyperparameters:

```
BUFFER_SIZE = int(1e6)  # replay buffer size
BATCH_SIZE = 256        # minibatch size
GAMMA = 0.99            # discount factor
TAU = 1e-3              # for soft update of target parameters
LR_ACTOR = 1e-3         # learning rate of the actor 
LR_CRITIC = 1e-3        # learning rate of the critic
WEIGHT_DECAY = 0        # L2 weight decay
LEARN_EVERY = 1         # update the networks 3 times after every timestep
LEARN_NUMBER = 3        # update the networks 3 times after every timestep
EPSILON = 1.0           # noise factor
EPSILON_DECAY = 0.99    # noise factor decay
CLIPGRAD = .1           # clip gradient parameter
seed = 10               # random seed
```

The **Actor Neural Networks** use the following architecture :

```
Input Layer (3*8=24) ->
Fully Connected Hidden Layer (450 nodes, Batch Normlization, relu activation) ->
Fully Connected Hidden Layer (350 nodes, Batch Normlization, relu activation) ->
Ouput Layer (2 nodes, tanh activation)
```


The **Critic Neural Networks** use the following architecture :

```
Input Layer (24) ->
Fully Connected Hidden Layer (256+2 nodes [including actions], Batch Normlization, relu activation) ->
Fully Connected Hidden Layer (128 nodes, Batch Normlization, relu activation) ->
Ouput Layer (1 node, no activation)
```

Some important changes:

- I add noise decay factor *EPSILON_DECAY* for the noise factor *EPSILON*.
- I use gradient clipping when training the critic network `clip_grad_norm_(self.critic.parameters(), self.clipgrad)`.
- I update the actor and critic networks 3 times after every timestep.

## Results

With all these hyperparameters and Neural Networks, the result is quite good:

![Score](images/score.jpg)

**The result satisfies the goal of this project as the average (over 20 episodes) of those scores is at least +0.5, and in 277 episodes only**. 
```

## Future Work Ideas

- Changing the hyperparameters used to acheive target score faster.
- Implement different algorithm for different agent and make a match for these two agents to see which algorithm performs better.
- Implement [Twin Delayed DDPG (TD3)](https://spinningup.openai.com/en/latest/algorithms/td3.html) for this multi-agent environment to improve the performance of agents.
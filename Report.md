[//]: # (Image References)

[image1]: rewards.png "Plot Reward"

# Project 1: Navigation

My Deep Q-Learning algorithm (`Navigation.ipynb`) is based on an Artificial Neural Network consisting of 3 Linear Layers with ReLu activation each.
For mapping the 37 space dimensions to 4 discrete actions the input layer has size 37 x 64, the first hidden layer has size 64 x 64 and the final output layer has dimension 64 x 4.
In order to solve the Banana environment (i.e. maximizing the total reward by picking as many yellow bananas as possible in each period) the agent follows a combination of policy evaluation and policy improvement in each iteration. The first term means picking an action based on an epsilon-greedy policy. Either the agent picks a random action (exploration) or decides for an action based on the argmax of the output of a simple forward pass for a given state (i.e. the action which maximizes the state-action value function for a given state -> exploitation). The exploration-exploitation tradeoff is controlled by the hyperparameter epsilon. It is initially set to 1.0 and is decreased by a factor of 0.995 after each episode. Epsilon is lowered until it becomes 0.1 and is fixed after that.
On the other hand policy improvement is built of two phases - a sampling phase and a learning phase. Each timestep a so called Replay Buffer (BUFFER_SIZE = 1e5) is extended by important information like state, action, reward, next state and the fact whether the end of the game is reached. By random sampling a certain number of instances (BATCH_SIZE = 64) from this buffer and providing it to the agent the agent tries to improve the model weights of the DQN based on these experience replay data. Technically, this means carrying out the Backpropagation algorithm with the goal to minimize the loss between target state action values and predicted state action values. The idea is to replicate the DQN architecture at the beginning and to make a soft update to the weights of the target network by copying over the weights of the local network. The tradeoff between old target parameters and local ones is handled by hyperparameter tau. In the underlying training procedure tau is chosen to be 1e-3. For the sake of completeness the state action values of the target network for given states follows the Bellman equation (i.e. considering the rewards plus gamma times the maximum predicted Q values for the next states).
As it is usual in backprop algorithm, after calculation of the forward run of the local network respectively forward run and consecutive Belman equation of the target network, the loss can be computed. I decided to choose a Mean Squared Error Loss. Taking the gradient of the loss w.r.t. model parameters and feeding it back through the network the optimizer (Adam) is finally able to update the weights. In this context the hyperparameter LR is the step size for updating the weights. It is set to LR = 5e-4. In practice, it shouldn't be set too high because one could face the problem of getting stuck in local minimas and thus not reaching the global optimimum. On the other hand, too small values lead to very slow learning behavior - in the worst case not learning at all.
All in all, each iteration of an episode the agent picks an action (policy evaluation), interacts with its environment (thus receiving the next state, reward and the information if the game is done) and learns from experience replay data.
It should be mentioned that learning is carried out every 4th iteration.
In total, the agent plays 2000 episodes each terminating either after 1000 iterations are reached or if the game is done. The agent is able to solve the environment in 521 episodes (i.e. achieving an average reward of 13.04 for 100 consecutive episodes).
The trained model weights are stored in `checkpoint.pth`. The following plot shows the scores during training.

![Plot Reward][image1]

Compared to the hint that training should take around 1800 episodes until the environment is solved my results are very good.
Of course, one could even improve performance by e.g. doing a hyperparameter optimization or by implementing a prioritized experience replay focusing on most important samples.
In my case, I already did further investigation regarding navigation in the more challenging environment when the agent should learn from raw pixels directly. I changed the model to a convolutional neural network taking as input the four most recent frames which have been stacked together to grayscale images. However, after a certain amount of episodes the Ipython notebook `Navigation_Pixels_stackedInput.ipynb` crashes which seems to be a problem with Unity environment. Therefore, I cannot make a reliable statement if the agent is learning from its environment.
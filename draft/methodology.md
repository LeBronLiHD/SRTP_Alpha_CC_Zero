# Monte Carlo Tree Search

Monte Carlo Tree Search (MCTS) is a search algorithm widely used in chess engines. Its implementation process is similar to human chess. It will concentrate on analyzing certain moves that seem to be most beneficial to oneself, and will not waste too much energy on some moves that conform to the rules but seem unreasonable. At the same time, it will also deduce the same way before making a decision, by enumerating possible future situations, and then evaluating the current various moves. Due to the huge search space of Go, MCTS has great advantages over other algorithms, and can greatly reduce the amount of calculation while basically ensuring the accuracy.  The implementation steps of MCTS are mainly to re-express the decision problem through the tree structure, and then use the Monte Carlo method to search. In each deduction, the current state of the chess surface will be used as the root node, and the extended child nodes will be searched downward according to certain rules. The results of the simulation are then back-propagated from the child nodes to the root node and the statistics corresponding to the root node are updated. The condition for finally jumping out of the search may be reaching the set search times limit or time limit.

The essence of Monte Carlo tree search is to maintain a tree, and each node on the tree stores the statistics of the situation $s$ corresponding to the node. These statistics include the number of visits of the node $N(s,m)$, the probability of being selected $P(s,m)$, the total action value $W(s,m)$  and the average action value $Q(s,m)$. The deep neural network $f_{\theta}$ participates and guides in the search process. The network combines the policy network and the value network, and returns the move vector $p$ and the position winning probability estimate $v$ to the Monte Carlo tree based on the current position information $s$, where the move vector is the move probability of each legal action in the current situation. Monte Carlo tree search conducts the search under the guidance of the move vector $p$ and the position winning probability estimate $v$, which can make the search more directional and reduce the computational complexity of the search.  Figure 1 is an example of the overall search process of a Monte Carlo tree. The entire search process can be divided into the following 4 steps: selection, expansion, evaluation and return, and move.

## Selection
In each simulation, the MCTS chooses a move $m$ based on the position $s$. Taking the situation state of the root node $s_r$ as an example, MCTS will calculate the $Q(s,m)+U(s,m)$ value corresponding to each legal move in the current situation according to the statistical data stored in the node, and select the value. The largest move $m$ is used as the search direction. $Q(s,m)$ is the average action value mentioned above, which will be described later. The formula of $U(s,m)$ is shown in the following formula $(1)$. For the move selection of any expanded node, you can choose according to the method of this step.
$$
U(s,m) = c_{puct}P(s,m)\frac {\sqrt {\sum_b N(s, b)}} {1 + N(s,m)} \tag1
$$
$\sum_b N(s, b)$ in the formula $(1)$ represents the number of times the root node is visited; $N(s,m)$ is the number of visits from the root node to a child node; $P( s,m)$ is the prior probability of moves output by the deep neural network $f_{\theta}$; $c_{puct}$ is a hyperparameter. It is easy to see from the formula that at the beginning of the search, $U(s,m)$ is larger, that is, the search strategy tends to search for moves with high prior probability and low number of visits; as the total number of explorations increases, $U(s,m)$ gradually decreases, that is, the search strategy gradually begins to choose moves with high action value. Therefore, it is not difficult to see that $c_{puct}$ is used to determine the exploration level. When the value is large, MCTS will tend to explore unsimulated moves, and when the value is small, the convergence speed will be accelerated.

## Expansion
After the selection step, the MCTS advances from the root node to a leaf node whose situational state is $s_l$ . At this time, since the leaf nodes are not expanded and do not store the corresponding statistical data, they need to be expanded and evaluated with the deep neural network $f_{\theta}$. $f_{\theta}$ Input the current position $s_l$ , and output the prior probability $p_l$ of the move corresponding to the position and the estimated winning rate of the current position $v_l$. Then use $p_l$ to expand the leaf node, and the statistical data stored in each legal move $(s_l, m)$ of the node is initialized as: $\{N(s_l, m)=0, W(s_l, m)=0, Q(s_l, m)=0, P(s_l, m)=p_l\}$, and the value $v_l$ is returned. Since the expansion of this leaf node is completed, go to the next step.

## Evaluation and Return
Every time MCTS takes a step $a$, the statistics of the edges traveled are updated in reverse. The formulas of this update process are shown in the following equations (2), (3) and (4).

$$
N(s_a,m_a) = N(s_a,m_a) + 1 \tag2
$$

$$
W(s_a, m_a) = W(s_a, m_a) + v_a \tag3
$$

$$
Q(s_a, m_a) = \frac {W(s_a, m_a)} {N(s_a, m_a)}  \tag4
$$

The execution process of MCTS is essentially a recursive process that selects the expanded nodes. There are two ending conditions for recursion: one is to encounter a leaf node, and the MCTS will expand the node at this time to obtain the neural network's output position estimate $v_l$; the other is that the chess game is divided into a victory Negative, get the value $v$ of the win, draw and lose. The above process is the evaluation. After that, MCTS will pass $v_l$ or $v$ back to the nodes it passed before, and update the statistics of those nodes.

## Move
Until the MCTS ends its search, it will continue to perform the previous steps and play against itself. The condition for finally jumping out of MCTS can be a time limit or a limit on the number of searches. When the constraints are reached, the statistical information of each node edge is generally relatively complete, and it can basically cover most of the moves and chess games. At this point, MCTS has reached the real decision-making stage, and it will use the statistical data stored on the edge of the root node $s_r$ to calculate the probability of each move $\pi(m|s_r)$ Choose a move $m_r$, $\pi The calculation of (m|s_r)$ is determined by the following formula (5):
$$
\pi(m|s_r) = \frac {N(s_r, m)^{\frac 1 \tau}} {\sum_b N(s_r,b)^{\frac 1 \tau }} \tag5
$$
 Among them, $\tau$ is a coefficient about the exploration level. After the MCTS executes the move $m_r$, the subtree data of the previous root node $s_r$ except the subtree corresponding to $m_r$ will be discarded, and the data of the subtree corresponding to $m_r$ will be retained. At this time, the child node reached by MCTS becomes the root node, and the next round of simulation is restarted.
 
# deep neural network

## input features

Chess pieces are divided into 7 types of chess pieces: rooks, horses, guns, elephants, soldiers, generals, and pawns. Each side has 7 characteristic planes, and a chess game has a total of 14 characteristic planes. Other feature planes include (to be added: color, number of rounds, repeating positions, historical move records, and other features). The checkerboard size is 9∗10, so the input feature dimension is 9×10×N.

## network structure

The network structure adopted by this algorithm combines the strategy network and the value network. The main body of the network is composed of a convolution block and multiple residual blocks, and the residual block is followed by a strategy header and a value header respectively.



The convolution block uses 256 convolution kernels to extract features. The size of the convolution kernel is 3×3 and the stride is 1. After the convolution operation, batch normalization is performed, and then the Relu activation function is used.



The internal structure of the residual block is similar to the convolution block. One residual block contains two convolution layers, and a shortcut is added before the input and the last activation function to form a residual structure.



The strategy head uses the softmax activation function to output the probability of all possible moves, and the value head uses the tanh activation function to output the winning rate of the mover in the current situation.



## Network training process

Firstly, the chess game data used for network training is obtained through the self-play process. In this process, use the latest neural network parameters $f_{\theta}$ to complete self-play with MTCS, each move is determined by the action probability vector π obtained after N times of simulation, until the winner is determined and the winner is obtained. result z. After a large number of self-play games, the position s of each move in each chess game, the calculated action probability vector π and the game result z are saved, and the training neural network chess score data is obtained.



The input of the neural network is the position s, and the output is the predicted action probability vector p and the win rate v, namely: $$ f_{\theta}(p, v)=f_{\theta_{i}}(s) $$



The training goal of the neural network is to make the output of the neural network closer to the result obtained by the simulated search, that is, to minimize the error between the move probability p and the search probability π and the predicted winning rate v and the game result z, so as to improve the neural network's move. Level. Therefore, the following loss function is defined:
$$
l=(z-v)^{2}-\pi^{T} \log p+c\|\theta\|^{2}
$$
The first term is the mean square error between the predicted winning rate v and the game result z, the second term is the cross entropy loss between the move probability p and the search probability π, and the third term is the L2 regularization term to prevent parameter overfitting , c is a hyper parameter that adjusts the regularization level. 

 Use the chess data obtained in the self-playing session to train the neural network, so that the network parameter θ is continuously updated by the gradient descent through the loss function l, and the latest neural network parameter $f_{\theta}$ is obtained after the training is completed, which is used for the next round. Iterative self-play process until neural network parameters with sufficiently high chess playing level are obtained.

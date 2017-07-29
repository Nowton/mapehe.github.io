---
layout: post
title: Solving CartPole-v0 with TensorFlow
---

In case you haven’t heard of it, [OpenAI Gym](https://gym.openai.com/) is a really neat tool
that enables you to quickly get hands on with your favorite machine
learning architecture on several Atari games, board games and all kinds
of other toy models of varying difficulty. This [Cart Pole game](https://gym.openai.com/envs/CartPole-v0)
seems kind of an entry level thing, so I solved it (leaning heavily on [this](http://karpathy.github.io/2016/05/31/rl/) excellent
post by Andrej Karpathy) and posted the script on [GitHub](https://github.com/mapehe/cart_pole).

The game looks like this:

{: style="text-align:center"}

![alt text]({{site.url}}/images/cart_pole.gif  "Cart Pole illustration")

You try to balance the pole by moving the cart left and right, if the balance is lost, the game cuts back to the beginning.
You "win" when you survive for 200 frames. The problem is considered solved when you survive for 195 frames on average
over 100 consecutive games.

## Comparison to supervised learning

Neural networks learn to classify things correctly by attempting to minimize some measure of error.
Given an input $x$, the net produces a vector $y$. Assuming that we have 
a standard supervised learning problem such as [MNIST](https://www.tensorflow.org/get_started/mnist/beginners),
the component $y_i$ can be interpreted as the level of certainty that the input $x$ belongs to
category $i$ (assuming that we label the categories by $1,2,\dots,n$). In case $x$ is in category $j$,
the perfectly correct output $\tilde{y}$ would be 

<center>$$\tilde{y}_i = \begin{cases}1, \text{ if } i = j \\ 0, \text{ otherwise} \end{cases},$$</center>

i.e. $x$ is certainly in category $j$ and certainly not in category $i$ for $i \neq j$.
The classification error is measured by comparing $y$ to
the correct labeling $\tilde{y}$ via some loss function $H\left(y,\tilde{y} \right)$, often the cross entropy

<center>$$\begin{equation}\label{eq::cross_entropy}H\left(y,\tilde{y} \right) = - \sum \limits_{i} \tilde{y}_{i} \, \log y_{i}.\end{equation}$$</center>

The weights are then adjusted a little, so that the loss $H(y, \tilde{y})$ decreases slightly.
Now, if you feed the same $x$ to the network, the output $y$ will be slightly more similar to $\tilde{y}$.
You just run this for a long time and eventually you can, for example, tell if [there is a hot dog in the picture or not](https://itunes.apple.com/us/app/not-hotdog/id1212457521?mt=8).

The present situation isn't really that different. A video game can
be considered a sequence of tuples $(x_t, a_t)$, with $t$ the frame number, $x_t$ the game state
at frame $t$ and $a_t$ the action we took at frame $t$. In Cart Pole, the state $x_t$ consists of four numbers: cart position,
cart velocity, pole angle and pole velocity at tip. The action $a_t$ is either to push cart to the left or to the right ($a_t$ is $0$
or $1$ respectively). Given a state $x_t$ we could use a neural net to produce a vector $y_t \in \mathbb{R}^2$ and
then determine $a_t$ e.g. by checking which component of $y_t$ is greater. Given the optimal action
$\tilde{y}_t$ at state $x_t$, we could update the weights exactly as in a supervised learning.
We would, under some assumptions, eventually play the game near optimally.
The problem, of course, is that there is no direct access to the optimal output $\tilde{y}_t$.

As was explained in Karpathy's [post](http://karpathy.github.io/2016/05/31/rl/), the reinforcement learning way of dealing with
this situation is to evaluate our choices throughout the game in **hindsight**: We first finish a round of the game and only rate our actions afterwards. We then encourage further favorable actions and discourage choices that are rated poorly.

## Choosing the action

Before we focus on training the network, let's briefly describe how we are going to turn the output of our neural network $y_t$
into an action $a_t$. Instead of simply setting $a_t = 0$ or $a_t=1$ by checking which component of $y_t$ is greater, we are going
to define a [Markov decision process](https://en.wikipedia.org/wiki/Markov_decision_process). This is a fancy way of saying
that we'll always make a random decision so that $y_t$
represents the probabilities of choosing either action:

<center>$$y_{t,1} = \mathbb{P} \left[ a_t = 0 \right] \quad \text{and} \quad y_{t,2} = \mathbb{P} \left[ a_t = 1 \right].$$</center>

Allowing randomness has its advantages: The network will explore different strategies and in the long run
the successful strategies should become increasingly prevalent.

## Formulating the loss

We'll accept the following as sort of an axiom: The longer the pole remained balanced after the action $y_t$, the
better choice $y_t$ was. This kind of makes sense when you think about it, right?
Bad choices tend to  quickly screw things up, while good choices
help to keep the pole balanced in the long run. Hence moves that were
taken many frames before losing balance should be considered
favorable.

Justified by this heuristic, we assign $y_t$ an unstandardized reward $\rho_t$ via

<center>$$\rho_t = \sum \limits_{i=0}^{T-t} \gamma^i,$$</center>

where $T$ is the number of the last frame, $0 < \gamma < 1$ is our "discount rate" (a hyperparameter we set to $\gamma = 0.99$). We then
standardize the rewards $\rho_t$ via

<center>$$\begin{equation}\label{eq::reward}r_t = \frac{\rho_t - \mu_\rho}{\sigma_\rho}\end{equation},$$</center>

where $\mu_\rho$ is the mean of values $\rho_1, \rho_2, \dots, \rho_T$ and $\sigma_\rho$ is their standard deviation. Defined as above,
$\rho_t$ encourages actions that were taken long before the last frame $T$ in a way that the rewards will not blow up even if $T$
is very large (since a [geometric series](https://en.wikipedia.org/wiki/Geometric_series) converges). The standardization
$\eqref{eq::reward}$ makes sure that at least the last few choices are discouraged (usually about the last half is)
and also helps to control the reward size.

Define an auxiliary vector $\chi_{a_t}$ as $\chi_{a_t} = (1,0)$ if $a_t = 0$ and $\chi_{a_t} = (0,1)$ if $a_t=1$ and let

<center>$$\begin{equation}\label{eq::loss} L = \sum \limits_{t=1}^T r_t \, H(y_t, \chi_{a_t}),\end{equation}$$</center>

where $H$ is as in $\eqref{eq::cross_entropy}$ and $r_t$ is as in $\eqref{eq::reward}$.

This choice of $L$ can be based on intuition from a supervised classification problem:
In supervised learning, we can decrease $H$ in $\eqref{eq::cross_entropy}$ to encourage the network to change the output $y$
a bit towards $\tilde{y}$. If you drop the rewards $r_t$ from $\eqref{eq::loss}$ we would actually be (in a similar way)
encouraging our network to take whatever actions it chose to take during the game. The point in $\eqref{eq::loss}$ is that an
action $a_t$ for which $r_t$ is large is encouraged *more* than an action for which $r_t$ is small. If $r_t < 0$, the net is
*discouraged* to choose action $a_t$ at state $x_t$ in the future. It would now seem that the
poor choices should eventually become exceedingly improbable.

## Implementation

Using loss $\eqref{eq::loss}$, Cart Pole can be solved very similarly to
[MNIST](https://www.tensorflow.org/get_started/mnist/beginners) in TensorFlow.
Loss $\eqref{eq::loss}$ can be written in TF as
{% highlight python %}
loss    =   tf.tensordot( rewards,
                          tf.reduce_sum(-yhot * tf.log(y), reduction_indices = [1]),
                          axes=1)
training_step = tf.train.AdamOptimizer(1e-3).minimize(loss)
{% endhighlight %}
the actual backpropagation works in the convenient, automated, way
{% highlight python %}
sess.run(training_step, feed_dict={x:o_s, y_:a_s, rewards:normalized_rewards})
{% endhighlight %}
where the arrays `o_s` and `a_s` keep track of the states $x_t$ and actions $a_t$ throughout the game. The `normalized_rewards`
input corresponds to $r_t$'s defined in $\eqref{eq::reward}$. The Markov decision process can also be implemented in a few lines:

{% highlight python %}
prob_0 = sess.run(y, feed_dict={x:[observation]})[0][0]
if np.random.random() < prob_0:
    action = 0
else:
    action = 1
{% endhighlight %}

The rest of it is mostly just some bookkeeping.

Without really testing a lot of designs, I wrote a standard fully connected 3-layer network with ReLU activations which seems to
converge in a reasonable time. Here are the durations of 500 first rounds of a run:

{: style="text-align:center"}
![alt text]({{site.url}}/images/cart-pole-500.png  "500 rounds")

It works, but the results are not that great. On the [game page](https://gym.openai.com/envs/CartPole-v0) you can see that
it takes us around 10 times the number of rounds in which some people have reached the solution to get there. Also, the long term behavior
of our model is pretty bad. The kind of sporadic downward spikes you see near round 500 at the plot continue to appear even if the
experiment is run for a lot longer than 500 rounds. Maybe it's because we don't adapt our learning rate, or perhaps because
the game won't continue beyond 200 frames and that breaks our rewarding system in some way. It's pretty funny to watch the
model develop some bad habits like smashing left at full speed all the time as that kind of behavior occasionally gets reinforced
excessively.

While slightly unrefined, our model does contain a foundation to build better solutions on. Sharing improvements
in comments is encouraged!

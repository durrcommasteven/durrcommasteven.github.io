---
title: "A Dumb Battleship Algorithm"
last_modified_at: 2018-10-01T16:20:02-05:00
categories:
  - Blog
tags:
  - Projects
  - Probability
  - GANs
  - Machine Learning
  - Approximations

excerpt: "The dumber the better"
header:
  overlay_image: /assets/images/post_1/four_image_combination.png
  overlay_filter: 0.2 # same as adding an opacity of 0.5 to a black background
  caption: "Stable Diffusion's attempts at creating a 'Dutch Book'"
---

A while ago my friend Chad suggested that we make algorithms to play battleship on sets of random boards and see whose was better. This was all inspired by a [blogpost](https://datagenetics.com/blog/december32011/index.html) Chad had read from datagenetics, where Nick Berry applied battleship algorithms to random boards and saw which ones worked better than others.

I only had a few free hours, so I decided that I wanted to make an algorithm that was as good as possible, under the strict constraint that it was also as simple and dumb as possible. I wanted to avoid complicated math, difficult combinatorics, and any algorithm that would require walking down some huge decision tree I would have to manually populate.

As basic as the underlying idea was, the project raised a number of interesting ideas, and put the pros and cons of neural networks (NN's) in sharp relief.

As the datagenetics blogpost concludes, a really good battleship algorithm would be one which takes in the current state of the board -- the squares where it sees a ship, the squares where it sees ocean, and the squares it has not attacked yet -- and uses it to determine the probability that each unattacked square has a ship on it. It then goes through those unattacked squares, and chooses the most probable one to be its next target.

This is a pretty trivial statement. Essentially it is saying: "The best battleship algorithm is the one which has the most accurate understanding of the opponents board".

The hard part is generating a function from board state to probabilities. The first 'dumb' answer that comes to mind to solve this would be the Monte Carlo algorithm.

Under a Monte Carlo algorithm, I start with my partially explored board, where every unexplored square has a value of zero. I then place the ships that remain undiscovered in some random configuration such that they fit. I record where the ships were, adding 1/N to the value of each square they occupy, and remove the ships. If I do this whole process N>>1 times, I should get a pretty good probability function telling me how likely each square is to have a ship on it.

Immediately this is too complicated for me.

I would have to take a given board state (for example, the board below), and generate thousands of boards which matched my current knowledge. For me, that's too much.

Ideally, I would just have a function that takes in knowledge about a board and outputs a probability distribution. Why don't we use a neural network -- a universal function approximator -- to try to get something close to this.

Here is the idea: Train a neural network on partially explored battleship boards with the goal of having it output a probability density function for the output board.

# The Neural Network (NN)

Here the data comes in the form of images of the board's state, with each 'pixel' being either hit, miss, or unknown. These images contain features that are to a degree translationally invariant -- the same arrangement of ships can be shifted around the board. There are also certain small-scale features of the images that can tell us about the underlying arrangement of ships. The feature of a 'hit' with unexplored squares all around tells us that a ship is either to left of it, right of it, above it, or below it.

This type of input data is perfect for particular type of NN known as convolutional (also called a convnet). These NN's apply convolutions to an image using many different kernels. The kernels are varied during training and, once optimized, tend to pick out important features of the input image.

To keep things simple, I used the neural network architecture shown below:

We have one convolutional layer (no max pooling or anything -- I want this as simple as possible) and one fully connected layer (both with Relu activation). We get our probability distribution output by reshaping from a length 100 list to a 10x10 array.

Next we need to train the NN. We want the input data to be the types of board states the NN will likely see during gameplay -- partially revealed boards with hits and misses according to the history of the game. The labels for this data would then be the true states of the boards. During training, we teach the NN to use the board states with the correct underlying board.

Then as the NN plays battleship, it will see the types of boards it was trained on and will be able to use these to predict where the ships are and are not.

There is a slight problem here: the boards the NN will see during gameplay are precisely the boards an optimized NN algorithm will naturally lead itself to. How can we pick the right board states to train the NN, if the boards we want are precisely those that we need a trained NN to obtain? (what a confusing sentence)

# The First Born Approximation

This same type of issue comes up in physics often. In particular, we see this in the problem of a plane wave hitting a potential and scattering off. If we go through the math of this problem we end up with the following formula:

Our goal is to solve for $latex \psi(r)$, the scattered wave.

To find $latex \psi(r)$, we must evaluate the right hand side. But to evaluate the right hand side, we already need to know $latex \psi(r)$. The way physicists get around this is to assume that $latex \psi(r)$ will be something like the plane wave, $latex \psi_{0}(r)$. So we replace $latex \psi(r')$ on the right hand side with $latex \psi_{0}(r')$. We can then evaluate this and obtain what is known as the First Born Approximation, written $latex \psi_{1}(r)$.

We say 'First' because in principle I can do this as many times as I want. Replacing $latex \psi$ in the right hand side of our equation with $latex \psi_{1}$ and evaluating gives us the second Born approximation, $latex \psi_{2}$, and so on.

The idea is that if our initial guess of a plane wave is sufficiently close to the right answer, repeated application of this algorithm will make $latex \psi_{n}$ converge correctly to $latex \psi$.

Lets get back to the problem of getting data to train the NN model. If we wrote it in the form of an equation, we would have something like:

That is, the trained model comes from taking an untrained model, and applying the training algorithm (which itself requires the trained model to generate training data).

We can approach this from exactly the same Born mentality as before: assume that the strategy of the trained model will be in some way a variation of the strategy of the untrained model (random guessing). We can then write what I will call the First Born Approximation of the NN model.

Now, again, if I really wanted to I could keep going and instead get the second, third, or fourth Born Approximations of the NN. The reasons not to do this are, however, extremely compelling:

I don't care nearly enough to do it.
The whole point of this was to make it simple.
I reeeeaaally don't want to do it.

Great! Now I have a neural network that has been trained to take as an input some partially explored board, and give as an output an entire board with each square getting a number between 0 and 1. The greater the number, the more likely the square is to be occupied.

# Are Outputs Probabilities?

This brings up a subtle question. How do I know I can interpret these numbers as probabilities?  At this point all I know is that if my neural network is perfect, then the more likely a square is to be occupied, the closer the NN's output for that square is to 1. But does this mean the outputs are necessarily probabilities? Not at all.

Really, all I can know is that my NN produces some monotonically increasing function of the probability.

There are two responses to this concern:

1)

I don't care. All I care about is choosing the most probable unexplored square. So as long as the output of the neural network is a strictly increasing function $latex f$ of the actual probability (i.e. one-to-one as well as monotonically increasing), our choice of most probable next square is unchanged.

max($latex \{ p(x_{i, j}) | x_{i, j} \in unexplored \}$) = max($latex \{ f(p(x_{i, j})) | x_{i, j} \in unexplored \}$)

And our algorithm stays the same.

2)

If you're really invested in this question, there is a way to make sure that the neural network's output corresponds to its internal probabilities and not just some monotonic function of them.

The output of a neural network depends on the loss function that it is trained with. Essentially, this is the function used to determine the difference between the correct probability distribution and the one the NN produced. It tells the algorithm how 'incorrect' the NN is, and the algorithm tries to minimize it.

So which loss function forces the NN to tell us the probabilities themselves? [The derivation I give is mostly lifted from a Terry Tao [blogpost](https://terrytao.wordpress.com/2016/06/01/how-to-assign-partial-credit-on-an-exam-of-true-false-questions/) about a related topic].

Let $latex p(x_{i, j})$ be the probability that the square at (i, j) is occupied, and let $latex q(x_{i, j})$ be the output that the NN actually gives for that square.

Then the loss function we want would be the function, $latex L$, such that the expected loss

$latex p(x_{i, j}) L(1, q(x_{i, j}))+(1-p(x_{i, j})) L(0, q(x_{i, j}))$

is minimized when $latex q(x_{i, j})=p(x_{i, j})$.

This can be read as "(the probability of being correct) times (the loss if correct) plus (the probability of being incorrect) times (the loss if incorrect)"

To simplify things, we can see that $latex L(0, q(x_{i, j})) = L(1, 1-q(x_{i, j}))$. In words: the punishment for having the output be 0, when you predicted it would be 1 with probability $latex q(x_{i, j})$ should be the same as the punishment for having the output be 1 when you said it would be zero with probability $latex 1 - q(x_{i, j})$

Using this, we can make things more concise and say that $latex L$ must minimize

$latex p(x_{i, j}) L(q(x_{i, j}))+(1-p(x_{i, j})) L(1 - q(x_{i, j}))$

When $latex q(x_{i, j})=p(x_{i, j})$.

We ensure this by setting the derivative with respect to $latex p(x_{i, j})$ to zero at $latex q(x_{i, j})=p(x_{i, j})$. That is:

$latex 0 = p(x_{i, j}) L'(p(x_{i, j})) - (1-p(x_{i, j})) L'(1 - p(x_{i, j}))$

This is solved by $latex L( p ) = - Log( p )$, which is equal to a loss function known as cross entropy. Cross entropy, it so happens, is exactly the function my training algorithm uses to compare the NN output to the actual board.

If we let ocean squares = 0, and squares with ships = 1, then the full boards we train our NN on are already probability distributions, $latex p(x_{i, j})$. Using the cross entropy loss function during this training will then force our outputs to be probabilities! (again, not that we care)

Once training is done, we have our algorithm!

Now we just have to do some analysis to see how good (or shitty) it is.

# Watching the Game

First, just for fun, lets see how a typical board is played.

To the left we have the actual gameplay, with purple representing an unknown square, white a miss, and black a hit.  To the upper right, we have the NN's internal probability distribution (darker = more likely to be a ship), and to the lower right we have the actual board.

There's a lot of remarkable stuff going on in this game, and much of it is surprisingly intuitive. When we watch the algorithm play, it seems almost human in how it appears to be gaining an internal understanding of the board.

We see that when the NN gets a hit or a miss, the probability distribution suddenly gets a vertical and horizontal streak centered at that point.

To the left we have the actual gameplay, with purple representing an unknown square, white a miss, and black a hit.  To the upper right, we have the NN's internal probability distribution (darker = more likely to be a ship), and to the lower right we have the actual board.

There's a lot of remarkable stuff going on in this game, and much of it is surprisingly intuitive. When we watch the algorithm play, it seems almost human in how it appears to be gaining an internal understanding of the board.

We see that when the NN gets a hit or a miss, the probability distribution suddenly gets a vertical and horizontal streak centered at that point.

If its a hit, then the NN tends to follow these dark probability streaks until it understands that either it's searching in the wrong direction, or its found all of the ship.

The NN also understands the directionality of the ships it sees. It always seems to make sure that both ends of a line of hits are unoccupied before moving on, but it doesn't really care about the immediate 'left' and 'right' of these hits, as we see when the NN attacks the vertical ship in the above game. Clearly it understands the general shapes of the ships on the board.

# Data Analysis

Let's move on to an analysis that's more quantitative: the distribution of the lengths of a different games. I let my algorithm run for a while, and after around 9000 games I checked the distribution of how long they tended to take.

The median here (i.e. the number of moves needed to win at least half the games) was 52. But how does this compare?

This quantity is something that Nick Berry analyzed in his blog for four different algorithms: one that uses the exact probability, two custom algorithms of his own design, and one that chooses random squares until it wins.

Below we show the fraction of games won as a function of the number of moves made. The dotted red line is my algorithm's median, and the other vertical lines are the medians of Nick Berry's algorithms.

My algorithm was vastly better than random and also beat out both of his custom algorithms. Unfortunately, it was about 10 moves behind his probabilistic algorithm, but for not really trying too hard on perfecting this, I don't feel too bad about that.

Another interesting analysis shows that my algorithm even 'knows' how many ships it has left to find. I applied my algorithm to around 500 partially revealed random boards (the same type used as training data). Then I summed up the probability that the algorithm gave for each unexplored square to have a ship on it, and compared this number with the actual number of squares with ships left to be found. The two sets of numbers line up almost exactly.

We can also easily see that there are some serious problems with this algorithm. For instance, if we look at the game length distribution, we see that there was at least one game which took around 98 moves.

This really makes no sense, since in theory one could simply attack squares in a checkerboard pattern and then fill squares in to finish off the ships. Even this very inefficient algorithm would complete a game in much less than 98 moves.

To find out what's going on here, we can look at an especially long game and see what mistakes it makes.

This game took 96 moves, and around two thirds into it, the algorithm start to fail pretty spectacularly. The NN clearly doesn't know to assign exactly zero probability to single unknown squares surrounded by misses. In fact, because of this flaw, the algorithm spends over half the game just looking for the length 2 ship.

Lets see how the algorithm plays during one of its faster games, this one taking just 25 moves.

Clearly things are easier in a game of battleship when ships are touching each other and in the center of the board.

# Potential Improvements

**(skip if you don't care)**

I can immediately see a few ways it would be possible to improve this algorithm.

1)

I could go to a higher 'Born approximation' of the NN. This would probably improve performance, but it wouldn't be very fun to do.

2)

If the NN saw a hit in the input board at position (i, j), it had to learn to propagate that information through all the convolutions, through the fully connected layer, all the way to the output at exactly position (i, j). This took up some of the learning capacity of the NN, or at least slowed down training.

A good network architecture should allow efficient representation of data. That could mean fewer layers, or fewer neurons, or an entirely different approach to the project.

A better architecture here would probably include some kind of direct linear connection between the input and output layers so that information about hits and misses could instead flow immediately through these.

3)

Finally, I could also include some hard logic. I could add a filter that would apply the results of a few hard coded logical deductions about the board. But doing this would entirely defeat the entire purpose of my dumb battleship algorithm. I'd have to think hard and come up with 'theorems' about battleship boards -- exactly what I was trying to avoid.




# Final Thoughts

The weaknesses of this algorithm stem from the same quality that provides so much strength -- its use of fast heuristics over some protracted application of logic.

Humans aren't natural logicians. Instead, we use sets of heuristics to analyze the different patterns we come across. This is exactly what neural networks do as well (remember, our brains are neural networks). Neural networks are effective because they're able to find good internal representations of the various patterns observed in the data, and (in supervised learning) learn to correlate these with the training labels.

My NN could have learned to give zero probability to the unexplored squares surrounded by misses. The fact that it doesn't do this is likely a result of the Born approximation we applied, resulting in a lack of this pattern in the training data.

But that the NN was so smart in some ways, but so dumb in this way, really exemplifies the fact that it is not using reason, and it is not able to extrapolate using logic. Humans need intense training to be able to algorithmically apply logic using our heuristic brains, and neural networks are largely the same.

This neural network is simply pattern matching over the landscape of data it has been trained on, and it is unable to stretch its 'understanding' beyond that. [Insert allusion to the human experience here]








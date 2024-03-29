---
title: "Transformer Quantum States I"
last_modified_at: 2023-01-01T16:20:02-05:00
categories:
  - Blog
tags:
  - Projects
  - Transformers
  - Quantum Mechanics
  - Machine Learning

excerpt: "Why use a language model for physics?"
header:
  overlay_image: /assets/images/post_3/black_background.png
  overlay_filter: 0.3 # same as adding an opacity of 0.5 to a black background

---
# Transformer Wavefunctions: Part 1

A while back I was using transformer models to learn the ground states of Hamiltonians (along with my friend Ven). 
Let's outline the way these neural network wavefunctions might be useful.

## Neural Network Wavefunctions

Wavefunctions are defined over basis states, $\Psi(\sigma)$, where each $\sigma$ is some basis element. 
for a particle in a box, $\sigma$ would be spatial position, $x$, and $\Psi(x)$ maps positions to complex numbers. 
For a 1d spin model, $\sigma$ would be a given spin configuration (like $[0,1,1,1,0,1]$). 
Here, $\Psi(\sigma)$ effectively maps sequences of 1's and 0's to complex numbers. 

If you wanted to describe the ground state of the particle in a box model, you might just take $\Psi(x)$ to be some physically plausible candidate, or some sum of candidates, and use the variational principle to minimize the energy. 
Just use some numerical integration and differentiation to find $\theta$ which minimizes $\langle \Psi_\theta | H | \Psi_\theta\rangle$, and we're left with a pretty good approximate ground state.

Neural networks have been useful in the context of lattice models. Why is this?

Suppose I have some lattice Hamiltonian, $H$, acting on $N$ spins, and I'd like to approximate its ground state. 
Starting with the same approach, suppose we define some parameterized model $\Psi_\theta(\sigma)$ which seems plausible (or has enough capacity to model whatever the answer might be). We simply define a 

Like before, we now want to find the parameters $\theta$ which minimize 
$$\langle \Psi_\theta | H | \Psi_\theta\rangle$$.

We can simplify this a bit by writing it as follows

$$
\langle \Psi_\theta | H | \Psi_\theta\rangle = \sum_{\sigma, \sigma'} \Psi_\theta(\sigma')^* H_{\sigma', \sigma} \Psi_\theta(\sigma) = \sum_{\sigma, \sigma'} |\Psi_\theta (\sigma')|^2 H_{\sigma', \sigma} \frac{\Psi_\theta(\sigma)}{\Psi_\theta(\sigma')}
$$

Define $E_{loc}(\sigma')$ as 

\begin{equation}
    E_{loc}(\sigma') = \sum_{\sigma} H_{\sigma', \sigma} \frac{\Psi_\theta(\sigma)}{\Psi_\theta(\sigma')}
\end{equation}

For sufficiently restricted Hamiltonians, (which covers almost none of them, but almost all of the ones we care about) $H_{\sigma', \sigma}$ will nearly always vanish, and so $E_{loc}(\sigma')$ becomes tractable to compute. 

We are then left with

\begin{equation}
    \langle \Psi | H | \Psi \rangle = \sum_{\sigma'} |\Psi_\theta(\sigma')|^2 E_{loc}(\sigma')
\end{equation}

Great! no problem, just compute this and use gradient descent to minimize with respect to $\theta$.

For 100 spins, you can simply plug this into your laptop and after just a couple thousand lifetimes of the universe, you'll have one iteration done. 

But grad school is just six years, and kids don't have long attention spans anymore, so maybe you want something faster.
If this is the case, you'll have to approximate the gradient. 

Let's try sampling uniformly over $n$ states, $\{ \sigma_i \}$ to get an energy estimate $\tilde{E}_n$. This seems nice, because all we need is an encoder -- some function mapping sequences to numbers. Feed it some randomly selected sequences, and minimize the energy of the outputs.

An unbiased estimator using $n$ uniformly selected samples is given by

$$
\tilde{E}_n = \frac{2^L}{n} \sum_{i=1}^n |\Psi_\theta(\sigma_i)|^2 E_{loc}(\sigma_i)
$$
 
where $L$ is the number of spins in the chain. This is clear, since 

$$
\langle \tilde{E}_n \rangle = \frac{2^L}{n} \sum_{i=1}^n \langle |\Psi_\theta(\sigma_i)|^2 E_{loc}(\sigma_i) \rangle
$$

which is (averaging over all $2^L$ unique spin combinations, $\sigma'$)

$$
\frac{2^L}{n} \sum_{i=1}^n \left(\frac{1}{2^L}\sum_{\sigma'} |\Psi_\theta(\sigma')|^2 E_{loc}(\sigma') \right) = 
\sum_{\sigma'} |\Psi_\theta(\sigma')|^2 E_{loc}(\sigma') 
$$

and equals 

$$
\langle \Psi | H | \Psi \rangle
$$

That was easy! Let's just see how big $n$ has to be. 
We'll need our gradients to be pretty accurate, so let's look at the variance of the estimate.

Well, the variance of a sample of $n$ elements is simply 

$$
\frac{2^{2L}}{n} Var[|\Psi_\theta(\sigma)|^2 E_{loc}(\sigma)]
$$

Already that exponential term to the left sucks -- how can we fix this? Well, the problem is clearly in how we sample the states. From the variance expression, we see that in order to control the uncertainty in our energy estimates, we'll need the number of samples to be exponential in the length of the chain. 

Again, we'd be stuck waiting for the lifetime of the universe for our estimate of the energy to come anywhere near its actual value. So what can we do about this exponential term?

Let's kill it off using importance sampling. 

## Importance Sampling

What we really care about is finding 
$$
\langle E_{loc}(\sigma)\rangle_\sigma
$$, where 
$$\sigma \sim |\Psi_\theta(\sigma)|^2$$ as the born rule tells us. If we use $q(\sigma)$ to describe the probability distribution we use to sample the states, then we can write this as

$$
\langle E_{loc}(\sigma)\rangle_\sigma = \sum_{\sigma} q(\sigma) \left( q(\sigma)^{-1} |\Psi_\theta(\sigma)|^2 E_{loc}(\sigma)\right)
$$

So far, we've taken $q(x)$ to be the uniform distribution over states. 

What should we set $q(x)$ to in order to minimize the variance? Well we can actually compute this using Lagrange multipliers: 

$$
\mathcal{L} = \sum_{\sigma} q(\sigma)^{-1} |\Psi_\theta(\sigma)|^4 E_{loc}(\sigma)^2 - \langle E_{loc}(\sigma)\rangle_\sigma^2 - \lambda (1 - \sum_{\sigma} q(\sigma))
$$

Above, in the lagrangian, we have the expression for the variance of our estimate when we sample from $q(\sigma)$, followed by an expression forcing $q(\sigma)$ to be normalized.

To find the distribution which minimizes the variance of an energy estimate, set the functional derivative w.r.t. to $q(\sigma)$ to zero, and find

$$
0 = -q(\sigma)^{-2} | \Psi_\theta(\sigma)|^4 E_{loc}(\sigma)^2 + \lambda 
$$

Which tells us that the variance is minimized for 

$$
q(\sigma) \propto |\Psi_\theta(\sigma)|^2 E_{loc}(\sigma)
$$

We don't have access to this probability distribution exactly, but if we change our setup, we can have access to 
$$
|\Psi_\theta(\sigma)|^2
$$. 
We just need to go from an encoder network (which maps states to numbers) to a decoder network. 

## Autoregressively Decoding Quantum States

So far I've been considering functions which map states to amplitudes -- (complex numbers). But that doesn't help us with the sampling part, and without that, we're back to waiting until the sun explodes.

This is where language models become relevant -- encoders (like BERT) can embed sentences into vector spaces, but we would also like to generate text. The way we typically do this is by using autoregressive sampling, which sounds fancy, but really it's just an application of the chain rule of probability:
$$
p(\vec{\sigma}) = p(\sigma_1) p(\sigma_2 | \sigma_1) p(\sigma_3 | \sigma_2, \sigma_1) \cdot \cdot \cdot p(\sigma_N | \sigma_{N-1}, ..., \sigma_2, \sigma_1)
$$

This is what language models do.

We'll use a transformer to do this. It'll have two outputs: the log probability, and the phase contribution for each term in the sequence. 
This way, we can simulate sampling from the wavefunction, and evaluate the phase and amplitude of any spin sequence. 

In a future post, we'll go over why we would use a transformer for this, and show the specific benefits. 
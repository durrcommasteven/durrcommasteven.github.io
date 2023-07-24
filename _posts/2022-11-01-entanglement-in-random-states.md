---
title: "Dutch books"
last_modified_at: 2022-10-01T16:20:02-05:00
categories:
  - Blog
tags:
  - Projects
  - Probability
  - GANs
  - Machine Learning

excerpt: "(My favorite kind of book)"
header:
  overlay_image: /assets/images/post_1/four_image_combination.png
  overlay_filter: 0.2 # same as adding an opacity of 0.5 to a black background
  caption: "Stable Diffusion's attempts at creating a 'Dutch Book'"

gallery:
  - url: /assets/images/post_1/stab_diff_dutch_book_2.jpg
    image_path: /assets/images/post_1/stab_diff_dutch_book_2.jpg
    alt: "stab diff image 1"
    title: ""
  - url: /assets/images/post_1/stab_diff_dutch_book_3.jpg
    image_path: /assets/images/post_1/stab_diff_dutch_book_3.jpg
    alt: "stab diff image 2"
    title: ""
  - url: /assets/images/post_1/stab_diff_dutch_book_7.jpg
    image_path: /assets/images/post_1/stab_diff_dutch_book_7.jpg
    alt: "stab diff image 3"
    title: ""
---

\section{Entanglement within Random States}

A while back I was using transformer models to learn the ground states of Hamiltonians (along with my friend Ven). 
The way these neural network wavefunctions work is as follows:

Wavefunctions are defined over basis states, $\Psi(\sigma)$, where each $\sigma$ is some basis element. 
for a particle in a box, $\sigma$ would be spatial position, $x$, and $\Psi(x)$ maps positions to complex numbers. 
For a 1d spin model, $\sigma$ would be a given spin configuration (imagine a spin is either $1$ or $0$, and a configuration could be something like $\{0,1,1,1,0,1\}$). 
Here, $\Psi(\sigma)$ effectively maps sequences of 1's and 0's to complex numbers. 

If you wanted to describe the ground state of the particle in a box model, you might just take $\Psi(x)$ to be some physically plausible candidate, or some sum of candidates, and use the variational principle to minimize the energy. 
-----
The variational principle tells us that by minimizing the energy of a state, we get the ground eigenstate. 
This sounds trivial, but it's incredibly useful. 

Let's see how this is derived below:

Writing $| \psi \rangle = \sum_i \psi_i |i\rangle$, we have

$$
\langle \psi | H | \psi \rangle = \sum_{i, j} \psi^*_i  \psi_j \langle i | H | j \rangle
$$
Now which normalized vector minimizes this? Let's find out
$$
\mathcal{L} = \lambda(1-\sum_i \psi_i^* \psi_i) + \sum_{i, j} \psi^*_i  \psi_j \langle i | H | j \rangle
$$
Setting the functional derivative to zero tells us
$$
\frac{d}{d\psi^*_i}\mathcal{L} = -\lambda \psi_i + \sum_{j}  \psi_j \langle i | H | j \rangle =0
$$
Okay so what is this saying?
$$
\sum_{j}  \psi_j \langle i | H | j \rangle = \lambda \psi_i
$$
In other words, $H |\psi \rangle = \lambda |\psi \rangle$, which is the eigenvalue equation. 

Therefore minimizing the expectation of the energy should give us the ground state. 

An interesting side-effect of this derivation is that _every_ eigenstate is an extreme of the energy. 

-----
Just use some numerical integration and differentiation to find $\theta$ which minimizies $\langle \Psi_\theta | H | \Psi_\theta\rangle$, and we're left with a pretty good approximate ground state.

Fancy neural networks tend to come into play when we're discussing lattice models. Why is this?

Suppose I have some lattice hamiltonian, $H$, acting on $N$ spins, and I'd like to approximate its ground state. 
Starting with the same approach, suppose we define some parametrized model $\Psi_\theta(\sigma)$ that seems plausible (or has enough capacity to model whatever the answer might be).
Now, like before, we want to find $\argmin_\theta \langle \Psi_\theta | H | \Psi_\theta\rangle$. 

We can simplify this a bit by writing it

$$
\langle \Psi_\theta | H | \Psi_\theta\rangle = \sum_{\sigma, \sigma'} \Psi_\theta(\sigma')^* H_{\sigma', \sigma} \Psi_\theta(\sigma) = \sum_{\sigma, \sigma'} |\Psi_\theta (\sigma')|^2 H_{\sigma', \sigma} \frac{\Psi_\theta(\sigma)}{\Psi_\theta(\sigma')}
$$

Define $E_{loc}(\sigma')$ as 
\begin{equation}
    E_{loc}(\sigma') \equiv \sum_{\sigma} H_{\sigma', \sigma} \frac{\Psi_\theta(\sigma)}{\Psi_\theta(\sigma')}
\end{equation}
For sufficiently restricted Hamiltonians, (which covers almost none of them, but almost all of the ones we care about) H_{\sigma', \sigma} will nearly always vanish, and so this is tractable to compute. 

We are then left with

\begin{equation}
    \langle \Psi | H | \Psi \rangle = \sum_{\sigma'} |\Psi_\theta(\sigma')|^2 E_{loc}(\sigma')
\end{equation}

Great! no problem, just compute this and use gradient descent to minimize with respect to $\theta$.

For 100 spins, you can simply plug this into your laptop and after just a couple thousand lifetimes of the universe, you'll have one iteration done. 

But grad school is just six years, and kids dont have long attention spans anymore, so maybe you want something faster.
If this is the case, you'll have to approximate the gradient. 

Let's try sampling uniformly over $n$ states, $\{ \sigma_i \}$ to get an energy estimate $\tilde{E}_n$. 
$$
\tilde{E}_n = \frac{1}{n \sum_j |\Psi_\theta(\sigma_j)|^2} \sum_{i=1}^n |\Psi_\theta(\sigma_i)|^2 E_{loc}(\sigma_i)
$$

That was easy! Let's just see how big $n$ has to be. 
We need our gradients to be pretty accurate, so what's variance of the energy estimate?
Well, this depends on $|\Psi_\theta(\sigma_i)|^2$. 
Typical ground states tend to be sparse -- let's suppose $|\Psi_\theta(\sigma_i)|^2 E_{loc}(\sigma_j)$ is distributed according to 

$$
P(|\Psi_\theta(\sigma_i)|^2 E_{loc}(\sigma_j) = x) = \exp(-x \lambda) Z
$$

So this option also sucks. What do we do?
The obvious example 

$$
\langle\tilde{E}_n^2 - E^2\rangle
\frac{1}{n^2} \sum_{i, j=1}^n |\Psi_\theta(\sigma_i)|^2 |\Psi_\theta(\sigma_j)|^2 E_{loc}(\sigma_i) E_{loc}(\sigma_j)
$$

E' = (1 / n) 2^N \sum_{i \in s'} p_i E_i

lets say 2^N is huge (as it normally is)
then E' =


var(E') = (1/n) var(2^N p_i E_i)

std(E') = \sqrt(2^N/n) std(p_i E_i)


var(p_i E_i) = 2^{-N} \sum_i (p_i E_i)^2 - 2^{-2 N} (\sum_i p_i E_i)^2

var(p_i E_i) = 2^{-N} \sum_i (p_i E_i)^2 - 2^{-2 N} (E)^2

var(p_i E_i) = 2^{-N} (\sum_i (p_i E_i)^2 - 2^{-N} (E)^2)


std(E') = \sqrt(1/n) sqrt(\sum_i (p_i E_i)^2 - 2^{-N} (E)^2)


How can we reduce this variance?

What if we sample (p_i E_i)

E' = <E_i>_p_i

var(E') = (1/n) \sum_{i ~ p_i}

sample according to 







For each basis state $\sigma$ untrained neural networks return logits, $l_\sigma$. 
These are passed to a softmax to obtain a probability for each state:
$$
p(\bm{\sigma}) = \frac{\exp(l_{\bm{\sigma}})}{\sum_{\bm{\sigma}'} \exp(l_{\bm{\sigma}' })} = \frac{\exp(l_{\bm{\sigma}})}{Z}
$$
We can then obtain a real wavefunction by taking the square root. 
$$
\psi(\bm{\sigma}) =\frac{\exp(l_{\bm{\sigma}}/2)}{\sqrt{Z}}
$$
To understand the entanglement within randomly initialized neural network states, we begin by assuming random logits, beginning with the assumption that our logits are initially Gaussian, with mean $\mu$ and standard deviaion $\sigma$.
 This could arise from the central limit theorem (\textbf{is this the case at initialization for us?}). We immediately see that $\mu$ is of no import, as the partition function within softmax removes it. This leaves $\sigma$ as the parameter of interest.  

We can immediately gain some traction by examining limiting cases. $\sigma \rightarrow 0$ would correspond to equal probability for all basis-states.
 Note this wavefunction is simply 
$$
|\psi\rangle = \mathcal{N} \bigotimes_i \left( \sum_{s_i} |s_i\rangle  \right)
$$
Where above $i$ indexes sites, while $s_i$ enumerates each possible local state. $\mathcal{N}$ is an overall normalization. By using a suitable local basis change, this can be written as a product state, and therefore has no entanglement.

In the opposite limit of $\sigma \rightarrow \infty$, one basis-state dominates the others, and we have
$$
\psi(\bm{\sigma}) =\delta(\bm{\sigma}, \argmax_{\bm{\sigma}'} l_{\bm{\sigma}'}) 
$$
And again we have no entanglement, a our wavefunction is again a product state (assuming a product state basis). 

Between these two extremes, we expect some growth and decay of entanglement. 

We can compute the n-Renyi entropy using the replica trick, evaluating the expectation value of twist operators on an n-fold copy of the original state. For simplicity, we focus on the 2-Renyi entropy below, where the twist operator takes the form of a swap.

by doubling the Hilbert space and computing the expectation of the swap operator:
$$
Tr[\rho_A^2] = \langle \psi | \langle \psi | \text{Swap}_A |\psi \rangle | \psi \rangle
$$
Where $\text{Swap}_A$ acts as follows:
$$
\text{Swap}_A |\bm{\sigma}_A \bm{\sigma}_B \rangle | \bm{\sigma}'_{A} \bm{\sigma}'_{B} \rangle = |\bm{\sigma}'_A \bm{\sigma}_B \rangle | \bm{\sigma}_{A} \bm{\sigma}'_{B} \rangle
$$
 Writing this outin terms of our wavefunction,
\begin{multline}
\langle \text{Swap}_A \rangle = \sum_{\bm{\sigma}_{A_i}, \ \bm{\sigma}_{B_i}} \langle \sigma_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_1}} | \langle \sigma_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_2}} | \\
\frac{e^{(l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_2}})/2}}{Z} \cdot \frac{e^{(l_{\bm{\sigma}_{A_3}, \bm{\sigma}_{B_3}} + l_{\bm{\sigma}_{A_4}, \bm{\sigma}_{B_4}})/2}}{Z} \\
 | \sigma_{\bm{\sigma}_{A_4}, \bm{\sigma}_{B_3}} \rangle | \sigma_{\bm{\sigma}_{A_3}, \bm{\sigma}_{B_4}} \rangle
\end{multline}
Which yields
$$
\langle \text{Swap}_A \rangle = \sum_{\bm{\sigma}_{A_i}, \ \bm{\sigma}_{B_i}} 
\frac{e^{(l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_2}})/2}}{Z} \cdot \frac{e^{(l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_2}})/2}}{Z}
$$

Now we can attempt to approximate this expression. Take the regions of interest, $A$ and $B$, to be of equal size. Each element, $\exp(l_{\bm{\sigma}})$, can be seen as a sample from a log-normal distribution with parameters $\mu$ and $\sigma$. Taking $2^N$ to be large,
$$
\frac{Z}{2^N} = \frac{1}{2^N}\sum_{\bm{\sigma}'} \exp(l_{\bm{\sigma}' }) \approx \exp(\mu + \sigma^2 / 2)
$$
Where $\exp(\mu + \sigma^2 / 2)$ is the known mean of such a log-normal distribution.

We can now approximate the numerator:
$$
e^{(l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_2}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_2}})/2}
$$
Taking each of the logits as being i.i.d. Gaussian r.v.s with mean $\mu$ and variance $\sigma^2$, we note that
$$
(l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_2}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_2}})/2 \sim \mathcal{N}(2 \mu, \sigma^2)
$$
Therefore 
$$
2^{-2N} \sum_{\bm{\sigma}_{A_i}, \ \bm{\sigma}_{B_i}} e^{(l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_2}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_2}})/2} \approx \exp(2\mu + \sigma^2 / 2)
$$

Combining these, we see that 
$$
\langle \text{Swap}_A \rangle = \frac{\sum_{\bm{\sigma}_{A_i}, \ \bm{\sigma}_{B_i}} e^{(l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_2}} + l_{\bm{\sigma}_{A_2}, \bm{\sigma}_{B_1}} + l_{\bm{\sigma}_{A_1}, \bm{\sigma}_{B_2}})/2}}{Z^2}\approx \frac{\exp(2\mu + \sigma^2 / 2)}{\exp(2\mu + \sigma^2)}
$$
And so 
$$
\langle \text{Swap}_A \rangle \approx \exp(-\sigma^2 /2)
$$
We can perform this type of analysis for each n-Renyi entropy, finding for each
$$
\langle \text{n-Twist}_A \rangle \approx \frac{\exp(2\mu + n \sigma^2 / 4)}{\exp(2\mu + n \sigma^2 / 2)} = \exp(-n \sigma^2 / 4)
$$

The applicability of this approximation breaks down with as the variance of the relevant log-nomral distributions grow. Within log-normal distributions, the variance scales as $(\exp(\sigma^2)-1)\exp(2 \mu + \sigma^2)$. Therefore we expect breakdown as $\sigma$ grows.

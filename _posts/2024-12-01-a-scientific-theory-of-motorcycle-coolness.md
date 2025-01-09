---
title: "When Does Riding a Motorcycle Look Cool?"
last_modified_at: 2024-12-01T16:20:02-05:00
categories:
  - Blog
tags:
  - Thoughts

excerpt: "A quantitative model of motorcycle coolness"
header:
  overlay_image: /assets/images/post_6/cool_pic.png
  overlay_filter: 0.25 # same as adding an opacity of 0.5 to a black background
---

<figure>
<img src="{{site.baseurl}}/assets/images/post_6/moped_vid_1.webp" alt="moped_vid" style="width:100%">
</figure>

Something I noticed a while back was that, although in general, people on motorized two-wheeled vehicles (hereafter referred to as MOTORCYCLES) look cool, there are cases when they do not.

For example, both of these riders look cool: 

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/two_cool_riders.png" alt="two-cool-riders" style="width:40%">
<figcaption><b>Two self-evidently cool-looking motorcycle riders.</b></figcaption>
</figure>

This rider, however, definitely does not look cool.

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/moped.jpeg" alt="uncool-rider" style="width:40%">
<figcaption><b>A very clearly uncool-looking motorcycle rider.</b></figcaption>
</figure>

To explain this phenomenon, I have formulated a theory: coolness on a motorcycle is a function of seating angle of the back with respect to the vertical, as shown below:

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/cool_plot.png" alt="coolness-plot" style="width:80%">
<figcaption><b>Seating angle vs coolness on a motorcycle. Leaning backward is considered negative, and leaning forward is considered positive.</b></figcaption>
</figure>

Around $\theta=0$ this obeys the equation

$$
\text{Coolness}(\theta) = C_{mop.} + (1-e^{-\left(\frac{\theta}{\pi/10}\right)^2})(C_{sup.} - C_{mop.})
$$

Where above, $C_{mop.}$ is the coolness of a moped (coolness minimum), and $C_{sup.}$ is roughly the coolness of the superbike (coolness supremum). 

To be specific, let's take a look at the seating angle of the two cool riders. (Leaning backward is considered a negative angle, and leaning forward is considered positive.)

On the *superbike*, the rider has an angle of roughly $\pi/3$.
<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/superbike_with_angle.png" alt="superbike" style="width:40%">
</figure>

On the *harley*, the rider has an angle of around $-\pi/6$.
<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/harley_with_angle.png" alt="harley" style="width:40%">
</figure>

But on the *moped* the rider has an angle of $0$. This represents the minimum of the coolness vs angle curve. 
<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/moped_with_angle.png" alt="moped" style="width:40%">
</figure>

This makes sense, and clearly we've proven this theory is true beyond a doubt — but it also raises some questions (helpfully indicated by the question marks below).

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/cool_plot_with_questions.png" alt="coolness-plot-question" style="width:80%">
<figcaption><b>Seating angle vs. coolness on a motorcycle -- pushing the limits of our laws of coolness.</b></figcaption>
</figure>

## Exotic Motorcycle Riding Configurations

What does my seat-angle theory of motorcycle coolness say about these regimes?

How do we even begin to investigate the coolness of $\theta=\pm \pi/2$. 
First, we need to ask, how does one ride a motorcycle when sitting at $\theta=\pm\pi/2$?

At these extreme points in seating-angle-space, the orientation of one's legs might have two distinct positions which are equally good options to the rider -- on the footpegs, or off the bike entirely. 
The orientation that the biker eventually chooses depends on which side of the foot-placement-phase they end up on. 

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/pi_over_2_two_options.png" alt="pi-over-two" style="width:80%">
<figcaption><b>Which is the natural way to ride a bike when the back has an angle $\theta = \pi/2$ to vertical? (Facing directly into the seat)</b></figcaption>
</figure>

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/neg_pi_over_2_two_options.png" alt="neg-pi-over-two" style="width:80%">
<figcaption><b>Which is more natural? ($\theta = -\pi/2$)</b></figcaption>
</figure>

Now we need to try to evaluate the coolness of these configurations. On the one hand, all are extraordinarily dangerous — it is impossible to see what's in front of you when your face is oriented directly into the seat ($\theta=\pi/2$). Dangerous things are inherently cool, so this works in their favor. On the other hand, these positions also look incredibly stupid, which can be uncool. 

These positions represent exotic configurations in motorcycle riding space, and they might only exist in certain extreme environments: like empty bolivian salt flats, or within the interiors of neutron stars. In these regimes, we might observe a breakdown of our parochial understanding of what motorcycling _is_ -- let alone how cool a given position is. 

Probing these kinds of profound mysteries are exactly what science is about -- so please [take this poll](https://forms.gle/7C9GrTeW9SPvEPWz8) to help contribute to humanity's collective understanding of the coolness of motorcycle-riding positions. 

# **UPDATE**

Big news everyone -- I was recently informed by my estonian architect friend that one of these exotic phases of motorcycle usage [has already been employed](https://en.wikipedia.org/wiki/Rollie_Free) in the real world. 

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/exotic_biking.png" alt="exotic-biking" style="width:80%">
<figcaption><b>Rollie Free, what a legend.</b></figcaption>
</figure>

He was aptly named "Rollie Free" -- and roll free he did, in a speed-o no less. This is science in action. I (a scientist) make a prediction and, like Einstein's prediction of gravitational lensing being observed during a full solar eclipse, my prediction of the existence of an exotic state of motorcycling is confirmed by examining nature (wikipedia). Rollie even did this this on salt flats (Bonneville Salt Flats) -- just as I predicted.

<figure style="display: flex; flex-direction: row; justify-content: center; align-items: center; text-align: center; gap: 20px;">
  <div style="text-align: center;">
    <img src="{{site.baseurl}}/assets/images/post_6/pi_over_2_2.png" alt="exotic-biking-theory" style="width:45%;">
    <figcaption><b>Theoretical prediction.</b></figcaption>
  </div>
  <div style="text-align: center;">
    <img src="{{site.baseurl}}/assets/images/post_6/exotic_biking.png" alt="exotic-biking-reality" style="width:45%;">
    <figcaption><b>Experimental evidence.</b></figcaption>
  </div>
</figure>

Rollie's near-nudity reinforces his insane coolness more than theoretical predictions ever could. We now know for sure how one rides a motorcycle when they drive at an angle of $\pi/2$ to vertical, and how profoundly cool they look. 

<figure style="display: flex; flex-direction: column; align-items: center; text-align: center;">
<img src="{{site.baseurl}}/assets/images/post_6/Rollie_free_1920.png" alt="exotic-biking" style="width:40%">
<figcaption><b>This time Rollie decided against his traditional underwear-only approach to motorcycling. Perhaps this was after a severe chafing incident.</b></figcaption>
</figure>

I'll be sure to keep this blog updated _the second_ I learn of further real-world examples of exotic phases of motorcycle riding, and their corresponding coolnesses. 
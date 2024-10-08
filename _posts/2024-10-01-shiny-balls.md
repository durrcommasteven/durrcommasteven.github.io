---
title: "Shiny Balls"
last_modified_at: 2024-08-01T16:20:02-05:00
categories:
  - Blog
tags:
  - Projects
  - Ray tracing
  - Fractals

excerpt: ""
header:
  overlay_image: /assets/images/post_5/Banner.png
  overlay_filter: 0 # same as adding an opacity of 0.5 to a black background

---

[see the notebook here](https://github.com/durrcommasteven/RayTracer/blob/main/ray_tracing.ipynb){: .btn .btn--warning}

A while ago my friend mentioned how it would be cool to code up a ray tracer. I agreed, it did seem like a cool thing to do, and so I made one. I think the exciting thing about ray tracing is how intuitive it is. You hear about how it works, and immediately you feel like you know how to make one. In fact, it's so intuitive that we kind of invented it before we figured out how vision actually works.

The ancient Greeks' idea (emission theory) was that 'eye-beams' leave their respective eyeballs and bounce around, eventually hitting something with some color and brightness that they (somehow) then would convey back to the eye. 

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/emission_theory_picture.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>A scientific diagram presumably explaining how a emission theory helps us see dragons.<br>(Dragon helpfully placed between A and B)</b></figcaption>
</figure>

Now it turns out that this is, of course, not how vision works. Thanks to modern science, we all now know that we're in a simulation, and that our optic nerves are mere lines of code in a supercomputer which, when it finally finishes updating, will restart and end our collective existence. 

But before the great reboot occurs, we all need something to occupy our time. This brings us back to videogames, 3D graphics, and ray tracing. 

Just like emission theory, ray-tracing (also spelled (by me – depending on which one I want to use) raytracing) is a way of generating graphics by simulating the light rays 'leaving' an eye. Despite the ancient philosophers being wrong about the underlying physics, it turns out that ray-tracing totally works as a way of generating images.

In essence, the only photons that actually matter when you see something are the ones that are going to hit your eyeball. And if you time-reverse these, they're leaving your eyeball – bam emission theory (and ray-tracing). 

One slight difference is that technically this doesn't simulate an eyeball – it simulates a pinhole camera, which has no depth of field – but let's ignore this. 

# The Goal 

I want to construct the most basic nontrivial ray-tracing environment possible. I want reflections, shades, interesting shapes, but nothing too hard. My solution is to define an environment of shiny balls in a room with different shaded walls.

To be specific, our room is a 1×1×1 cube, with opposite corners on (0,0,0) and (1,1,1).

I also made the bold (easier) choice of having everything be greyscale. 

The spheres will be perfectly reflective, and the walls will be perfectly absorbing. 

# Coding up Ray Tracing 

The math behind ray tracing really is easy. If you can break your surface into triangles, then you basically take your rays, find if they intersect with your triangles, and reflect or absorb as desired. 

## The Math

_skip if you don't care_

A given ray starts at a position, $\vec{pos}$, and points in a direction $\vec{v}$ (we'll take ||V||=1). 

Basically, we just want to solve for how far a ray has to go before it hits the plane defined by the triangle (that's the variable $c$ in the following):
$$
\vec{pos.} + c \cdot \vec{v} = \vec{z}_1 + a*(\vec{z}_2 - \vec{z}_1) + b*(\vec{z}_3 - \vec{z}_1)
$$

Then to make sure our ray is actually hitting inside the triangle, we just need to check that $a>=0$, $b>=0$, $a+b <= 1$. And finally, we want the ray to be moving forward $c>=0$.

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/math_drawing.png" alt="laffer-sketch" style="width:100%">
</figure>

# Shading the Room, Shining the Balls 

I don't know what it is about these rooms I was making early on, but they were 100% cursed. Like something about them was off. 

#example of cursed room 
<figure>
<img src="{{site.baseurl}}/assets/images/post_5/cursed_room.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>An example of a cursed room. I think I've had nightmares that took place here.</b></figcaption>
</figure>

## Disco Balls

Unfortunately, it turns out that when you try to triangulate a perfectly reflective sphere, you end up with a disco ball. 

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/disco_ball.jpg" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>A disco ball in the corner of a still somewhat cursed room</b></figcaption>
</figure>

We need a solution

Each wall in my room is its own shade of gray. But this alone really does not look good. Depth isn't really conveyed well. 

Humans rely really heavily on shading to infer depth, so let's add some shading in here. 
After a little tinkering, I settled on this function to make things look pretty 

$$
\text{shade} = \frac{\text{init_shade}}{(0.1 + \text{wall_distance}/2)^{1.2}}
$$

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/no_shading_and_shading.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>A weird angle in a cursed room (left) and a room which has been exorcised (right).<br> Not only does shading make things look less haunted, we can also tell what we're seeing.</b></figcaption>
</figure>

Thankfully, reflecting things off a sphere might be even easier than reflecting them off a triangle. Just solve for when (if ever) the ray is a distance $r$ (the radius) from the center of a sphere, $\vec{x}$. 

$$ 
||(\vec{pos.} + c \cdot \vec{v}) - \vec{x}||^2 = r^2
$$
We solve this with the regular old quadratic formula, taking $|v| = 1$ and writing $\vec{x}^* = \vec{x} - \vec{p}$ gives us:
$$
c^2 - 2 c \vec{v} \cdot \vec{x}^* + (||\vec{x}^*||^2 -r^2) = 0
$$
Find your solution with real and positive $c$, and you're done. 

Putting this all together, we just have to place our spheres, and start reflecting. With each reflection iteration the remaining unabsorbed light rays bounce one more time.

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/initial_example.gif" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>Iterative reflections inside an un-cursed room.</b></figcaption>
</figure>

There, done. Now what? 

Well we can ask ChatGPT to make sculptures for us

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/ai_scuptures.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>Four sculptures made by o1. <br>(Top left) ChatGPT's embarassing attempt at being deep and making a question mark. <br>(Top right) A double helix which ChatGPT says represents the fact that it, too is made of a sort of genetic code of data. <br>(Bottom left) ChatGPT claimed that these tiny spheres correspond to the corners of a projected tesseract (I did not check this). <br>(Bottom right) A spiral, because ChatGPT likes spirals (it does look neat).</b></figcaption>
</figure>

# In Search of Fractals

But honestly I want to make something way more cool looking than these images – and what's more cool-looking than a fractal? Nothing, that's what. I recently learned about something called apollonian sphere packing – it's an extension of the apollonian gasket to 3d, and 
[this blog](https://observablehq.com/@esperanc/3d-apollonian-sphere-packings) (which cites [this other blog](https://www.keanw.com/2012/02/sphere-packing-in-autocad-creating-an-apollonian-packing-using-f-part-2.html)) described a pretty simple way to generate it. 

So I coded it up and started making pretty pictures with my ray tracer:

## Iteration 1 
At iteration 1, we have 4 spheres in a tetrahedron. 

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/app_8.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>Generation 1 of an Apollonian sphere packing (4 shiny balls in a tetrahedron).</b></figcaption>
</figure>

Looks pretty cool. Let's go to generation 2

## Iteration 2

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/high-quality-balls.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>Generation 2 of an Apollonian sphere packing (shiny balls are adding up).</b></figcaption>
</figure>

Cool, now we have a weird shiny raspberry

Already these visualizations raise some interesting questions, like 
- What is the fractal dimension of this [sphere packing](https://www.worldscientific.com/doi/abs/10.1142/S0218348X94000739)?
- And why do I want to bite into these spheres so much?

## Higher Iterations 

While my code allows for GPU use, I'm using my tiny GPU-less laptop, and after generation 2 it starts to complain.

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/num_spheres.png" alt="laffer-sketch" style="width:50%">
</figure>

As you increase the generations of this fractal, the number of spheres increases exponentially. Here's just the spheres which make up the 6th generation.

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/last_gen_6.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>Even after 1000 reflections, the light rays are getting stuck between spheres <br>(note the dark shadows between nearby spheres).</b></figcaption>
</figure>

This kind of sucks, and although it's certainly possible to plot these sphere packings at high fractal generations, I'm being maximally dumb here, and using essentially no tricks to speed things up. Ray tracing is also just known for being slow. Tons of rays just get stuck inside this massive sphere of spheres – that's all the black regions between them. 

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/other_persons_apollonian.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b><a href="https://observablehq.com/@esperanc/3d-apollonian-sphere-packings" target="_blank">This blog post</a> plots Apollonian sphere packing in a much more efficient way (not using ray tracing).</b></figcaption>
</figure>

Let's try to get a fractal some other way – actually, let's try to _use_ those annoying dark regions where rays are getting stuck. 

# Secret Sierpinski 

Remember that first generation with the tetrahedron? Well look at the middle – there are three spheres being reflected. Each of those spheres is reflecting its own smaller set of three spheres, etc. This is feeling pretty fractal-y

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/app_8_emph.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>What's going on in here?</b></figcaption>
</figure>

Look between those three spheres. There are some rays that are still bouncing around a few generations, which show up as black. This is clearer if we plot just the rays which haven't hit a wall yet. 

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/serpinski_zoom_1.0_inverted_app.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>All the rays which are still reflecting after ~ 10 reflections iterations (in black).</b></figcaption>
</figure>

This looks quite a bit like a Sierpinski triangle – and it looks that way because it's being constructed the same way. Consider how repeated reflections work here. When a bunch of rays go between our four spheres, each sphere reflects them at the three other spheres. This gives us three dark patches in our image. But each of these dark gaps is reflected at three other spheres too, etc. This leads to an iterative construction that exactly matches that of a Sierpinski triangle.

<figure>
<img src="{{site.baseurl}}/assets/images/post_5/Sierpinski_triangle_evolution.png" alt="laffer-sketch" style="width:100%">
<figcaption align = "center"><b>Sierpinski triangles are constructed using the same rule that determines how reflections work in the tetrahedron of shiny balls.</b></figcaption>
</figure>

It turns out there's a close relationship between apollonian gaskets (which our sphere-packing is a 3d version of) and Siepinski triangles – they share the same combinatorial structure. But who cares about that honestly. Let's just see some pretty pictures 

<figure style="text-align: center;">
    <div style="display: flex; justify-content: center;">
        <img src="{{site.baseurl}}/assets/images/post_5/zoom_video_files_7.34145301536491.png" alt="laffer-sketch" style="width: 50%; margin-right: 10px;">
        <img src="{{site.baseurl}}/assets/images/post_5/inverted_zoom_video_files_7.34145301536491.png" alt="laffer-sketch" style="width: 50%;">
    </div>
    <figcaption><b>Prepare your body for entering the sphere. The ball reflections (left) and the still-reflecting rays in black (right).</b></figcaption>
</figure>

<figure style="text-align: center;">
    <div style="display: flex; justify-content: center;">
        <img src="{{site.baseurl}}/assets/images/post_5/zoom_video_files_6992.9295994298245.png" alt="laffer-sketch" style="width: 50%; margin-right: 10px;">
        <img src="{{site.baseurl}}/assets/images/post_5/inverted_zoom_video_files_6992.9295994298245.png" alt="laffer-sketch" style="width: 50%;">
    </div>
    <figcaption><b>The sphere pattern at roughly 7000x zoom. Conveniently, since this is a fractal, you do get the picture at this point. It's all the same, literally.</b></figcaption>
</figure>

Now I know what you're thinking: "does this mean that if I get 4 ball-bearings together, go into a white room, make myself invisible, and zoom in between them, I'll see a fractal??"

I think so, but that whole procedure seems really involved. How about we zoom in with this ray tracer thing and see what happens. (I've always wanted to make one of those videos zooming into a fractal)

{% include video id="s5R1V4rIczk" provider="youtube" %}

{% include video id="-jbl_bxJAJw" provider="youtube" %}

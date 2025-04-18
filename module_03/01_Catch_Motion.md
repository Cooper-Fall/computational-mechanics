---
jupytext:
  formats: notebooks//ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.2
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

> __Content modified under Creative Commons Attribution license CC-BY
> 4.0, code under BSD 3-Clause License © 2020 R.C. Cooper, L.A. Barba,N.C. Clementi__

+++

# Catch things in motion

This module of the _Computational Mechanics_ course is our launching pad to investigate _change_, _motion_, and _dynamics_, using computational thinking, Python, and Jupyter.

The foundation of physics and engineering is the subject of **mechanics**: how things move around, when pushed around. Or pulled... in the beginning of the history of mechanics, Galileo and Newton sought to understand how and why objects fall under the pull of gravity.

This first lesson will explore motion by analyzing images and video, to learn about velocity and acceleration.

+++

## Acceleration of a falling ball

Let's start at the beginning. Suppose you want to use video capture of a falling ball to _compute_ the acceleration of gravity. Could you do it? With Python, of course you can!

Here is a neat video you found online, produced over at MIT several years ago [1]. It shows a ball being dropped in front of a metered panel, while lit by a stroboscopic light. Watch the video!

You can find several toolkits for handling images and video with Python; you'll start with a simple one called [`imageio`](https://imageio.github.io). Import this library like any other, and let's load `numpy` and `pyplot` while you're at it.

```{code-cell} ipython3
import imageio 
import numpy as np
import matplotlib.pyplot as plt
plt.style.use('fivethirtyeight')
```

### Read the video

With the `get_reader()` method of `imageio`, you can read a video from its source into a _Reader_ object. You don't need to worry too much about the technicalities here—you'll walk you through it all—but check the type, the length (for a video, that's number of frames), and notice you can get info, like the frames-per-second, using `get_meta_data()`.

If you do, you suggest to install `imageio-ffmpeg` package, an ffmpeg wrapper for Python that includes the `ffmpeg` executable. You can install it via `conda`:

 `conda install imageio-ffmpeg -c conda-forge`

+++

### Show a video frame in an interactive figure

With `imageio`, you can grab one frame of the video, and then use `pyplot` to show it as an image. But you want to interact with the image, somehow.

So far in this course, you have used the command `%matplotlib inline` to get our plots rendered _inline_ in a Jupyter notebook. There is an alternative command that gives you some interactivity on the figures: `%matplotlib notebook`. Execute this now, and you'll see what it does below, when you show the image in a new figure.

Let's also set some font parameters for our plots in this notebook.

```{code-cell} ipython3
%matplotlib widget
```

Now you can use the `get_data()` method on the `imageio` _Reader_ object, to grab one of the video frames, passing the frame number. Below, you use it to grab frame number 1100, and then print the `shape` attribute to see that it's an "array-like" object with three dimensions: they are the pixel numbers in the horizontal and vertical directions, and the number of colors (3 colors in RGB format). Check the type to see that it's an `imageio` _Image_ object.

To use this ability, you write a function with the events you want to capture, and then connect this function to the Matplotlib "event manager" using [`mpl_connect()`](https://matplotlib.org/devdocs/api/backend_bases_api.html#matplotlib.backend_bases.FigureCanvasBase.mpl_connect). In this case, you connect the `'button_press_event'` to the function named `onclick()`, which captures the $(x,y)$ coordinates of the mouse click on the figure. Magic!

+++

## Projectile motion

Now, you'll study projectile motion, using a video of a ball "fired" horizontally, like a projectile. Here's a neat video you found online, produced by the folks over at [Flipping Physics](http://www.flippingphysics.com) [2].

```{code-cell} ipython3
from IPython.display import YouTubeVideo
vid = YouTubeVideo("Y4jgJK35Gf0")
display(vid)
```

```{code-cell} ipython3
def onclick(event):
    '''Capture the x,y coordinates of a mouse click on the image'''
    ix, iy = event.xdata, event.ydata
    coords.append([ix, iy]) 
```

We used Twitter to communicate with the author of the video and ask permission to use it in this lesson. A big _Thank You_ to Jon Thomas-Palmer for letting us use it!

```{code-cell} ipython3
%%html
<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">You have my permission to use the video. Please provide attribution to me. I’d enjoy seeing what you do with it too!</p>&mdash; Jon Thomas-Palmer (@FlippingPhysics) <a href="https://twitter.com/FlippingPhysics/status/926785273538666497?ref_src=twsrc%5Etfw">November 4, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 
```

### Capture mouse clicks on many frames with a widget

Capture the coordinates of mouse clicks on a _sequence_ of images, so
that you may have the positions of the moving ball caught on video. We
know how to capture the coordinates of mouse clicks, so the next
challenge is to get consecutive frames of the video displayed for us, to
click on the ball position each time. 

Widgets to the rescue! There are currently [10 different widget types](http://ipywidgets.readthedocs.io/en/stable/examples/Widget%20List.html) included in the `ipywidgets` library. The `BoundedIntText()` widget shows a text box with an integer value that can be stepped from a minimum to a maximum value by clicking up/down arrows. Stepping through frames with this widget, and clicking on the ball position each time, gets us what you want.

Digitizing the ball positions in this way is a bit tedious. But this could be a realistic scenario: you captured video of a moving object, and you need to get position data from the video frames. Unless you have some fancy motion-capture equipment, this will do the trick.

Let's load the Jupyter widgets:

```{code-cell} ipython3
from ipywidgets import widgets
```

Download the video, previously converted to .mp4 format to be read by `imageio`, and then load it to an `imageio` _Reader_. Notice that it has 3531 frames, and they are 720x1280 pixels in size. 

Below, you're showing frame number 52, which you found to be the start of the portion shown at 50% speed. Go ahead and use that frame to capture mouse clicks on the intersection of several $10\rm{cm}$ lines with one vertical, so you can calculate the scaling from pixels to physical distance.

```
from urllib.request import urlretrieve
URL = 'http://go.gwu.edu/engcomp3vid1?accessType=DOWNLOAD'
urlretrieve(URL, 'Projectile_Motion.mp4')
```

```{code-cell} ipython3
reader = imageio.get_reader('Projectile_Motion.mp4')
```

```{code-cell} ipython3
image = reader.get_data(52)
image.shape
```

```{code-cell} ipython3
fig = plt.figure()
plt.imshow(image, interpolation='nearest')

coords = []
connectId = fig.canvas.mpl_connect('button_press_event', onclick)
```

## Exercise 

Grab the coordinates of the 0, 10, 20, 30, 40, ..., 100-cm vertical positions so you can create a vertical conversion from pixels to centimeters with `gap_lines`.

```{code-cell} ipython3
coords
```

```{code-cell} ipython3
y_lines = np.array(coords)[:,1]
y_lines
```

```{code-cell} ipython3
gap_lines = y_lines[1:] - y_lines[0:-1]
gap_lines.mean()
```

You computed the vertical distance between in pixels between  distances
of $10\rm{cm}$ marks (averaging over your clicks). The
0.1-m/`gap_lines.mean()` converts distances in pixels to distances in
meters.

+++

Now the fun part! Study the code below: you create a `selector` widget
of the `BoundedIntText` type, taking the values from 52 to 77, and
stepping by 1. We already played around a lot with the video and found
this frame range to contain the portion shown at 50% speed. 

Re-use the `onclick()` function, appending to a list named `coords`, and
you call it with an event connection from Matplotlib, just like before.
But now you add a call to
[`widgets.interact()`](http://ipywidgets.readthedocs.io/en/stable/examples/Using%20Interact.html),
using a new function named `catchclick()` that reads a new video frame
and refreshes the figure with it.

Execute this cell, then click on the ball position, advance a frame,
click on the new ball position, and so on, until frame 77. The mouse
click positions will be saved in `coords`.

Its better to click on the bottom edge of the ball image, rather than
attempt to aim at the ball's center.

```{code-cell} ipython3
selector = widgets.BoundedIntText(value=52, min=52, max=77, step=1,
    description='Frame:',
    disabled=False)

coords = []

def catchclick(frame):
    image = reader.get_data(frame)
    plt.imshow(image, interpolation='nearest');


fig = plt.figure()

connectId = fig.canvas.mpl_connect('button_press_event',onclick)

widgets.interact(catchclick, frame=selector);
```

```{code-cell} ipython3
coords # view the pixel coordinates of the projectile
```

Now, convert the positions in pixels to meters, using your scaling for
this video, and save the $x$ and $y$ coordinates to new arrays. Below,
you plot the ball positions that you captured.

```{code-cell} ipython3
x = np.array(coords)[:,0] *0.1 / gap_lines.mean()
y = np.array(coords)[:,1] *0.1 / gap_lines.mean()
```

```{code-cell} ipython3
# make a scatter plot of the projectile positions
fig = plt.figure()
plt.scatter(x,-y);
```

Finally, compute the vertical displacements, then get the vertical velocity and acceleration. And why not repeat the process for the horizontal direction of motion. The time interval is $1/60$ seconds, according to the original video description, i.e., 60 frames per second.

```{code-cell} ipython3
delta_y = (y[1:] - y[0:-1])
```

```{code-cell} ipython3
vy = delta_y * 60
ay = (vy[1:] - vy[:-1]) * 60
print('The acceleration in the y direction is: {:.2f}'.format(ay.mean()))
```

```{code-cell} ipython3
delta_x = (x[1:] - x[:-1])
vx = delta_x * 60
ax = (vx[1:] - vx[:-1]) * 60
print('The acceleration in the x direction is: {:.2f}'.format(ax.mean()))
```

## Saving your hard work

You have put a lot of effort into processing these images so far. Let's
save your variables `time`, `x`, and `y` so you can load it back later. 

Use the command `np.savez(file_name, array1,array2,...)` to save your arrays for use later.

The x-y-coordinates occur at 1/60 s, 2/60s, ... len(y)/60s = `np.arange(0,len(y))/60`

```{code-cell} ipython3
t = np.arange(0,len(y))/60
np.savez('../data/projectile_coords.npz',t=t,x=x,y=-y)
```

## Discussion

* What did you get for the $x$ and $y$ accelerations? What did your colleagues get?
* Do the results make sense to you? Why or why not?

+++

## Numerical derivatives

You just computed the average velocity between two captured ball positions using _numerical derivative_. The velocity is the _derivative_ of position with respect to time, and you can approximate its instantaneous value with the average velocity between two close instants in time:

\begin{equation}
v(t) = \frac{dy}{dt} \approx \frac{y(t_i+\Delta t)-y(t_i)}{\Delta t}
\end{equation}

And acceleration is the _derivative_ of velocity with respect to time; you can approximate it with the average acceleration within a time interval:

\begin{equation}
a(t) = \frac{dv}{dt} \approx \frac{v(t_i+\Delta t)-v(t_i)}{\Delta t}
\end{equation}

As you can imagine, the quality of the approximation depends on the size of the time interval: as $\Delta t$ gets smaller, the error also gets smaller.

+++

### Using high-resolution data

Suppose you had some high-resolution experimental data of a falling ball. Might you be able to compute the acceleration of gravity, and get a value closer to the actual acceleration of $9.8\rm{m/s}^2$?

You're in luck! Physics professor Anders Malthe-Sørenssen of Norway has some high-resolution data on the youbsite to accompany his book [3]. We contacted him by email to ask for permission to use the data set of a falling tennis ball, and he graciously agreed. _Thank you!_ His data was recorded with a motion detector on the ball, measuring the $y$ coordinate at tiny time intervals of $\Delta t = 0.001\rm{s}$. Pretty fancy.

```{code-cell} ipython3
filename = '../data/fallingtennisball02.txt'
t, y = np.loadtxt(filename, usecols=[0,1], unpack=True)
```

Okay! You should have two new arrays with the time and position data. Let's get a plot of the ball's vertical position.

```{code-cell} ipython3
fig = plt.figure(figsize=(6,4))
plt.plot(t,y);
```

Neat. The ball bounced 3 times during motion capture. Let's compute the
acceleration during the first fall, before the bounce. We can find the bounce locations in different ways. Here, we will look for large changes in velocity. When the ball reaches the largest negative velocity, that's the point when it strikes the ground, 

```{code-cell} ipython3
delta_y = (y[1:] - y[:-1])
```

```{code-cell} ipython3
dt = t[1]-t[0]
dt
```

```{code-cell} ipython3
vy = delta_y / dt
plt.figure();
plt.plot(t[0:-1], vy, 'o')

first_impact = np.argmin(vy)

plt.plot(t[first_impact], 
         vy[first_impact], 
         '*', 
         markersize = 20)

plt.text(t[first_impact], 
         vy[first_impact], 'first impact')
```

```{code-cell} ipython3
ay = (vy[1:first_impact] - vy[:first_impact-1]) / dt
print('The acceleration in the y direction is: {:.2f}'.format(ay.mean()))
```

Gah. Even with this high-resolution data, you're getting an average value of acceleration that is smaller than the actual acceleration of gravity: $9.8\rm{m/s}^2$. _What is going on?_ Hmm. Let's make a plot of the acceleration values…

```{code-cell} ipython3
fig = plt.figure(figsize=(6,4))
plt.plot(ay);
```

## Discussion

* What do you see in the plot of acceleration computed from the high-resolution data?
* Can you explain it? What do you think is causing this?

+++

## What you've learned

* Work with images and videos in Python using `imageio`.
* Get interactive figures using the `%matplotlib notebook` command.
* Capture mouse clicks with Matplotlib's `mpl_connect()`.
* Observed acceleration of falling bodies is less than $9.8\rm{m/s}^2$.
* Capture mouse clicks on several video frames using widgets!
* Projectile motion is like falling under gravity, plus a horizontal velocity.
* Save our hard work as a numpy .npz file __Check the Problems for loading it back into your session__
* Compute numerical derivatives using differences via array slicing.
* Real data shows free-fall acceleration decreases in magnitude from $9.8\rm{m/s}^2$.

+++

## References

1.  Strobe of a Falling Ball (2008), MIT Department of Physics Technical Services Group, video under CC-BY-NC, available online on [MIT TechTV](http://techtv.mit.edu/collections/physicsdemos/videos/831-strobe-of-a-falling-ball).

2. The Classic Bullet Projectile Motion Experiment with X & Y Axis Scales (2004), video by [Flipping Physics](http://www.flippingphysics.com/bullet-with-scales.html), Jon Thomas-Palmer. Used with permission.

3. _Elementary Mechanics Using Python_ (2015), Anders Malthe-Sorenssen, Undergraduate Lecture Notes in Physics, Springer. Data at http://folk.uio.no/malthe/mechbook/

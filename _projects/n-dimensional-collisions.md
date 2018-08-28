---
layout: page
title: N-Dimensional Collisions
description: N-dimensional physics engine writen in JavaScript
 and rendered with p5.js
demo: https://symmetries.github.io/n-dimensional-collisions
source: https://github.com/Symmetries/n-dimensional-collisions
---

{% include img-align.md %}
{% include math.html %}

![demo image](\images\n-dimensional-collisions\n-dimensional-collisions.png)
{:img-center}

## Description
This project consists of a simple physics engine in an arbitrary number
of dimensions. The engine itself is written in pure JavaScript.
In addition, p5.js was used for rendering.
A demo can be found [here]({{ page.demo }}).
The source code for this project can be found [here]({{ page.source }}).

## Tutorial

### Introduction
In order to create a physics engine in higher dimensions, we will need
to create a `Vector` class in order to describe the particle's position,
velocity and acceleration. Then, we will create a `Hyperball` class which
will make use of the `Vector` class. On the way, we will derive equations
that we will then translate into code. Finally, we will design a user
interface containing an HTML5 canvas which we will draw on using the
p5.js library.

### The Vector Class
The `Vector` class is the cornerstone of the rest of the project.
This class will implement all the common vector operations we will use.
Common operations with vectors include the inner product 
(also known as the dot product),
addition, scalar multiplication, and so on.
This means that we need the following methods:
* `static add(v1, v2)`
* `mult(scl)`
* `static dot(v1, v2)`
* `get norm()` 
* `neg()`

In order to make the implementation and use of this class easier,
it is useful to also define a `get(i)` method which returns the
value of the ith component of the vector.
In addition, we would like to be able to negate one of the
components of the vector, since we would like to make the objects
bounce elastically on the walls (all of which have a corresponding axis). 
This means we should also add a `flip(i)` method.
Another useful operation is the distance between two vectors.
This means the `Vector` class should have a static `dist(v1, v2)` method.
Furthermore, the operations the operations will return
a new `Vector` object
rather than modifying the existing one.
So, we can start writing our class:
{% highlight javascript %}
class Vector {
  constructor(...values) {
    if (values[0] instanceof Array) {
      values = values[0];
    }
    this.values = values;
    this.dim = values.length;
  }

  get(i) {
    return this.values[i];
  }
  
  static add(v1, v2) {
    Vector.checkSizes(v1, v2);

    let result = [];
    for (let i = 0; i < v1.dim; i++) {
      result.push(v1.get(i) + v2.get(i));
    }
    return new Vector(result);
  }

  mult(scl) {
    let result = [];
    for (let i = 0; i < this.dim; i++) {
      result.push(this.get(i) * scl);
    }
    return new Vector(result);
  }

  static dot(v1, v2) {
    Vector.checkSizes(v1, v2);

    let result = 0;
    for (let i = 0; i < v1.dim; i++) {
      result += v1.get(i) * v2.get(i);
    }
    return result;
  }

  get norm() {
    return Math.sqrt(
      Vector.dot(this, this));
  }
  
  static checkSizes(v1, v2) {
    if (v1.dim != v2.dim) {
      throw Error(
        "Vectors must have same dimension"
      );
    }
  }

  static dist(v1, v2) {
    return Vector.sub(v1, v2).norm
  }

  flip(i) {
    let result = [];
    for (let j = 0; j < this.dim; j++) {
      result.push(this.get(j) * 
        (i == j ? -1 : 1));
    }
    return new Vector(result);
  }
}
{% endhighlight %}

The code can be experimented with
[here](https://repl.it/@diegolopez/Vector-Class).
As we can notice, the `norm()` getter has been implemented using
the identity \\(\lVert \vec v \rVert^2 = \vec v \cdot \vec v \\).

### The Hyperball Class

We would like to have a Hyperball class which contains all
the properties and behaviour expected of an n-dimensional ball,
 or n-ball.
So, we are expected to have the following properties
* `r`, a scalar of number denoting the radius of the particle
* `pos`, a `Vector` denoting the position of the particle
* `vel`, a `Vector` denoting the velocity of the particle
* `acc`, a `Vector` denoting the acceleration of the particle
* `mass`, a scalar or number denoting the mass of the particle
* `dim`, a scalar or number denoting the dimension of the particle

We can also expect to have the following methods
* `update(box, dt)`
* `static updateAll(hyperballs, box, dt)`
* `isColliding(other)`
* `collisionResponse(other)`
* `crossSection(cuts)`

The method `update(box, dt)` simply updates the positions of each
`Hyperball` after some amount of time `dt` inside of a region given
by `box`. Indeed, `box` is a `Vector` containing all the lengths
of the sides of the region.
The static method `updateAll(hyperballs, box, dt)` takes an array of
`Hyperball` objects, updates each of them, and also checks for any
collisions. If one is found, then `collisionResponse(other)` is called.
The method `isColliding(other)` returns whether or not there is a
collision with the `Hyperball` other.
The method `collisionResponse(other)` will update the velocities of 
the two `Hyperball` objects that collided.
The formula will be derived in the following subsection.
Finally, the `crossSection(cuts)` method will return the cross
section of the hyperball through hyperplanes of the form
\\(x_i = c \\) defined by the
argument `cuts`.  

#### Elastic Collision in N-Dimensions

In order to calculate the final velocities, we will need to solve a
system of equations. First, we will introduce some notation.
We have

* \\( \vec v_1 \\), the initial velocity of the first particle,
* \\( \vec v_2 \\), the initial velocity of the second particle,
* \\( \vec v_1^\prime \\), the final velocity of the first particle,
* \\( \vec v_2^\prime \\), the final velocity of the second particle,
* \\( \vec s_1 \\), the position of the first particle,
* \\( \vec s_2 \\), the position of the second particle.

We wish to solve the following system of equations:

\\[
\begin{aligned}
\frac{1}{2}m_1 \left \lVert \vec v_1 \right \rVert ^2 + 
  \frac{1}{2}m_2 \left \lVert \vec v_2 \right \rVert ^2 &=
  \frac{1}{2}m_1 \left \lVert \vec v_1^\prime \right \rVert ^2 +
  \frac{1}{2}m_2 \left \lVert \vec v_2^\prime \right \rVert ^2 &\; (1)\\\\ 
m_1 \vec v_1 + m_2 \vec v_2 &=
  m_1 \vec v_1^\prime + m_2 \vec v_2^\prime &\; (2) \\\\ 
m_2 (\vec v_2^\prime - \vec v_2) &= \lambda( \vec s_2 - \vec s_1 ) &\; (3)
\end{aligned}
\\]

The equations (1), (2) and (3) are due to conservation of energy,
conservation of momentum and squishification, respectively.
In truth, the last equation is not actually called squishification,
but it seems like a good name for it. In essence, the change in
momentum of the first (or second) particle must be along the same axis
which passes through the centers of the two particles.
The following diagram should show it more clearly:

![squishification diagram](\images\n-dimensional-collisions\squishification.png)
{: img-center}

The two particles are in contact for a short amount of time 
\\( \Delta t \\)
between \\( t_0 \\) and \\( t_1 \\). During that time, the two particles
apply a force equal in magnitude but opposite in direction on each other.
It suffices to only look at the force applied on \\( m_2 \\).
In truth, the force applied on \\( m_2 \\) varies over time between
\\( t_0 \\) and \\( t_1 \\). This means we need to integrate the force
over that interval. So, we have

\\[ \int_{t_0}^{t_1} \vec F_{1 \to 2} dt. \\]

However, force is the derivative of momentum, so we can write

\\[ \int_{t_0}^{t_1} \frac{d\vec p_2}{dt} dt.\\]

By the fundamental theorem of calculus, we have

\\[ \Delta \vec p_2 = \int_{t_0}^{t_1} \vec F_{1 \to 2} dt. \\]

From the above diagram, we can see that
\\(\vec F(t) = \lambda(t) (\vec s_2 - \vec s_1) \\).
Substituting, we have

\\[
\Delta \vec p_2 = \int_{t_0}^{t_1} \lambda(t) (\vec s_2 - \vec s_1) dt,
\\]

which simplifies to

\\[
\Delta \vec p_2 = (\vec s_2 - \vec s_1) \int_{t_0}^{t_1} \lambda(t) dt.
\\]

Since \\( \int_{t_0}^{t_1} \lambda(t) \\) is just a number, we can write

\\[
m_2 (\vec v_2^\prime - \vec v_2) = 
  \lambda \left ( \vec s_2 - \vec s_1 \right ),
\\]

which is what we wanted.
It is possible to find a similar expression for \\( m_2 \\)
using equations (2) and (3) by first rewriting (2).

\\[
\begin{aligned}
m_1 \left (\vec v_1^\prime - \vec v_1\right ) &= 
  -m_2 \left (\vec v_2^\prime - \vec v_2 \right ) \\\\ 
m_1 \left (\vec v_1^\prime - \vec v_1\right ) &= 
  -\lambda (\vec s_2 - \vec s_1) &\; (4)
\end{aligned}
\\]

Now, we simplify equation (1).
\\[
m_1 \left ( \\| \vec v_1^\prime \\| ^2 - \| \vec v_1 \\| ^2\right ) =
  -m_2 \left ( \\| \vec v_2^\prime \\| ^2 - \\| \vec v_2 \\| ^2 \right ) 
  \quad (5) 
\\]

We use the identiy
\\( \\| \vec v \\| ^2 - \\| \vec u\\| ^2 
  = (\vec v - \vec u) \cdot (\vec v + \vec u) \\)
to rewrite (5).

\\[
\begin{aligned}
&m_1(\vec v_1^\prime - \vec v_1) \cdot 
  (\vec v_1^\prime + \vec v_1)
  =-m_2 (\vec v_2^\prime - \vec v_2) \cdot
  (\vec v_2^\prime + \vec v_2) \quad (6)
\end{aligned}
\\]

Plugging (3) and (4) into (6):

\\[
-\lambda(\vec s_2 - \vec s_1)\cdot (\vec v_1^\prime + \vec v_1) =
  -\lambda(\vec s_2 - \vec s_1) \cdot (\vec v_2^\prime + \vec v_2)
\\]

Since \\( \lambda \neq 0 \\), we have

\\[
(\vec s_2 - \vec s_1)\cdot (\vec v_1^\prime + \vec v_1) =
  (\vec s_2 - \vec s_1) \cdot (\vec v_2^\prime + \vec v_2)
  \quad (7)
\\]

From equations (3) and (4), we can isolate
\\( \vec v_1^\prime \\) and \\(\vec v_2^\prime \\). 

\\[
\begin{aligned}
\vec v_1^\prime &= 
  \frac{-\lambda}{m_1}(\vec s_2 - \vec s_1) + \vec v_1 &\; (8) \\\\ 
\vec v_2^\prime &=
  \frac{\lambda}{m_2} (\vec s_2 - \vec s_1) + \vec v_2 &\; (9)
\end{aligned}
\\]

Substituting (8) onto the left side of (7)

\\[
\begin{aligned}
&(\vec s_2 - \vec s_1) \cdot \left ( 
  \frac{-\lambda}{m_1} (\vec s_2 - \vec s_1) + 2\vec v_1 \right ) 
  = \frac{-\lambda}{m_1} \\| \vec s_2 - \vec s_1 \\| ^2 + 
  2\vec v_1 \cdot (\vec s_1 - \vec s_1)
\end{aligned}
\\]

Substituting (9) onto the right side of (7)

\\[
\begin{aligned}
&(\vec s_1 - \vec s_1) \cdot \left (
  \frac{\lambda}{m_2} ( \vec s_2 - \vec s_1) + 2 \vec v_2  \right ) 
  = \frac{\lambda}{m_2} \\| \vec s_2 - \vec s_1 \\|^2 +
  2\vec v_2 \cdot (\vec s_2 - \vec s_1)
\end{aligned}
\\]

Combining both sides

\\[
\begin{aligned}
&\frac{-\lambda}{m_1} \\| \vec s_2 - \vec s_1 \\| ^2 + 
  2\vec v_1 \cdot (\vec s_2 - \vec s_1)
  = \frac{\lambda}{m_2} \\| \vec s_2 - \vec s_1 \\|^2 +
  2\vec v_2 \cdot (\vec s_2 - \vec s_1)
\end{aligned}
\\]

Now we can solve for \\( \lambda \\)

\\[
\begin{aligned}
\lambda \left ( \frac{1}{m_1} + \frac{1}{m_2}\right)
   \\| \vec s_2 - \vec s_1 \\| ^2 &=
  -2(\vec s_2 - \vec s_1) \cdot (\vec v_2 - \vec v_1) \\\\ 
\lambda \left (\frac{m_1 + m_2}{m_1 m_2}\right )
  \\| \vec s_2 - \vec s_1 \\| ^2 
  &=-2(\vec s_2 - \vec s_1) \cdot (\vec v_2 - \vec v_1) \\\\ 
  \lambda &= \frac{-2m_1 m_2}{m_1 + m_2} 
  \frac {(\vec s_2 - \vec s_1) \cdot (\vec v_2 - \vec v_1)}
  {\\| \vec s_2 - \vec s_1 \\| ^2} 
\end{aligned}
\\]

We can now substitute \\( \lambda \\) back into equations (8) and (9)

\\[
\begin{aligned}
\vec v_1^\prime &= \frac{2 m_2}{m_1 + m_2} 
  \frac{(\vec s_2 - \vec s_1) \cdot (\vec v_2 - \vec v_1)}
  {\\| \vec s_2 - \vec s_1 \\|^2}
   (\vec s_2 - \vec s_1) + \vec v_1 \\\\ 
\vec v_2^\prime &= \frac{-2m_1}{m_1 + m_2}
  \frac{(\vec s_2 - \vec s_1) \cdot (\vec v_2 - \vec v_1)}
  {\\| \vec s_2 - \vec s_1 \\|^2}
  (\vec s_2 - \vec s_1) + \vec v_2
\end{aligned}
\\]

which concludes the derivation.

#### Hyperball Cross Sections

Projecting higher dimensional objects into a 2D screen is far
from a trivial problem. There are many ways to solve this, and
this project will solve it by computing a cross section of the
hypersballs. However, in the case of a 6 dimensional object,
for example, a cross section of a cross section of a cross section
must be computed in order to be displayed as an ordinary 3D ball.
The easiest cross sections to compute are those which are perpendicular
to a certain axis, i.e. given by hyperplanes of the form
\\( x_i = c \\). In general, we would like to be able to compute the resulting
hyperball after being "sliced" by a sequence of hyperplanes 
\\( x_{k_1} = c_1, \ x_{k_2} = c_2, \ \cdots, \ x_{k_m} = c_m \\).
In other words, we are interested in the points which satisfy those equations as
well as the equation for the initial hyperball:

\\[
\left \\{ \vec p \in \mathbb{R}^n \middle | \\|\vec p - \vec o\\| \leq r \right \\},
\\]

where \\( \vec o \\) is the center of the hyperball and \\( r \\)
is its radius.
To simplify this problem, we can find an equation for a single slice 
\\( x_k = c \\) and simply apply the equation as many times as necessary.
In components, this is written as:

\\[
(x_1 - o_1)^2 + \cdots + (x_{k-1} - o_{k-1})^2 + 
(x_k - o_k)^2 + (x_{k+1} - o_{k+1})^2 + \cdots + (x_n - o_n)^2 \leq r^2
\\]

Now, we substitute \\( x_k = c \\):

\\[
\begin{aligned}
\(x_1 - o_1)^2 + \cdots + (x_{k-1} - o_{k-1})^2 + 
(c - o_k)^2 + (x_{k+1} - o_{k+1})^2 + \cdots (x_n - o_n)^2 &\leq r^2 \\\ 
(x_1 - o_1)^2 + \cdots + (x_{k-1} - o_{k-1})^2 + 
(x_{k+1} - o_{k+1})^2 + \cdots (x_n - o_n)^2 &\leq r^2 - (c - o_k)^2
\end{aligned}
\\]

This equations only has solutions if \\( r^2 - (c - o_k)^2 \geq 0\\).
If it does, then we have a new equation in terms of 
\\( x_1, \cdots, x_{k-1}, x_{k+1}, \cdots, x_k \\).
We can now ignore the component \\(x_k\\) and now we have a
new hypersphere in \\( n-1 \\) dimensions with 

\\[
\vec o^\prime = \begin{bmatrix}o_1 \\\\ o_2 \\\\ \vdots \\\\ 
o_{k-1} \\\\ o_{k+1} \\\\ \vdots \\\\ o_{n-1} \\\\ o_n \end{bmatrix},
\quad r^\prime = \sqrt{r^2 - (c - o_k)^2}.
\\]

In order to do the generalized version with \\( m \\) slices,
we simply apply this process on each successive cross section.

#### Towards or Away?

There is one more thing to take into consideration:
There might be a case were two hyperballs are colliding, 
but they are actually moving away from each other.
This depends on the sign of the dot product between
\\( \vec v_2 - \vec v_1 \\) and \\( \vec s_2 - \vec s_1 \\).
Indeed, \\( \vec s_2 - \vec s_1 \\) is the vector going from
the center of the first particle to the center of the second particle
and \\( \vec v_2 - \vec v_1 \\) is the second particle's velocity from
the first particle's frame of reference. If the dot product is negative,
it means these two vectors are pointing in opposite directions and so the
particles are moving towards each other.

#### The Code

We can finally write our `Hyperball` class.

{% highlight javascript %}
class Hyperball {
  constructor(radius, pos, vel, acc, mass, dim) {
    this.radius = radius;
    this.pos = pos;
    this.vel = vel;
    this.acc = acc;
    this.mass = mass;
    this.dim = dim;
  }

  update(box, dt) {
    this.vel = Vector.add(this.vel, this.acc.mult(dt));
    this.pos = Vector.add(this.pos, this.vel.mult(dt));

    for (let i = 0; i < this.dim; i++) {
      if (this.pos.get(i) < this.radius && this.vel.get(i) < 0) {
        this.vel.flip(i);
      } 

      if (this.pos.get(i) > box[i] - this.radius && this.vel.get(i) > 0) {
        this.vel.flip(i);
      }
    }
  }

  static updateAll(hyperballs, box, dt) {
    hyperballs.forEach(hyperball =>
      hyperball.update(box, dt)
    );

    for (let i = 0; i < hyperballs.length; i++) {
      for (let j = i + 1; j < hyperballs.length; j++) {
        if (Hyperball.isColliding(hyperballs[i], hyperballs[j])) {
          Hyperball.collisionResponse(hyperballs[i], hyperballs[j]);
        }
      }
    }
  }

  crossSection(cuts) {
    let pos = this.pos.values.slice();
    let r = this.r;

    for (let kv of cuts) {
      let i = kv[0];
      let c = kv[1];
      r = (r**2 - (pos[i] - c)**2)**0.5;
      pos.splice(i, 1);

      if (isNaN(r)) {
        return null;
      }
    }
    return new Hyperball(r, new Vector(pos));
  }

  static isToward(h1, h2) {
    return Vector.dot(Vector.sub(s2, s1), Vector.sub(v2, v1)) < 0;
  }

  static isColliding(h1, h2) {
    return Vector.dist(h1.pos, h2.pos) < h1.r + h2.r;
  }

  static collisionResponse(h1, h2) {
    let m1 = h1.mass;
    let m2 = h2.mass;
    let s1 = h1.pos;
    let s2 = h2.pos;
    let v1 = h1.vel;
    let v2 = h2.vel;
    let lambda = -2 * m1 * m2 / (m1 + m2) * 
      Vector.dot(Vector.sub(s2, s1), Vector.sub(v2, v1)) /
      Vector.sub(s2, s1).norm**2;

    h1.vel = Vector.add(Vector.sub(s2, s1).mult(-lambda / m1), v1);
    h2.vel = Vector.add(Vector.sub(s2, s1).mult(lambda / m2), v2);
  }
}
{% endhighlight %}

### The User Interface

The website consists of some text with some buttons and some sliders.
The HTML is quite straight-forward:

{% highlight html %}
<html>
  <head>
    <title> N-Dimensional Collisions </title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.6.0/p5.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.6.0/addons/p5.dom.js"></script>
    <script src="vector.js"></script>
    <script src="hypersphere.js"></script>
    <script src="index.js"></script>
    <link rel="stylesheet" type="text/css" href="main.css">
  </head>
  <body>
    <h1>N-Dimensional Collisions</h1>
    <div>
      <p> Visible Dimension(s): </p>
      <button id="decrementDisplayDimButton">-</button>
      <span id="displayDimSpan"></span>
      <button id="incrementDisplayDimButton">+</button>
    </div>
    <div>
      <p> Hidden Dimension(s): </p>
      <button id="decrementExtraDimButton">-</button>
      <span id="extraDimSpan"></span>
      <button id="incrementExtraDimButton">+</button>
    </div>
    <p> Total Dimension(s): <span id="totalDimSpan"></span></p>
    <p> Hyperball(s): <span id="hyperballSpan"></span></p>
</script>
</body>
</html>
{% endhighlight %}

The first two scripts load the p5.js library. Then there's the files vector.js and
hyperball.js files we wrote above. The file index.js will be written in the 
next section. Finally there's the css file, which goes as follows:

{% highlight css %}
canvas {
        display: block;
        position: fixed; 
        top: 0;
        left: 0;
        z-index: -1;
}
p {
        margin: 0.25em 0 0.25em 0;
        font-size: 100%;
}
h1 {
        font-size: 120%;
}
body {
        color: white;
}
html {
        font: 60px arial, sans-serif;
}
div {
        display: flex;
        padding: 0;
        margin: 0;
        align-items: center;
}
button {
        opacity: 0.5;
        border-radius: 50%;
        font-size: 1em;
        font-family: monospace;
        width: 1.25em;
        height: 1.25em;
        background-color: blue;
        border: none;
        outline: none;
        margin: 0;
        padding: 0;
}
button:hover {
        opacity: 1;
}
button:disabled {
        opacity: 0.2;
        box-shadow: none;
}
button:active {
        background-color: darkblue; 
        transform: translateY(0.1em);
}
input[type=range] {
        -webkit-appearance: none;
        border-radius: 0.25rem;
        width: 50%;
        height: 0.5rem;
        background: #d3d3d3;
        outline: none;
        opacity: 0.5;
        -webkit-transition: .1s;
        transition: opacity .1s;
}
input[type=range]:hover {
        opacity: 1;
}
input[type=range]::-webkit-slider-thumb {
        -webkit-appearance: none; 
        border-radius: 50%;
        appearance: none;
        width: 1rem; 
        height: 1rem; 
        background: blue; 
        cursor: pointer; 
}
input[type=range]::-moz-range-thumb {
        border-radius: 50%;
        width: 1rem; 
        height: 1rem; 
        background: blue; 
        cursor: pointer; 
}
{% endhighlight %} 

### Animation

#### Non Overlapping Hyperballs

{% highlight html %}

{% include math.html %}
<p> hello </p>
{% endhighlight %}

#### Movement

## Final Words

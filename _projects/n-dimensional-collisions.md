---
layout: page
title: N-Dimensional Collisions
description: N-dimensional physics engine writen in JavaScript
 and rendered with p5.js
demo: https://symmetries.github.io/n-dimensional-collisions
---

{% include img-align.md %}
{% include math.html %}

![demo image](\images\n-dimensional-collisions\n-dimensional-collisions.png)
{: img-center}

## Description
This project consists of a simple physics engine in an arbitrary number
of dimensions. The engine itself is written in pure JavaScript.
In addition, p5.js was used for rendering.
A demo can be found [here]({{ page.demo }}).

## Tutorial

### The Vector Class
The first thing that we will need in order to make this project is to
create a `Vector` class.
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
* `pos`, a `Vector` denoting the position of the particle
* `vel`, a `Vector` denoting the velocity of the particle
* `acc`, a `Vector` denoting the acceleration of the particle
* `mass`, a scalar or number denoting the mass of the particle
* `r`, a scalar of number denoting the radius of the particle
* `dim`, a scalar or number denoting the dimension of the particle

We can also expect to have the following methods
* `update(box, dt)`
* `static updateAll(hyperballs, box, dt)`
* `isColliding(other)`
* `collisionResponse(other)`

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
\frac{m_1 \left \lVert \vec v_1 \right \rVert ^2}{2} + 
  \frac{m_2 \left \lVert \vec v_2 \right \rVert ^2}{2} &=
  \frac{m_1 \left \lVert \vec v_1^\prime \right \rVert ^2}{2} +
  \frac{m_2 \left \lVert \vec v_2^\prime \right \rVert ^2}{2} &\; (1)\\\\ 
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
  = (\vec v - \vec u) \cdot (\vec v + \vec u) \\) to rewrite (5).

\\[
\begin{aligned}
&m_1(\vec v_1^\prime - \vec v_1) \cdot 
  (\vec v_1^\prime + \vec v_1) \\\\ 
  &\qquad=-m_2 (\vec v_2^\prime - \vec v_2) \cdot
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
  \frac{-\lambda}{m_1} (\vec s_2 - \vec s_1) + 2\vec v_1 \right ) \\\\ 
&\qquad= \frac{-\lambda}{m_1} \\| \vec s_2 - \vec s_1 \\| ^2 + 
  2\vec v_1 \cdot (\vec s_1 - \vec s_1)
\end{aligned}
\\]

Substituting (9) onto the right side of (7)

\\[
\begin{aligned}
&(\vec s_1 - \vec s_1) \cdot \left (
  \frac{\lambda}{m_2} ( \vec s_2 - \vec s_1) + 2 \vec v_2  \right ) \\\\ 
&\qquad = \frac{\lambda}{m_2} \\| \vec s_2 - \vec s_1 \\|^2 +
  2\vec v_2 \cdot (\vec s_2 - \vec s_1)
\end{aligned}
\\]

Combining both sides

\\[
\begin{aligned}
&\frac{-\lambda}{m_1} \\| \vec s_2 - \vec s_1 \\| ^2 + 
  2\vec v_1 \cdot (\vec s_2 - \vec s_1) \\\\ 
&\qquad = \frac{\lambda}{m_2} \\| \vec s_2 - \vec s_1 \\|^2 +
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
  &=-2(\vec s_2 - \vec s_1) \cdot (\vec v_2 - \vec v_1) 
\end{aligned}
\\]

The value of \\( \lambda \\) is then

\\[
\lambda = \frac{-2m_1 m_2}{m_1 + m_2} 
  \frac {(\vec s_2 - \vec s_1) \cdot (\vec v_2 - \vec v_1)}
  {\\| \vec s_2 - \vec s_1 \\| ^2} 
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

#### The Code

We can finally write our `Hyperball` class.

{% highlight javascript %}
console.log("test");
{% endhighlight %}

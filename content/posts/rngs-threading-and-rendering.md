---
title: "RNGs, Threading, and Subtle Rendering Bugs"
date: 2020-02-02T14:40:04-08:00
draft: true
---

I've been working on writing a [renderer](https://www.github.com/afnanenayet/nib).
It started out with a lot of scaffolding, writing out interfaces for how
different components of the renderer should be defined, how they should
interact, figuring out how to handle serialization, etc. At first it was a
little boring because I couldn't see any results, so I wanted to get to an MVP.
I thought I'd create a scene similar to the one that Pete Shirley defines in
"Ray Tracing in One Weekend" (check it out if you want to write a raytracer,
it's a fun and very digestible project).

First off, let me show you the ground truth render. This is what I want my
image to look like. It was rendered with 1000 samples per pixel (spp) at a 1920
by 1080 resolution. All of the images shown in this article have the same
resolution.

{{< figure src="/images/rngs-threading-and-rendering/ground_truth.png" title="Ground truth render (8 threads, 1000 spp)" >}}

I thought creating this would be easy. Calculating a ray-sphere intersection
isn't terribly difficult, and I wrote a very basic ray tracing integrator. It
seemed like I had implemented the techniques correctly, but for some reason, my
picture looked like this:

{{< figure src="/images/rngs-threading-and-rendering/buggy_low_spp.png" title="Buggy render (8 threads, 100 spp)" >}}

Looks almost like a painting! It seemed like the light interactions were
generally in the right place. We see shadows in the right places, you can even
see that the light bouncing off of the spheres onto the adjacent spheres looks
correct too. It seems like there's color banding and it looks like the spheres
have imperfect surfaces. We know that these have to be perfect spheres because
the spheres are calculated analytically. Anything that looks less than perfect,
barring random noise from Monte Carlo sampling, means that some part of the
rendering pipeline is incorrect.

## Ray Tracing and Sampling

In order to understand what I did wrong, we need to understand the process for
creating an image with basic ray tracing. The principle behind ray tracing is
to create a realistic looking image by simulating light light interactions.
In real life, light travels from light sources and bounces off of surfaces and
interacts until the light ray eventually reaches our eye or a camera.

Ray tracing does this process backwards: it shoots out rays from the eye or
virtual camera and traces how these rays bounce off of objects into other
objects or lights, taking into account the materials and colors. There are
mathematical models for simulating all sorts of surfaces (such as matte,
glossy, and reflective surfaces), which provide a model that dictates how light
rays will travel through a scene.

_As an aside: ray tracing is "backwards" for efficiency. If we were to trace
rays from light sources we'd waste a lot of CPU cycles as light paths that
don't hit the camera don't contribute to the image._

Here's some pseudocode outlining how we can recursively calculate the color
value of some outgoing ray:

```python
def color(ray, depth):
    """ Calculate a color value given an initial light ray
    """
    if depth > DEPTH_LIMIT:
        return [0, 0, 0]

    initial_color, outgoing_ray = intersection(ray)

    # Component-wise multiplication
    return initial_color.comp_mul(color(outgoing_ray, depth + 1))
```

Calculating `initial_color` is pretty intuitive, it's a function of the
material that the ray hits. If you hit a perfectly red object, you attenuate
any accumulated light with `[1, 0, 0]` (assuming that the array is `[R, G, B]`).
`outgoing_ray` is calculated as a function of the bidirectional scattering
distribution function (BSDF), which is a model for how light is scattered
outwards upon hitting some material. For example, a perfectly reflective
surface will simply reflect the incoming ray around the normal vector.

{{< figure src="/images/rngs-threading-and-rendering/sphere_mirror_ray_intersection.svg" height="300px">}}

It turns out you can make reflective surfaces have "fuzzier" reflections by
perturbing the outgoing vector. The surface looks less and less reflective as
the outgoing ray strays further from the reflection of the incoming ray.

{{< figure src="/images/rngs-threading-and-rendering/partially_reflective.svg" height="300px">}}

A perfectly diffuse (or matte) surface calculates the outgoing ray by taking
the reflected incoming ray and sampling the entire sphere around the
reflection, so rays are perfectly scattered outwards from the interaction.

You may have also noticed that I said we *sample* the sphere. Ray tracing is
not an analytic process, it performs a Monte Carlo integration of the function
that represents the color for each pixel. Monte Carlo integration is a
stochastic process, and there is a lot of randomness involved. For example,
when generating the initial rays from the virtual camera, we actually generate
random samples from the camera outwards. We take the results of each of these
samples and average them together to get the color value for a particular
pixel. Casting more rays per pixel results in a more accurate and smoother
output image. We don't just sample the camera, we sample the outgoing rays from
diffuse surfaces as well.

## Now, We Debug

The first thing I did was write a bunch of tests, mostly to check that my match
was correct. I wrote unit tests for the ray-sphere intersection, ensure that
for each casted ray, we were actually returning the closest *valid* collision,
and so on. I found some small bugs, but after fixing them, I still got the
same image, so no dice.

I had recently parallelized the renderer, and on a whim, I decided to run it
with one thread to see if I had introduced some sort of concurrency bug.
Running it with one thread yielded this image:

{{< figure src="/images/rngs-threading-and-rendering/one_thread_low_spp.png" title="Buggy render (1 thread, 100 spp)" >}}

Wait, that's correct. How did multithreading break the rendering pipeline?
Each pixel is calculated independently, and the methods don't modify any data
that's shared across threads (this is a requirement for the parallelization
library I'm using, Rayon), so we know it's not a data race or some other weird
memory access violation.

I had a hunch, though. If you look at the buggy image, you can see that there
almost appears to be color banding. There's an odd "clumping" of colors along
the face of each surface. Knowing that the outgoing rays from the surfaces of
these spheres is dependent on some random sampling, I suspected that there was
an issue with the RNG or how I was sampling, and that I wasn't actually
uniformly sampling something or the RNG wasn't producing random numbers in a
uniform distribution. If the random outgoing rays weren't actually that random,
it would explain why the surfaces of the sphere didn't look smooth -- there
would be rays pointing in the same direction in close proximity to each other,
causing regions of the sphere to have the same color and lighting values.

To further confirm that this was an issue with random number generation, I took
the multithreaded implementation and bumped the sampling resolution to 1000 spp:

{{< figure src="/images/rngs-threading-and-rendering/buggy_high_spp.png" title="Buggy render (8 threads, 1000 spp)" >}}

It looks way better with ten times the samples. It was very clear to me at this
point that there was a sampling issue (remember that more samples leads us
closer to the correct value in Monte Carlo integration), and we knew it was
affected by parallelization.

Knowing that going from eight threads to a single thread fixed the problem made
me suspect the random number generation itself. Going back to my renderer's
implementation, I realized that each thread has a copy of the random sampler
*with the same seed*, so each thread was actually generating identical numbers
in the same order. As a result, the overall distribution of numbers in the
rendering pipeline wasn't uniformly random. Lucky for me, it's a really easy
problem to fix: just use a different seed for each thread.

{{< figure src="/images/rngs-threading-and-rendering/correct_low_spp.png" title="Correct render (8 threads, 100 spp)" >}}

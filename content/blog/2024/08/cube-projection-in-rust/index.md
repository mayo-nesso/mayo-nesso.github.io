---
title: "[LiP] The Magic of 3D: Projecting a Cube in Rust"
weight: 1
tags: ["Rust", "Matrix", "3D", "LiP"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://mayonesso.com/blog/2024/cube-projection-in-rust"
disableHLJS: true # to disable highlightjs
disableShare: true
hideSummary: false
summary: "Render a 3D cube using ASCII characters in the terminal"
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: false
UseHugoToc: true
cover:
    image: "cover.webp"
    relative: false
    hidden: false
---

## • Introduction

Hey there! Today, we're going to dive into something that might make your head spin faster than a dusty CPU fan on a hot summer day: 3D projection... in Rust.

Yep, I know that sounds crazy, but bear with me.</br>
This rudimentary version is just a naive approximation of what game engines do nowadays.

I was heavily inspired by this video [ASMR Programming - Spinning Cube - No Talking](https://www.youtube.com/watch?v=p09i_hoFdd0), so thanks to its author!

![it's alive!](imgs/0_its_alive.gif#center)

---

## • TLDR

[Source code](https://github.com/mayo-nesso/spinning_cube)

---

## • The General Idea

First, let's set the plan.</br>
The idea is to render a 3D cube on a 2D screen.
</br>And to do that, we are going to make use of that thing called Math, and, to make everything a little bit more complicated, we are going to write a program in Rust to achieve our mission.

So, it's not your one-line "Hello, World!" program, but it's not rocket science either. Well, actually, it kind of is – but the fun kind, where no one gets blown up if we mess up a calculation.

What are we going to do is:

i.- Take each face of the cube</br>
ii.- Apply math to calculate the position in the 3D space, taking into consideration rotations on the X, Y, or Z axis</br>
iii.- Draw the points that are closer to the camera, applying some projection to make things bigger or smaller accordingly

---

## • The Math Behind the Madness

Now, before you run away screaming "Math! My old nemesis!", let me tell you something, we are not trying to publish a post-doc thesis.

All the formulas that we are going to use look scarier than they really are, so even if math is not your thing, let's give them a chance!

In this [Wikipedia article on the Rotation matrix](https://en.wikipedia.org/wiki/Rotation_matrix), they explain to us that:

> ...a rotation matrix is a transformation matrix that is used to perform a rotation in Euclidean space.

This could be performed in two dimensions, but that is not enough for us, so; fast your seat belt; and prepare to travel to the next level; The 3D world!

![The 3d world!](imgs/1_3Dv2.jpg#center)

So [if we jump to this part of the article](https://en.wikipedia.org/wiki/Rotation_matrix#General_3D_rotations)  we have:

![One of many options](imgs/2_rotation_matrix_formula.png#center)

This is one of many ways to use math to apply the required transformations (yep, another and arguably better are [Quaternions](https://en.wikipedia.org/wiki/Quaternion)!).

If we multiply this little monster by a `[i, j k]` vector, we have;

![Rotation Matrix](imgs/3_rotation_matrix.png#center)

which gives us;

![Rotation Matrix Solution](imgs/4_solution.png#center)

And that's all, that is the farther that we are going to go, in terms of esoteric math.</br>
**That is our rotation matrix!**

This, in a code-friendly style, translates each row to the `calculate_x`, `calculate_y`, and `calculate_z` functions in our `rendering.rs` file.

```rust
fn calculate_x(
  i: f32, 
  j: f32, 
  k: f32, 
  alpha: f32,
  beta: f32,
  gamma: f32,
) -> f32 {
  let sin_a = alpha.to_radians().sin();
  let cos_a = alpha.to_radians().cos();
  let sin_b = beta.to_radians().sin();
  let cos_b = beta.to_radians().cos();
  let sin_c = gamma.to_radians().sin();
  let cos_c = gamma.to_radians().cos();

  i * cos_a * cos_b // < --- I re arranged the terms a little bit, first `i` components, then `j`, then `k`...
    + j * cos_a * sin_b * sin_c
    - j * sin_a * cos_c
    + k * cos_a * sin_b * cos_c  
    + k * sin_a * sin_c
}

...
```

### Why So Trigonometric?

You might be wondering, "Why all the sines and cosines? Did the developer have a particular fondness for waves?" Well, not exactly. Sines and cosines are like `The Good, the Bad and the Ugly` of rotation in mathematics (in other words, a classic). They allow us to describe circular motion, which is exactly what we need for rotation.

Let's break it down:

i. `alpha`, `beta`, and `gamma` are our rotation angles around the z, y, and x axes respectively (why this inverse order? Well, remember that wikipedia article? Different transformations can be used, and this one uses `alpha` as yaw, `beta` as pitch, and `gamma` as roll [angles](https://simple.wikipedia.org/wiki/Pitch,_yaw,_and_roll))

ii. We convert these angles to radians (because computers prefer radians to degrees, they're quirky like that)

iii. We calculate the sine and cosine of each angle

iv. We then combine these values in a specific way to rotate our point

---

## • From 3D to 2D: The Projection

![A question of perspective](imgs/5_a_question_of_perspective.jpg#center)

Now that we've rotated our point, we need to project it onto our 2D screen. This is where the `calculate_for_surface` function comes in:

```rust
fn calculate_for_surface(
  cube_x: f32,
  cube_y: f32,
  cube_z: f32,
  ch: char,
  z_buffer: &mut [f32; CANVAS_WIDTH * CANVAS_HEIGHT],
  buffer: &mut [char; CANVAS_WIDTH * CANVAS_HEIGHT],
  alpha: f32,
  beta: f32,
  gamma: f32,
  distance_from_camera: f32,
  projection_scale: f32,
) {
  // Apply rotation transformations to the cube coordinates
  let x = calculate_x(cube_x, cube_y, cube_z, alpha, beta, gamma);
  let y = calculate_y(cube_x, cube_y, cube_z, alpha, beta, gamma);
  let z = calculate_z(cube_x, cube_y, cube_z, alpha, beta, gamma);
  // Adjust z-coordinate based on camera distance
  let z = z - distance_from_camera;
  
  // Calculate the inverse of z (ooz: one over z)
  //
  // In our coordinate system:
  // i.- z = 0 is at the origin
  // ii.- Positive z values are closer to the camera
  // iii.- Negative z values are further from the camera
  //
  // After subtracting distance_from_camera, all z values become negative
  // Smaller negative z values are closer to the camera
  let ooz = - 1.0 / z;  
  // So now, regarding ooz:
  // i.- All values are positive
  // ii.- Larger values indicate closer proximity to the camera (useful for z-buffer comparisons)
  
  // Convert 3D coordinates to 2D screen space
  // Note how we use here ooz to 'shrink' or 'expand' the projection
  // xp = ... + x * ...: Positive x values move the point to the right, so we add to the center of the screen.
  let xp = (CANVAS_WIDTH as f32 /2.0 + x * ooz * projection_scale * ASPECT_RATIO) as isize;
  // yp = ... - y * ...: Positive y values (in 3D space) move the point up, 
  // but because screen space has the y-axis increasing downward, we subtract to move the point upward on the screen.
  let yp = (CANVAS_HEIGHT as f32 / 2.0 - y * ooz * projection_scale) as isize;


  // Check if (xp,yp) point is within the canvas boundaries
  if xp < 0 || (xp as usize) >= CANVAS_WIDTH {
      return;
  }
  if yp < 0 || (yp as usize) >= CANVAS_HEIGHT {
      return;
  }
  
  // Calculate the buffer index for the current point
  let idx = xp + yp * CANVAS_WIDTH as isize;
  let idx = idx as usize;
  
  // Update the z-buffer and character buffer only if this point is closer to the camera
  if ooz > z_buffer[idx] {
      z_buffer[idx] = ooz;
      buffer[idx] = ch;
  }
}
```

This function is where the magic happens. It takes our 3D point, rotates it, and then projects it onto our 2D canvas.

### The Secret Sauce: Perspective Division

The key to this projection is the line `let ooz = - 1.0 / z;`. This is called perspective division, and it's what gives our projection that 3D feel.

Here's why it works:

i. Objects further away appear smaller.</br>
ii. In our 3D space representation, smaller z values mean the point is further away (and since our origin is `0.0`, the more negative a value is, is further away). And since we push away our points even more with `- distance_from_camera` all our points should be expected to be negative.</br>
iii. By multiplying by `-1.0` and dividing by `z`, we have a variable `ooz` that has positive values. The larger the closer to the camera, the bigger that should be on screen.</br>
iv. So, points with smaller `ooz` values (further away) have a smaller effect on the final x and y coordinates.

</br>
It's like squishing the entire 3D world onto your flat screen, with the added bonus of making far-away things look... well, far away.

---

## • Putting It All Together

Now that we understand the individual pieces, let's look at how they all fit together in the `draw_cube` function:

Here we will 'draw' each face of the cube, point by point.

```rust
pub fn draw_cube(
  z_buffer: &mut [f32; CANVAS_WIDTH * CANVAS_HEIGHT],
  buffer: &mut [char; CANVAS_WIDTH * CANVAS_HEIGHT],
  params: &CubeParameters,
) {
  let mut cube_x = -1.0 * HALF_CUBE_WIDTH as f32;
  while cube_x < HALF_CUBE_WIDTH as f32 {
    let mut cube_y = -1.0 * HALF_CUBE_WIDTH as f32;
    while cube_y < HALF_CUBE_WIDTH as f32 {
      // Axis following Rigth-Hand rule
      //      y
      //      |
      //      |
      //      |_ _ _ _ x
      //     /
      //    /
      //   z
      //

      //  Plane K; Back side, We update X and Y, and keep Z constant 
      //        ______
      //      /|  K   |
      //     / |      |
      //    |  |______|
      //    | /      /
      //    |/______/
      //
      let x_value = cube_x;
      let y_value = cube_y;
      let z_value = -1.0 * (HALF_CUBE_WIDTH as f32);
      calculate_for_surface(
        x_value, y_value, z_value,
        'K',
        z_buffer, buffer,
        params.alpha, params.beta, params.gamma,
        params.distance_from_camera, params.projection_scale);

      // Plane F; Front side, We update X and Y, and keep Z constant 
      //       ______
      //     /      /|
      //    /______/ |
      //    |      | |
      //    |  F   | /
      //    |______|/
      //
      let x_value = cube_x;
      let y_value = cube_y;
      let z_value = 1.0 * (HALF_CUBE_WIDTH as f32);
      calculate_for_surface(
        x_value, y_value, z_value,
        'F',
        z_buffer, buffer,
        params.alpha, params.beta, params.gamma,
        params.distance_from_camera, params.projection_scale);


      // ... draw other faces ...

      cube_y += params.resolution_step;
    }
    cube_x += params.resolution_step;
  }
}
```

This function is like a meticulous artist, painting our cube one point at a time. It loops over the x and y coordinates of each face, calling `calculate_for_surface` for each point.</br>
The result? A beautiful 3D cube rendered on our 2D screen.

Check how we are using letters to represent each face of the cube:
> F = Front, K = Back,</br>
> L = Left, R = Right,</br>
> T = Top, B = Bottom.</br>

![Cube](imgs/6_faces.png#center)

---

## • The Z-Buffer: Our Unsung Hero

You might have noticed we're passing around something called a `z_buffer`. This isn't just some fancy term we made up to sound smart (although it does sound pretty cool). The `z-buffer` is crucial for determining which parts of our cube are visible.

In the `calculate_for_surface` function, we have this bit of code:

```rust
// Update the z-buffer and character buffer only if this point is closer to the camera
if ooz > z_buffer[idx] {
    z_buffer[idx] = ooz;
    buffer[idx] = ch;
}
```

This is checking if the current point is closer to the camera than what we've drawn so far. If it is, we update the `z-buffer` and draw the point.

This ensures that closer parts of the cube properly obscure parts that are further away. It's like giving our rendering engine depth perception!

And yes, if you were wondering, the concept of [Z-fighting](https://en.wikipedia.org/wiki/Z-fighting) had to come from somewhere!

---

## • Conclusion: From Math to Magic

So there you have it, folks. We've taken a journey from dry mathematical formulas to a living, breathing 3D cube rendered in ASCII characters. We've rotated points in 3D space, projected them onto a 2D plane, and even given our rendering engine the ability to understand depth.

The next time you're playing your favorite 3D game, spare a thought for the math happening behind the scenes. It's not just pushing pixels – it's a beautiful dance of trigonometry, linear algebra, and clever programming.

Remember, every masterpiece of 3D graphics you see, from the latest AAA game to that cool 3D chart in your PowerPoint presentation, is built on these fundamental principles. It's just that most of the time, the math is hidden away in optimized graphics libraries and powerful GPUs.

But now you know the secret. You've peeked behind the curtain and seen the wizard at work.

Until next time, keep your code clean, your algorithms efficient, and your cubes rotating!

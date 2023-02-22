---
title: Player Setup and Animation
category: Zelda-like 4.0
---
## Scene setup
#### CharacterBody2D and CollisionShape2D

Our root node is going to be a CharacterBody2D. We will use this type for any object that can move and has collision.

By default it's set to "Grounded" for platformers. Set it to "Floating" for top-down perspective movement.

Add a CollisionShape2D as a child node. We will also give this a Capsule shape resource and set it to 5 px * 12 px. ###

#### AnimatedSprite2D

The last node we'll add is an AnimatedSprite2D. Give it a new Spriteframes resource.

First we'll create a new WalkDown animation, add two frames, loop it, and set it to play at 10 FPS. Do the same for WalkSide and WalkUp.

I am using one WalkSide for left and right. We'll flip the sprite horizontally if the player is moving left.

Now we'll rename it "Player" and save the scene as player.tscn

## Script

Now we are going to add a script for the player. Leave the template blank and save it as "player.gd"

First we'll add our variables.

```python
extends CharacterBody2D

const SPEED = 70

var input_direction
var sprite_direction

@onready var sprite = $AnimatedSprite2D
```

Our `SPEED` constant will be how many pixels per second the player will use.

The `input_direction` variable is going to be a Vector2 mapped to the arrow keys. We will complete this one in a moment.

The `sprite_direction` will be a string of which direction the player is facing. I will explain more in the *Animation* section.

The `sprite` onready variable is a reference to our AnimatedSprite2D node.

#### Input Direction

Let's create a getter function for `input_direction`. It is going to check for keyboard inputs and return a Vector2.

We'll take the combined presses of "ui_left" and "ui_right" and turn it into a local `x` variable. If the left arrow key is pressed `x` will equal -1, if right is pressed it's 1. They are added together so if both are pressed it cancels to 0.

Do the same for `y`.

We can now use these values in a Vector2. We normalize it so the player moves at the same speed diagonally. Finally we can return our `input_direction`.

```python
func _get_input_direction():
	var x = -int(Input.is_action_pressed("ui_left")) + int(Input.is_action_pressed("ui_right"))
	var y = -int(Input.is_action_pressed("ui_up")) + int(Input.is_action_pressed("ui_down"))
	input_direction = Vector2(x,y).normalized()
	return input_direction
```

We can add the *get* keyword to our `input_direction` definition and the function will update its value any time we ask for it.

```python
var input_direction: get = _get_input_direction
```

#### Movement

We have what we need to move the player now. Let's go back to `_physics_process` and make use of `input_direction`.

```python
func _physics_process(_delta):
	velocity = input_direction * SPEED
	move_and_slide()
```

Very simple. We are simply setting velocity to our `input_direction`, multiplying it by our `SPEED` constant, then moving the body.

We can run the scene now with F6 and the player will move with the arrow keys. Holding multiple directions should cancel out.

#### Sprite Direction

The last couple things we need to do is use the appropriate animation depending on the player's direction and start or stop the animation if the player is moving.

We're going to create another getter function, this time for our `sprite_direction` variable. I am going to put it right under `_get_input_direction`.

```python
func _get_sprite_direction():
	match input_direction:
		Vector2.LEFT:
			sprite_direction = "Left"
		Vector2.RIGHT:
			sprite_direction = "Right"
		Vector2.UP:
			sprite_direction = "Up"
		Vector2.DOWN:
			sprite_direction = "Down"
	return sprite_direction
```

Simple enough. If our `input_direction` is one of those four values it sets `sprite_direction` and returns it. It will not do anything special if the player is moving diagonally or is still.

Now let's define this as `sprite_direction`'s getter function.

```python
var sprite_direction = "Down": get = _get_sprite_direction
```

#### Animation

Let's create a function that accepts an animation parameter and will set the correct animation according to our `sprite_direction`.

```python
func set_animation(animation):
	var direction = "Side" if sprite_direction in ["Left", "Right"] else sprite_direction
	sprite.play(animation + direction)
	sprite.flip_h = (sprite_direction == "Left")
```

The first line of the function's body defines a new direction variable. If `sprite_direction` is either "Left" or "Right" it will set `direction` to "Side". If it is "Up" or "Down" it can remain the same.

The second line plays the animation. It takes our `animation` parameter and concatenates it with our new `direction` variable. If we are, say, pressing up to walk, it will play "WalkUp".

The final line flips our sprite if `sprite_direction` is "Left".

Now we just need some criteria for it to play and stop.

```python
if velocity:
		set_animation("Walk")
else:
		set_animation("Walk")
		sprite.stop()
```

`if velocity:` returns true if the player is moving at all. We set our animation to "Walk" if so. Otherwise we do the same thing but stop the sprite. Any new animation set just needs to be set up the same way as "Walk" and it can make use of the `set_animation` function.

We can try out the scene now. The player can move in 8 directions and will only change animation when moving orthogonally.
---
title: Player Movement and Animation
category: Zelda-like 4.0
---
## Introduction

#### Project Setup

First let's do some quick project setup. Since we'll be using 2D pixel art, we need to turn off the linear filtering on sprites. In **General>Rendering>Textures**, set the default texture filter from "Linear" to "Nearest".

![Project settings](/_images/zelda-like-1_project_settings.png "Project settings")

Now let's create an empty scene with a camera. The root will be a *Node2D* that we will name "Main". Add a *Camera2D* node and set its *Zoom* parameters to 4. Save it as "res://main.tscn".

![Main scene](/_images/zelda-like-1_main_scene.png "Main scene")

## Player Scene

#### CharacterBody2D and CollisionShape2D

Create a new scene. Our root node is going to be a *CharacterBody2D*. We will use this type for any object that can move and has collision. Name the root "Player" and save the scene as "res://player/player.tscn".

By default the *Motion Mode* is set to "Grounded" for platformers. Set it to "Floating" for top-down perspective movement.

Add a *CollisionShape2D* as a child node. We will give it a *CapsuleShape2D* resource and set the *Radius* and *Height* to 5 px and 12 px.

![Collision shape](/_images/zelda-like-1_collision_shape.png "Collision shape")

#### AnimatedSprite2D

The last node we'll add is an *AnimatedSprite2D*. Give it a new *SpriteFrames* resource.

![Player sprite sheet](/_images/zelda-like-1_player_spritesheet.png "Player sprite sheet")

First we'll create a new "WalkDown" animation, add two frames, loop it, and set it to play at 10 FPS. Do the same for "WalkSide" and "WalkUp".

I am using one "WalkSide" for left and right. We'll flip the sprite horizontally if the player is moving left.

## Script

Now we are going to add a script for the player. Leave the template blank and save it as "player.gd" in its default directory.

First we'll add our variables.

```gdscript
extends CharacterBody2D

const SPEED = 70

var input_direction
var sprite_direction

@onready var sprite = $AnimatedSprite2D
```

Our `SPEED` constant will be how many pixels per second the player will move.

The `input_direction` variable is going to be a Vector2 mapped to the arrow keys. We will complete this one in a moment.

The `sprite_direction` will be a string of which direction the player is facing. I will explain more in the *Animation* section.

The `sprite` onready variable is a reference to our *AnimatedSprite2D* node.

#### Input Direction

Let's create a getter function for `input_direction`. It is going to check for keyboard inputs and return a Vector2.

We'll take the combined presses of "ui_left" and "ui_right" and turn it into a local `x` variable. If the left arrow key is pressed `x` will equal -1, if right is pressed it will be 1. They are added together so if both are pressed it cancels to 0.

Do the same for `y`.

We can now use these values in a Vector2. We normalize it so the player moves at the same speed diagonally. Finally we return our `input_direction`.

```gdscript
func _get_input_direction():
	var x = -int(Input.is_action_pressed("ui_left")) + int(Input.is_action_pressed("ui_right"))
	var y = -int(Input.is_action_pressed("ui_up")) + int(Input.is_action_pressed("ui_down"))
	input_direction = Vector2(x,y).normalized()
	return input_direction
```

We can add the `get` keyword to our `input_direction` definition and the function will update its value any time we ask for it.

```gdscript
var input_direction: get = _get_input_direction
```

#### Movement

We have what we need to move the player now. Let's go back to `_physics_process` and make use of `input_direction`.

```gdscript
func _physics_process(_delta):
	velocity = input_direction * SPEED
	move_and_slide()
```

We are simply setting the body's velocity to our `input_direction`, multiplying it by our `SPEED` constant, and finally moving it. We don't even have to call the function in `_physics_process`; it is called implicitly through the getter declaration.

Add the player to the main scene we created earlier. We can run the scene now with F5 and the player will move with the arrow keys. Holding multiple directions should cancel each other out.

#### Sprite Direction

The last couple things we need to do is set the appropriate animation depending on the player's direction and start or stop the animation if the player is moving.

We're going to create another getter function, this time for our `sprite_direction` variable. I am putting this right under `_get_input_direction`.

```gdscript
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

```gdscript
  var sprite_direction = "Down": get = _get_sprite_direction
```

#### Animation

Let's create a function that accepts an animation parameter and will set the correct animation according to our `sprite_direction`.

```gdscript
func set_animation(animation):
	var direction = "Side" if sprite_direction in ["Left", "Right"] else sprite_direction
	sprite.play(animation + direction)
	sprite.flip_h = (sprite_direction == "Left")
```

The first line of the function's body defines a new direction variable. If `sprite_direction` is either "Left" or "Right" it will set `direction` to "Side". If it is "Up" or "Down" it can remain the same.

The second line plays the animation. It takes our `animation` parameter and concatenates it with our new `direction` variable. If we are, say, pressing up to walk, it will play "WalkUp".

The final line flips our sprite if `sprite_direction` is "Left".

Now we just need some criteria for it to play and stop.

```gdscript
  if velocity:
    set_animation("Walk")
  else:
	  set_animation("Walk")
	  sprite.stop()
```

`if velocity:` returns true if the player is moving at all. We set our animation to "Walk" if so. Otherwise we do the same thing but stop the sprite. Any new animation set just needs to be set up the same way as "Walk" and it can make use of the `set_animation` function.

We can try out the scene now. The player can move in 8 directions and will only change animation when moving orthogonally.

## Conclusion

Here is the full "player.gd" script:

```gdscript
extends CharacterBody2D

const SPEED = 70

var input_direction: get = _get_input_direction 
var sprite_direction = "Down": get = _get_sprite_direction

@onready var sprite = $AnimatedSprite2D


func _physics_process(_delta):
	velocity = input_direction * SPEED
	move_and_slide()
	
	if velocity:
		set_animation("Walk")
	else:
		set_animation("Walk")
		sprite.stop()


func set_animation(animation):
	var direction = "Side" if sprite_direction in ["Left", "Right"] else sprite_direction
	sprite.play(animation + direction)
	sprite.flip_h = (sprite_direction == "Left")


func _get_input_direction():
	var x = -int(Input.is_action_pressed("ui_left")) + int(Input.is_action_pressed("ui_right"))
	var y = -int(Input.is_action_pressed("ui_up")) + int(Input.is_action_pressed("ui_down"))
	input_direction = Vector2(x,y).normalized()
	return input_direction


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


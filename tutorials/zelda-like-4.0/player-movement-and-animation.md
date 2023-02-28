---
title: Player Movement and Animation
category: Zelda-like 4.0
---
## Introduction

#### Project Setup

First let's do some quick project setup. Since we'll be using 2D pixel art, we need to turn off the linear filtering on sprites. In **General>Rendering>Textures**, set the default texture filter from "Linear" to "Nearest".

![Filter](/_images/zelda-like-1_filter.png "Filter")

Now let's create an empty scene with a camera. The root will be a *Node2D* that we will name "Main". Add a *Camera2D* node and set its *Zoom* parameters to 4. Save it as "res://main.tscn".

![Main scene](/_images/zelda-like-1_main_scene.png "Main scene")

## Player Scene

#### CharacterBody2D and CollisionShape2D

Create a new scene. Our root node is going to be a *CharacterBody2D*. We will use this type for any object that can move and has collision. Name the root "Player" and save the scene as "res://player/player.tscn".

By default the *Motion Mode* is set to "Grounded" for platformers. Set it to "Floating" for top-down perspective movement.

Add a *CollisionShape2D* as a child node. We will give it a *CapsuleShape2D* resource and set the *Radius* and *Height* to 5 px and 12 px.

![Collision shape](/_images/zelda-like-1_collision_shape.png "Collision shape")

#### AnimatedSprite2D

The last node we'll add is an *AnimatedSprite2D*. Expand the *Animation* tab in the inspector and give it a new *SpriteFrames* resource.

Download this image and save it as "res://player/player.png".

![Player sprite sheet](/_images/zelda-like-1_player_spritesheet.png "Player sprite sheet")

Rename the "default" animation to "WalkDown". Set the FPS to 10. Press the button labeled *Add frames from sprite sheet (Ctrl+Shift+O)* and select the player sprite sheet file. Set *Horizontal* and *Vertical* to 6 and 6. Press *Add 2 Frame(s)*.

![Select frames](/_images/zelda-like-1_select_frames.png "Select frames")

Follow the same steps for the "WalkSide" and "WalkUp" animations. We will use "WalkSide" for both left and right. We'll flip the sprite horizontally when the player is moving left. Select "WalkDown" before moving on to set it as the default animation.

![Animated sprite](/_images/zelda-like-1_animated_sprite.png "Animated sprite")

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

The `input_direction` variable is going to be a Vector2 mapped to the arrow keys.

The `sprite_direction` will be a string of which direction the player is facing.

The `sprite` onready variable is a reference to our *AnimatedSprite2D* node.

#### Input Direction

Let's create a getter function for `input_direction`. It is going to check for keyboard inputs and return a Vector2. Name it `_get_input_direction`.

Let's create a local `x` variable and set it to the combined presses of "ui_left" and "ui_right". We will convert them from bool to int and negate "ui_left". If the left arrow key is pressed `x` will be -1, if right is pressed it will be 1. They are then added together; if both keys are pressed `x` cancels to 0.

Do the same with a new `y` variable and use "ui_up" and "ui_down".

Set `input_direction` to a normalized Vector2 composed of `x` and `y`. Finally we will return `input_direction`.

```gdscript
func _get_input_direction():
	var x = -int(Input.is_action_pressed("ui_left")) + int(Input.is_action_pressed("ui_right"))
	var y = -int(Input.is_action_pressed("ui_up")) + int(Input.is_action_pressed("ui_down"))
	input_direction = Vector2(x,y).normalized()
	return input_direction
```

Add the `get` keyword to our `input_direction` definition and the function will update its value any time we ask for it.

```gdscript
var input_direction: get = _get_input_direction
var sprite_direction
```

#### Movement

We have what we need to move the body now. Let's create the `_physics_process` above our other function and make use of `input_direction`. Set `velocity` to `input_direction * SPEED` and call `move_and_slide`. `velocity` is a built-in property of *CharacterBody2D* that is automatically used and updated when the node moves.

```gdscript
func _physics_process(_delta):
	velocity = input_direction * SPEED
	move_and_slide()
```

Add an instance of our player scene to the main scene we created earlier. We can run the scene now with F5 and the player will move with the arrow keys. Holding multiple directions should cancel each other out.

#### Sprite Direction

Now we need to set the appropriate animation depending on the player's direction and start or stop the animation if the player is moving.

We're going to create another getter function, this time for our `sprite_direction` variable. I am putting this right under `_get_input_direction`. We'll use a match statement that will take an orthogonal direction and set `sprite_direction` to a corresponding string before returning it.

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

Now let's define this as `sprite_direction`'s getter function.

```gdscript
var input_direction: get = _get_input_direction
var sprite_direction = "Down": get = _get_sprite_direction
```

#### Animation

Create a `set_animation` function that accepts an `animation` parameter. This will take an animation set such as "Walk" or later "Push" or "Carry" and play the correct animation depending on the player's direction.

Create a `direction` variable and set it to "Side" if `sprite_direction` is either "Left" or "Right". If it is "Up" or "Down" then `direction` can be the same. We use a [ternary-if expression]((https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html#if-else-elif) to accomplish this in a single line.

Next have our `sprite` node play `animation + direction`. We can use the `+` operator on two strings to concatenate them. If the up arrow key is being pressed it will play "WalkUp".

Finally we will set the `flip_h` property in `sprite` to the expression `sprite_direction == "Left"`.

```gdscript
func set_animation(animation):
	var direction = "Side" if sprite_direction in ["Left", "Right"] else sprite_direction
	sprite.play(animation + direction)
	sprite.flip_h = (sprite_direction == "Left")
```

Now we just need some criteria for the sprite to play or stop. When `velocity` is true (i.e. not equal to (0,0)) we will set the animation to "Walk". Otherwise, we'll do the same and also stop the sprite.

```gdscript
  if velocity:
    set_animation("Walk")
  else:
	  set_animation("Walk")
	  sprite.stop()
```

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


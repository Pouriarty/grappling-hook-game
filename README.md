# grappling-hook-game
2d grappling hook


The notable components the Player GameObject has right now are a capsule collider and a rigidbody which allow it to interact with physical objects in the level. There’s also a simple movement script (PlayerMovement) attached that lets your slippery character slide along the ground and perform basic jumps.

Click the Play button in the editor to start the game and try out the controls to see how they feel. A or D will move you left or right, and space will perform jumps. Be careful not to slip and fall off the rocks or you’ll die
Creating the Hooks and Rope

A grappling hooks system sounds fairly simple at first, but there are a number of things you’ll need in order to make it work well. Here are some of the main requirements for a 2D grappling hook mechanic:

    A Line Renderer which will show the rope. When the rope wraps around things, you can add more segments to the line renderer and position the vertices appropriately around the edges the rope wraps.
    A DistanceJoint2D. This can be used to attach to the grappling hook’s current anchor point, and lets the slug swing. It’ll also allow for configuration of the distance, which can be used to rappel up and down the rope.
    A child GameObject with a RigidBody2D that can be moved around depending on the current location of the hook’s anchor point. This will essentially be the rope hinge / anchor point.
    A raycast for firing the hook and attaching to objects.

Select Player in the Hierarchy and add a new child GameObject to the Player named RopeHingeAnchor. This GameObject will be used to position the hinge / anchor point of the grappling hook wherever it should be during gameplay.

Add a SpriteRenderer and RigidBody2D component to RopeHingeAnchor.

On the SpriteRenderer, set the Sprite property to use UISprite and change the Order in Layer to 2. Disable the component by unchecking the box next to its name.

On the RigidBody2D component, set the Body Type property to Kinematic. This point will not move around with the physics engine but by code

Set the layer to Rope and set the X and Y scale values to 4 on the Transform component
Select Player again and attach a new DistanceJoint2D component.

Drag and drop the RopeHingeAnchor from the Hierarchy onto the Connected Rigid Body property on the DistanceJoint2D component and disable Auto Configure Distance
Create a new C# script called RopeSystem in the Scripts project folder and open it with your code editor.

Remove the Update method.
At the top of the script, inside the RopeSystem class declaration, add the following variables as well as an Awake() method, and a new Update method
Taking each section in turn:

    You’ll use these variables to keep track of the different components the RopeSystem script will interact with.
    The Awake method will run when the game starts and disables the ropeJoint (DistanceJoint2D component). It'll also set playerPosition to the current position of the Player.
    This is the most important part of your main Update() loop. First, you capture the world position of the mouse cursor using the camera's ScreenToWorldPoint method. You then calculate the facing direction by subtracting the player's position from the mouse position in the world. You then use this to create aimAngle, which is a representation of the aiming angle of the mouse cursor. The value is kept positive in the if-statement.
    The aimDirection is a rotation for later use. You're only interested in the Z value, as you're using a 2D camera, and this is the only relevant axis. You pass in the aimAngle * Mathf.Rad2Deg which converts the radian angle to an angle in degrees.
    The player position is tracked using a convenient variable to save you from referring to transform.Position all the time.
    Lastly, this is an if..else statement you'll soon use to determine if the rope is attached to an anchor point.

Save the script and return to the editor.

Attach a RopeSystem component to the Player and hook up the various components to the public fields you created in the RopeSystem script. Drag the Player, Crosshair and RopeHingeAnchor to the various fields like this:

    Rope Hinge Anchor: RopeHingeAnchor
    Rope Joint: Player
    Crosshair: Crosshair
    Crosshair Sprite: Crosshair
    Player Movement: Player
Right now, you're doing all those fancy calculations for aiming, but there’s no visual candy to show off all that work. Not to worry though, you'll tackle that next
This method will position the crosshair based on the aimAngle that you pass in (a float value you calculated in Update()) in a way that it circles around you in a radius of 1 unit. It'll also ensure the crosshair sprite is enabled if it isn’t already
The LineRenderer will hold a reference to the line renderer that will display the rope. The LayerMask will allow you to customize which physics layers the grappling hook's raycast will be able to interact with and potentially hit. The ropeMaxCastDistance value will set a maximum distance the raycast can fire.

Finally, the list of Vector2 positions will be used to track the rope wrapping points when you get a little further in this tutorial
Here is an explanation of what the above code does:

    HandleInput is called from the Update() loop, and simply polls for input from the left and right mouse buttons.
    When a left mouse click is registered, the rope line renderer is enabled and a 2D raycast is fired out from the player position in the aiming direction. A maximum distance is specified so that the grappling hook can't be fired in infinite distance, and a custom mask is applied so that you can specify which physics layers the raycast is able to hit.
    If a valid raycast hit is found, ropeAttached is set to true, and a check is done on the list of rope vertex positions to make sure the point hit isn't in there already.
    Provided the above check is true, then a small impulse force is added to the slug to hop him up off the ground, and the ropeJoint (DistanceJoint2D) is enabled, and set with a distance equal to the distance between the slug and the raycast hitpoint. The anchor sprite is also enabled.
    If the raycast doesn't hit anything, then the rope line renderer and rope joint are disabled, and the ropeAttached flag is set to false.
    If the right mouse button is clicked, the ResetRope() method is called, which will disable and reset all rope/grappling hook related parameters to what they should be when the grappling hook is not being used
    That slug isn't going to get airborne without a rope, so now would be a good time to give him something that visually represents a rope, and also has the ability to “wrap” around angles.

A line renderer is perfect for this, because it allows you to provide the amount of points and their positions in world space.

The idea here is that you'll always keep the rope's first vertex (0) on the player's position, and all other vertices will be positioned dynamically wherever the rope needs to wrap around, including the current pivot position that is the next point down the rope from the player.

Select Player and add a LineRenderer component to it. Set the Width to 0.075. Expand the Materials rollout and for Element 0, choose the RopeMaterial material, included in the project's Materials folder. Lastly for the Line Renderer, select Distribute Per Segment under the Texture Mode selection

Drag the Line Renderer component to Rope System's Rope Renderer field.

Click the Rope Layer Mask drop down, and choose Default, Rope, and Pivot as the layers that the raycast can interact with. This will ensure that when the raycast is made, it'll only collide with these layers, and not with other things such as the player

Explaining the above code:

    Return out of this method if the rope isn't actually attached.
    Set the rope's line renderer vertex count (positions) to whatever number of positions are stored in ropePositions, plus 1 more (for the player's position).
    Loop backwards through the ropePositions list, and for every position (except the last position), set the line renderer vertex position to the Vector2 position stored at the current index being looped through in ropePositions.
    Set the rope anchor to the second-to-last rope position where the current hinge/anchor should be, or if there is only one rope position, then set that one to be the anchor point. This configures the ropeJoint distance to the distance between the player and the current rope position being looped over.
    This if-statement handles the case where the rope position being looped over is the second-to-last one; that is, the point at which the rope connects to an object, a.k.a. the current hinge/anchor point.
    This else block handles setting the rope's last vertex position to the player's current position
    
    Save the changes to your script and run the game again. Make a little jump with space bar, while aiming and firing at the rock above you. You can now admire the fruits of your labor as you watch the slug dangle peacefully above the rocks
    You can also switch to the scene view, select the Player, use the move tool (W by default) to move him around and watch how the rope line renderer's two vertices follows the grapple position and the player's position to draw the rope. Letting go of the player while moving him will result in the DistanceJoint2D re-configuring the distance correctly, and the slug will continue swinging by the connected joint

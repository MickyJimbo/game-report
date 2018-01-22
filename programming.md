# Programming Project Fat Spiderman

## Setting Up The Github Repository
Github only allows files of up to 20 megabytes, and whilst for the most part we wouldn’t be dealing with files bigger than this it was very important that we did not limit ourselves from such an early stage. Fortunately github provides a system for working with larger files called Git LFS.

There were several ways we could setup the github project, and whilst none of them are particularly complicated by far the simplest way to set up a Git controlled UE4 project is by using the inbuilt tools within the engine. [IMAGE] From here UE4 will setup a local git repository on your computer for the game project. This tool is particularly useful as it allows you to set up the .gitignore file automatically, and providing you have Git LFS installed on your machine you can also automatically generate a .gitattributes file. Github only allows files of up to 20 megabytes, and whilst for the most part we wouldn’t be dealing with files bigger than this it was very important that we did not limit ourselves from such an early stage. So using Git LFS was a necessity.

#### Unreal Engines source control tools
![UE4SourceControl](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/sourceControlUE4.png)

The trouble with Git LFS is that Github provides only a small amount of data with a free account. About 1GB of data and 1GB of storage. Our project, due to the inclusion starter content, started at the size of 750MB. So almost immediately we had used 75% of our storage and after the first download of the project files 75% of our data. For this reason we decided to pay a small fee and increase the limit to 50GB of data and 50GB of storage. Upon reading the terms of the data allowance I noticed that if the project is public anyone downloading the files would use the data allowance from of my account, this meant that the project had to be set to private to prevent others from downloading the project.

From here the only remaining thing to do was to make sure everyone in the team had contributor status to the project.
As a team we decided the best way to organise additions to the game would be to have a set of different branches. The main branch being the master where only commits and merges could be made if they were of a stable build. The next branch would be a develop branch which could be used for tests and inclusions of new features. The last tier of branches would be the new feature branches where anyone could test and create new features without breaking the develop build. This system provided a layer of buffers that made sure that we always had a working version of the game.

## Blueprints
I was responsible for three parts of the game; the swinging mechanics, the aiming system, and a button system to allow interaction with certain puzzles in the game.

### 1) Swing Mechanics
Whilst thinking about how I would be able to achieve swinging in game it became obvious that I would want to take advantage of Unreal Engine physics system. After some searching online I discovered I could use a physics constraint.

Before starting any blueprint work I decided to set up a simple system using a single physics constraint to test the ability to swing. This required only two static meshes and the physics constraint. I then decided to use the set location function on the character every tick to attach the player character to the swinging end of the physics constraint. This created some nice results, so I decided to take it to the next step and make it more dynamic.

In order to make it more dynamic I needed the ability to select an object, generate a physics object, and then set the character location to the tail end of the physics constraint. This was best achieved using a raycast from the player character camera. Once the raycast hit an object I could then attach the head of the physics constraint to the object at the location of the raycast hit. The other end of the physics constraint could then be set to the location of the character. Then again on every tick setting the characters location to the tail end of the physics constraint would produce the dynamic swinging mechanic that I was looking for. 

#### Spawning static meshes and linking them to the physics constraint
![SpawnStaticMeshes](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/staticMeshesForConstraint.png)

![LinkPhysics](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/staticmeshPhysicConstraintLinking.png)

There were, however, still some problems. If the character was running at the point of the physics constraint creation the velocity of the character would not carry on over into the swing mechanic. This was relatively easy to fix as at the point of creation I could get the characters velocity and applied onto the tail end of the physics constraint. 

#### Setting the initial velocity of the swing
![SwingInitVelocity](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/initialSwingVelocity.png)

This revealed another problem, the physics constraint acted more like a rigid pole than a rope and if the character was moving towards the head of the physics constraint It would still lose its velocity.

In an attempt to combat this I came up with a system of dynamic physics constraints connected as a chain. The hope was that by effectively simulating a chain it would act more like a rope than a rigid pole. Unfortunately increasing the number of chain links only resulted in the swing mechanics becoming more unpredictable and ultimately I had to abandon this method.

So now that I knew I was only able to use a single physics constraint to get the predictive results that I wanted I was going to have to get a bit more creative with the character movement. What I ended up doing was comparing the velocity of the character to the dot product of the vector between the head of the physics constraint and the character this effectively give me a value telling me whether the character was moving towards the head of the physics constraint or not.

#### Checking to see if the characters velocity is perpendicular to the head of the physics constraint
![Perpendicular](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/velocityPerpendicularity.png)

So depending on whether the character was moving towards the head of the physics constraint I could deactivate the swinging mechanics, then once the character was moving away I could reactivate the swinging mechanics. The results of this were fairly clean and felt nice to play with. However, there was still one more problem.

When the player decided to deactivate the swing mechanics you would be shot immediately into the ground. After a lot of digging around I discovered the reason for this was because using the set location function does not reset the velocity the character movement component uses. So all the time the character was airborne the character movement component would be adding to the Z velocity and when you were no longer resetting the location of the character the component assumed that the character had been falling the entire time and would shoot you into the ground. I was unable to find a way to reset this internal value so in the end I decided to remove the set location function and replace it with the launch player function which played better with the character movement component. Using the launch player function meant that instead of setting a location I had to match the velocity of the tail end of the physics constraint and the character.

#### Vector maths to match the velocity of the character to the tail end of the rope
![VelocityMatching](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/velocityMatching.png)

I also had to add a small amount of bias depending on how far away the character was from the tail end of the physics constraint, basically I would add more to the velocity of the character as the character drifted further from the tail end of the physics constraint. Another benefit of using the launch player function was that it had the unintended effect of making the swinging feel smoother.

#### The point the rope is spawned
![SpawnRope](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/grappleAttachOnClick.png)

#### Final rope blueprint
![GrappleLogic](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/grappleAndSwingLogic.png)


### 2) Aiming Mechanics
Once the swing mechanics were finalised it became quickly apparent that a simple raycast was not going to be sufficient for the type of gameplay we were aiming for. A swingable object that was further away would be much harder to aim at than a swingable object that was closer. As this game was intended to be more of a puzzle game than testing the players coordination a new system would have to be devised.

I decided to go with a cone scan. This puts an invisible cone in front of the character camera that checks for object collisions. I would then put the swingable objects that collide with the cone into an array. This left me with a set of potential aimable objects but in order to determine the the most suitable object to aim at it would require some additional processing.

In my first iteration of this processing I would fetch the dot products between each objects vector to the camera and the camera’s forward vector and the value closest to 1 would be chosen as the aimed object. However, this still meant that objects further away were harder to aim at, so in order to solve this I added a bias to objects further away making them just as easy to aim at as closer objects.
In the next iteration of the processing I discovered a much simpler and more accurate way of determining what the player was aiming at. Instead of a dot product check I realised I could use a screen based solution. I could convert an object’s location to a screen location and then simply check the distance between the cursor and the screen location of the object. The object with the shortest distance would then be selected as the aimed object.

![ScreenspaceWorldspace](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/screenspaceToWorldspace.png)

![MoreScreenspaceWorldspace](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/screenspaceDistanceAndOutput.png)

Another thing that needed to be solved was that objects could be aimed at through walls. This was due to the cone scan as it would not be able to detect if an object is being obscured. To fix this I fired a raycast from the camera to the centre of each object detected by the cone scan, if the first hit of the raycast was not the object that it was trying to hit then the object would not be selected as the aimed object.

![Obscured](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/objectObscured.png)

Lastly, the aiming system was plugged in to work for both swingable objects and food sources. Highlight message calls were then sent to the aimed objects using a blueprint interface so the objects being aimed at could choose how they would inform the player of this.

![AimSelection](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/selectingTheAimedActors.png)

### 3) Button System
The last mechanic I was responsible for was the button system. As this was going to need to interact with many other blueprints in the scene I designed it to be as simple as possible to use. In order for another blueprint to interact with the button all it would have to include was the button interface and then contain events for the activate and deactivate interface message calls. Any other functionality would be handled by the button itself.
 
There are four set requirements for the button which were as follows:
- The button can only be depressed when the player is heavy.
- If the player becomes too light whilst standing on the button the button will deactivate. 
- The button must be given an array of blueprints in the scene to send interface calls to.
- The button can start as either on or off, activating the button toggles its state

There are also four types of buttons contained within one blueprint:
- A momentary button which is only active when the character is standing on it.
- A timer button that once triggered will activate a timer and will deactivate when the timer runs out.
- A toggle button that will activate the first time it is stepped on and will deactivate the second time it is stepped on.
- A single use button that once activated cannot be deactivated.

When the idea of the button came up, we were not entirely sure how many different blueprints in the scene would use it so building it to be flexible was very useful. In the end we only used the timed button, but having the options available was very handy when designing the MVP level.

## The Trouble With Git And Unreal Engine Blueprints
Throughout the project we have had several moments where using Git has been a problem. The biggest being the lack of ability to merge and commit changes properly, there were quite a few moments when we needed to be working on the same blueprints but couldn’t because it would have potentially overwritten someone else's work. As blueprints are saved in a binary format Git is unable to detect small changes between the files, and any small changes result in the entire file being overwritten in a commit. This meant that in order to work on the same files some of us would need to manually merge our work later by hand.

Unreal has made an attempt to provide a merge and difference tool for version managing software, however, it seems incomplete and will often crash the engine when used. I imagine that using git and blueprints would become much easier if these tools were implemented better. 

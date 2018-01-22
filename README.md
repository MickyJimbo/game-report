# Project Fat Spiderman

By Michael Payne - 33332539

## Initial Idea of the project
After our first meeting we came up with a game design that we felt would both be challenging and interesting to make. The goal was to create a game with a character who could grapple and swing with the ability to change their own weight. Changing the characters weight would be the main way to interact with the puzzle elements of the game.

The concept for the game was simple but we felt it would allow for a lot of experimentation once we had the gameplay mechanics up and running.
## Team work
To make sure we were all on the same page we had several ways of arranging ourselves. At the beginning of each week we would have a team meeting where we would show the work that we had done and also discuss goals going forward into the next week. During the time when we couldn't be in the lab we would predominantly communicate using our slack channel. We also had a trello board set up but it went unused as the team meetings and the slack channel were sufficient for communication.

As we all had to take part in both programming and art for the project we initially decided that it would be a good idea to split up the tasks for various puzzle elements. A lot of ideas did not make it into the final game but here is a list of just a few that we discussed during our meetings:
- Platforms that would raise and lower based on your weight
- Breakable platforms
- Fans that would push you based on your weight
- Suction tubes that would only work if you are the correct weight
- Walls that would break when hit but only if you are heavy
- Pressure pads that only activates when you are heavy
- Movable swing objects
- Elevators at the end of the level that would only work if you were a certain weight 
- Timed doors and traps

However, people ended up gravitating more to certain tasks, for instance myself and Sam gravitated more to the programming whereas Ali, Nuno, Touraj and Rafael gravitated towards the art elements of the game.

## My Main Roles Within The Team
One of my main roles within the team was managing the github. This involved the initial setup, giving everyone contribution rights to the repository, making sure everyone's local repositories were set up correctly on their computers, and ensuring that everyone understood the right process for creating branches and merging into each other's work.  There were a few problems getting everyones machines to work with the github repository, however, we did eventually get a system where everyone could contribute. The main problems seem to be with getting Git LFS to work correctly on some machines.

The other main role that I had was in programming the swing mechanics, aiming mechanics and the button interaction mechanics. I spent a majority of my time experimenting with various methods to make sure that these gameplay mechanics were both fun and satisfying to use.

I was involved with small amounts of modelling and texturing, the main being the button for the button mechanics, but I was also involved in setting up some simple dynamic instance materials as place holders for the game. Towards the end I was also largely involved with the lighting of the final MVP level which helped a lot with giving it it's final look.

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

#### Converting screen space to world space helper function
![ScreenspaceWorldspace](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/screenspaceToWorldspace.png)

#### Using the helper function and output of the aimed object
![MoreScreenspaceWorldspace](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/screenspaceDistanceAndOutput.png)

Another thing that needed to be solved was that objects could be aimed at through walls. This was due to the cone scan as it would not be able to detect if an object is being obscured. To fix this I fired a raycast from the camera to the centre of each object detected by the cone scan, if the first hit of the raycast was not the object that it was trying to hit then the object would not be selected as the aimed object.

#### Using a raycast to detect if an object is being obscured
![Obscured](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/objectObscured.png)

Lastly, the aiming system was plugged in to work for both swingable objects and food sources. Highlight message calls were then sent to the aimed objects using a blueprint interface so the objects being aimed at could choose how they would inform the player of this.

#### Logic to manage highlighting and unhighlighting of aimed objects
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

#### Button options
![ButtonOptions](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/buttonOptions.png)

When the idea of the button came up, we were not entirely sure how many different blueprints in the scene would use it so building it to be flexible was very useful. In the end we only used the timed button, but having the options available was very handy when designing the MVP level.

#### Button initialisation
![Init](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/buttonInit.png)

#### Button activation logic on state changes
![ActivationLogic](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/buttonTypeLogic.png)

#### Button logic part 1
![ButtonLogic1](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/mainButtonLogic1.png)

#### Button logic part 2
![ButtonLogic1](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/mainButtonLogic2.png)

#### Event to handle light activation and button depression animation
![LightAndDepress](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/communicationEvents.png)

#### Final button blueprint
![Final](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/buttonLogic.png)

## The Trouble With Git And Unreal Engine Blueprints
Throughout the project we have had several moments where using Git has been a problem. The biggest being the lack of ability to merge and commit changes properly, there were quite a few moments when we needed to be working on the same blueprints but couldn’t because it would have potentially overwritten someone else's work. As blueprints are saved in a binary format Git is unable to detect small changes between the files, and any small changes result in the entire file being overwritten in a commit. This meant that in order to work on the same files some of us would need to manually merge our work later by hand.

Unreal has made an attempt to provide a merge and difference tool for version managing software, however, it seems incomplete and will often crash the engine when used. I imagine that using git and blueprints would become much easier if these tools were implemented better.

## Modelling And Texturing

#### WireFrame
![Wireframe](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/ButtonWire1.png)

#### ID Map
![IDMap](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/IDMap.png)

#### Colour
![ButtonColour](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/ButtonColour.png)

#### Roughness
![ButtonRoughness](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/ButtonRoughness.png)

#### Metallic
![ButtonMetallic](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/ButtonMetallic.png)

#### Normal
![ButtonNormal](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/ButtonNormal.png)

#### Substance Painter final material
![ButtonView2](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/Button2.png)

#### Material
![Material](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/DynMaterial.png)

#### In game
![InGame](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/button/ButtonIngame.png)

## Designing The MVP Level

#### Initial MVP level design
![InitialDesign](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/initialDesign.png)

We designed the MVP level during one of our team meetings, we gave ourselves a set of easily achievable puzzle elements that we could use within the game and then discussed how they could be used to make a compelling first level for our game.
The main goal for the MVP was to be able to give this game to someone who's never seen it before and have them be able to figure out how to progress through the level. By the time they came to the next level they would already know how to use every element in the game.
The MVP level consists of essentially five corridors. Each of the first four corridors introduce new gameplay elements as well as increasing difficulty. By the fifth corridor all of the gameplay elements have been introduced and the player will have to come up with creative use of these elements in order to complete the level.

#### Corridor 1
![Corridor1](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/corridors/corridor1.png)

In the first corridor we introduce three gameplay elements, the first being the characters ability to swing, the second, that the fans will push your character, and the third, that you can change your character’s weight to reduce the effects of the fan. In between the first and second corridor we're also teaching the player that they need to reduce their weight in order to make longer jumps.

#### Corridor 2
![Corridor2](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/corridors/corridor2.png)

In the second corridor we're teaching the player they need to be heavy in order to break the destructible wall. The player will also learn to chain swings together. Lastly, the player will also learn that they can change their weight during their swing.

#### Corridor 3
![Corridor3](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/corridors/corridor3.png)

In the third corridor we're reuse the fans to show that they can also be useful. By positioning the fans differently the fans can be used as launch pads, however the player must be the correct weight for it to have the best effect. Being too heavy has no effect and being too light has too much effect, it requires the player to take better control of their character’s weight.

#### Corridor 4
![Corridor4](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/corridors/corridor4.png)

In the fourth corridor we're introducing a weight controlled timed button that controls a laser grid on the other side of the corridor. If the player takes too long the laser grid will reactivate and potentially kill the player.

#### Corridor 5
![Corridor5](https://github.com/MickyJimbo/game-report/blob/master/Screenshots/corridors/corridor5.png)

In the fifth and final corridor we use all of the elements together. The character must be heavy in order to push the button that activates the timed door at the back of the room. The player must then shed weight to use the fan that launches them up high enough to reach the swing point. Finally the player must use a food source to regain weight to be able to break through the destructible wall on the other side of the door. The player must do this in a timely fashion otherwise the laser grid will reactivate and they will need to restart.

## Lighting
## Conclusion 

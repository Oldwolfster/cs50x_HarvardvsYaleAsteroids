# Harvard vs Yale Asteroids in Scratch
#### Video Demo:  <URL HERE>
#### Description:  This project stands on the shoulders of giants, inspired by the pioneering work of Steve Russell and his colleagues who brought us "Spacewar!" in 1962.  This version, is meant to push Scratch to the LIMIT, demonstrating that as long as you have the "standard building blocks", anything is possible!   Even if they are simple Scratch components.

### Overview:  The protagonist of the story is the Harvard ship, which must evade and destroy the Yale Asteroids and MIT sharpshooters.

### **Ways this implementation pushes Scratch to the limit  Details Below** 
- **Scoreboard and Level Announcement using "real fonts"**
- **Asteroid size algorithm**
- **Scoring algorithm**
- **Accuracy of MIT Sharpshooter**
- **Integrated instructions as the user needs the information**

### Component List:
- **Harvard Mothership:** Purely for aesthetics and storyline.  No Impact on game.
- **Harvard Player Ship:**  This is the only object controlled by the player.  It can...
  - Rotate Left or Right
  - Thrust
  - Shoot.
  - To stop, it must thrust opposite of its current vector
- **Harvard Photon Torpedo:** Fired from the Harvard Player Ship  
- **Yale Asteroids:**  Float in a random velocity and direction.  When shot break in to smaller pieces.
- **MIT Sharpshooter:** Ship that will shoot at the Harvard Player.
- **MIT Photon Torpedo:** 
- **Instruction Panel:**
- **Level Announcement:**
- **Scoreboard:**
- **Lives Display:**
- **Explosion Particles:**

### Key Variable: GameState is essential the "lifecycle" of the game.  It can have these values.

- **Pregame states are all negative**:
- -10 Display Instr From Mother Ship
- -9 Wait for user to read instructions
- -8 Mothership going to warp drive
- -7 Intro Over Start Main Loop
- -5 Show Level
- -4 Wait on lvl
- -3 fade lvl
- -2 ready to laucnh ship as invulnerable.

- ** States during gameplay
- 0 invulnerable (waiting for player to start)
- 1 vulnerable (The actual game play)
- 2 Dead (but still have lives remaining)
- 3 beat level
-11 Game Over

### Component Details:
- **Scoreboard:** While not the most exciting aspect of the game, i do believe this is what pushed Scratch the most.  I would like to try to package it to share with other Scratch developers.
  - The scoreboard consists of 8 "Digits"
  - Each "Digit" is identical with the exception of
    - a private variable indicating what "place it is i.e. ones, tens, hundreds, millions, etc.  
    - It's represented as the exponent of 10, i.e. ones = 1, tens=2, millions=7, etc.
    - Each "Digit" contains a costume for all numbers from 0 to 9
    - There is a function called "YoinkDigitFromScore"
      - It takes "Score"(aka NumberToMine) and "Place"(aka DigitToMine) as parameters.
      - As Scratch does not support return values it sets a private variable
      - As an example YoinkDigitFromScore (2, 12345) = 4 because 4 is in the tens place.
      - The formula is "Floor" of ((NumberToMine Mod (10^DigitToMine)) / ((10^DigitToMine)-1)
    - After "mining" the number, it uses that value to set the appropriate costume (i.e. set the correct character to visible 0-9)
- **Harvard Mothership:** While purely for aesthetics it does contain the logic to start and initialize the game:
  - When user clicks Green Flag or presses Z, it calls block "PrepareNewGame"
  - PrepareNewGame...
    - Calls block "SetInitialValues"
    - Broadcasts "Initializing" message.  (NOTE: Broadcast message is the only method i found to communicate between different objects)
  - It also has a listener for "Initializing" and adjusts the image as well as sets the gamestate global variable.
  - Finally, it has some random components for managing the visibility, rotation, etc.
- **Harvard Player Ship:**  
  - Main Game Loop:  Triggered via "Initializing" broadcast. Calls 4 other blocks
    - CheckForLaunch
    - CheckForLevelCompletion
    - CheckForDeath
    - MoveShip
  - Ship Controls
    - First, the ships movement is controlled by 4 key variables.
      - MoveX and MoveY is how much the ship will move each "cycle"
      - ChangeX and ChangeY are set when the ship is rotated.  
        - They indicate how much to adjust MoveX and MoveY when the thruster is hit. 
        - ChangeX is calculated as the cosine of the ship angle (scaled and adjusted for Scratch's coordinate system) 
        - ChangeY is calculated as the sine of the ship angle (scaled and adjusted for Scratch's coordinate system)
    - Next, all ship movement events follow the same process.
      - DropShieldIfUp (in case this life is just starting (switches gamestate from 0 to 1)
      - Checks if the gameState is 1, if so...
      - Performs action
    - Left and Right Action: Rotate ship and calculate ChangeX and ChangeY as described above.
    - Thrust:  Add ChangeX to MoveX and ChangeY to MoveY
    - Fire:  Creates a clone of the photon torpedo and fires it using the same vector as ship's ChangeX and ChangeY
    - Checking State:  There are groups called each cycle from the MainGameLoop checking various states.
      - CheckBorder:  If ship moves off screen to the top move it to the bottom, same horizontally.
      - CheckDeath:  Is there a collision with a Yale asteroid, MIT Ship, or MIT bullet?
      - CheckForLevelCompletion: Are all the asteroids cleared?    - 
-  **Yale Asteroids:** 
  - "When I start as Clone" event:
    - SetLocation
      - If a new asteroid (start of level) calls LocForNewAsteroid
      - If debris from larger asteroid being shot calls LocForDebris
    - SetSize
      - If a new asteroid (start of level) it starts at 30
      - For debris, it randomly decreases size.
      - This allows us to set up a rare situation with unusually small asteroids.
      - As the asteroids get smaller the score goes up exponentially. 
    - SetMovement: movement vector similar to Harvard Ship.
    - Runs loop while gameState > -3
      - Checks if asteroid hit border.
      - Checks if asteroid has been shot.
    - Scoring methodology is a little unique.
      - In classic asteroids there were exactly 3 sizes of asteroids.
      - Here, it's a range from 30% to rarely 7%.
      - As the smallest asteroids are rare, and hard to hit, the score goes up expotentially.
        - The math gets a little hairy as i could not find an operator for exponent, but i did find a substitution using logarithms.
        - Function that increases exponentially as the size of the Yale decreases.  Max size is 30
        - score = A * B to the power of (C − size)
        - A, B, and C are constants that you determine based on how you want to scale the scores.
          - A: This is a base score multiplier. Adjusting A will scale the entire score range up or down.
          - B: This is the base of your exponential function. A value greater than 1 will make the score increase exponentially as the asteroid size decreases. A typical starting point could be B=2, but you can adjust this for more or less steepness.
          - C: This is a size offset, which should be set to a value slightly larger than your maximum asteroid size to ensure the exponent stays positive. For your range, 31 could be a go
        - This methodology allowed me to tweak with very fine control how the scoring worked.
- **MIT Sharpshooter:** Ship that will shoot at the Harvard Player.
  - Upon initialization, it runs a loop to check if the level, gameState, and a random amount of time has passed to spawn.
  - Once spawned, it runs a loop in a block called RunMIT
    - Moves based on same vector math as everything else.
    - Calls 4 additional blocks.
      - CheckBorders (unlike Harvard Ship and Asteroid which "wrap" MIT "bounces" when it touches a border)
      - CheckCollision (Checking to see if MIT ship has been shot)
      - CheckPulse - graphic effect where the size has a slight pulse to it.
        - CheckFire - After a random amount of time, the MIT will shoot a bullet at the Harvard ship.  If the ship does not move it will be hit. 
- **Harvard Photon Torpedo:** 
  - "When I start as Clone" event:
    - Adopts same vector as ShipChangeX and ShipChangeY
    - Uses Pythagorean theorem to equalize magnitude
    - Scales by BulletSpeed variable.
    - Creates loop with two block calls while GameState=1
      - Move bullet.
      - Check to see if it hits edge (in which case delete it)
    - Listens for Message from Yale which triggers collision logic.
- **MIT Photon Torpedo:** 
  - Upon receiving broadcast, "MIT Missile Away"
    - Uses pythagorean theorem to calculate vector to hit Harvard ship
    - Repeats loop
      - Moving bullet,
      - Checking for collision with Harvard Ship.
      - Checking for hitting border (bullet is destroyed)
- **Instruction Panel:**
  - Only the verbiage in the instructions.  They are removed from logic in Harvard Mothership.
- **Level Announcement:** Fades in and out between levels.  Functions similar to Scoreboard.

- **Lives Display:**  Functions similar to Scoreboard.
- **Explosion Particles:**  Spawn when something is shot for special effects.

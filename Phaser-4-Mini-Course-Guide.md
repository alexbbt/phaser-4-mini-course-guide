# Build Your First Game with Phaser 4 — Course Guide

A per-video study guide for the mini course *Build Your First Game with Phaser 4* (falling-object catch game). Each section summarizes what to learn and do, then includes the cleaned English transcript.

**Playlist:** https://www.youtube.com/playlist?list=PLmcXe0-sfoSheQinG8d5JQBiTqiKnBgo7

**Local videos:** same folder as this file (`01`–`09` MP4s)

---

## Contents

1. [Project Setup & Core Concepts](#01-project-setup-core-concepts)
2. [Coordinates & Positioning](#02-coordinates-positioning)
3. [Player Movement with Keyboard Input](#03-player-movement-with-keyboard-input)
4. [Spawning Objects Over Time](#04-spawning-objects-over-time)
5. [Collision Detection Without Physics](#05-collision-detection-without-physics)
6. [Scoring, UI Text, and Game Feedback](#06-scoring-ui-text-and-game-feedback)
7. [Game Rules, Win Conditions, and Restarting](#07-game-rules-win-conditions-and-restarting)
8. [Where to Go Next (Wrap-Up)](#08-where-to-go-next-wrap-up)
9. [Updating Phaser 4 to the Latest Version](#09-updating-phaser-4-to-the-latest-version)

---

## 01. Project Setup & Core Concepts

**Duration:** 8:04 · **Video:** https://www.youtube.com/watch?v=cRsYgXm6I3k

**Goal:** Set up the starter project and build a mental model of how Phaser runs a game.

### Key concepts

- Phaser is a JavaScript framework for browser games (rendering, input, assets, game loop).
- You need a local web server (Live Server, Python `http.server`, or `http-server` via npm).
- Entry point is `index.html` → `main.js` creates `new Phaser.Game(config)`.
- Scenes own gameplay; every scene uses the lifecycle: **preload → create → update**.
- This project uses a **PreloadScene** (load assets) then starts **GameScene** (gameplay).
- `preload` runs once (load assets), `create` runs once (build world), `update` runs ~60×/sec (input + state).

### What to do

- Download the starter template and open it in VS Code (or your editor).
- Start a local server (e.g. Live Server → Go Live).
- Confirm the Phaser banner/version in the browser console.
- Add `preload` / `create` / `update` logs in GameScene to see the loop fire.

### Transcript

In this video, we're going to set up our first Phaser 4 project and more importantly understand how Phaser actually works. By the end of this video, you won't have a full game yet and that's intentional. What you will have is a clear mental model of how Phaser is structured, how a game runs, and where your code fits into that process. Once you understand that, everything else becomes much easier.

Over the course of this mini course, we're going to build a complete game together. It's a simple falling objects catch game, nothing fancy, but it will teach you the core ideas that apply to almost every Phaser game. We'll handle player movement, spawning objects, collision detection, scoring, and basic game rules. And we're going to do it without jumping straight into the physics engines, so you actually understand what's happening behind the scenes.

To keep things simple, I've included a small starter project in the description. This project already includes Phaser, a basic HTML file, and a simple scene. That means you don't have to fight with setup issues before we even start learning. But don't worry, we're still going to walk through exactly how this project works, so you understand what Phaser is doing behind the scene.

Now to follow along, go ahead and download the starter project template now and open it up in your IDE or code editor of your choice. For this series, I'll be using VS Code. Once you have the project files, next we'll need to set up a local web server so you can see the game running so we can test our changes locally. There's a variety of ways to do this, but a few of the common options are if you have Python 3 installed, you can use the built-in HTTP server to start a local server and see your project running in the browser.

If you have Node.js installed, you can use the HTTP server npm package. And if you're using VS Code, you can install an extension called Live Server. There are other options available to you, but we won't be covering them in this video. Since I'm using VS Code, I'm going to use the Live Server extension.

How this works is in VS Code, if you go into the extensions marketplace, you can search for Live Server. And here we'll have the Live Server by Ritwick Dey. You'll want to install that and then once you do, you should have a new option appear at the bottom of your toolbar where you can click go live and this will start your local web server and open up your game in your browser. Another way to run the Live Server is if you open up your command panel, you can type in Live Server and this will give you your options to start and stop the server.

server. Once you have your local web server running, you should see your Phaser game instance, which is just this background image here. And if you open up your developer tools in our console, we should see our Phaser banner with the version of Phaser that we're currently running. Now that we have our project running locally, let's take a quick look at the files that are included in our project template.

You don't need to memorize the structure yet, I just want you to understand the pieces that make the game run. run. So index.html, this is the file that is the entry point for our game in the browser. It loads our JavaScript code, which is where our actual game logic lives.

In most Phaser projects, the HTML file stays very small. It's really just a container for your game. Next, in main.js, this file is where our game actually starts. Right here, we create a brand new Phaser game instance.

This line is what launches our game. The configuration object we pass in, this tells Phaser things like how big our game should be, what scene we should start with, and how rendering should work. You don't need to understand every option yet. We'll keep things simple throughout this course.

So in our game configuration, our scale object here, this is where we have the properties that define the size of our game. So this is going to tell Phaser how big our canvas element should be when our game first loads. Then down here, where we call game.scene.add, this is one way we tell Phaser which scenes we want to use for our game. We call game.scene.start, we're telling Phaser to start this specific scene.

So for our preload scene, if we open up our preload-scene file, this is our first Phaser scene that our game will show. So in Phaser, almost everything happens inside our scenes. A scene controls what appears on the screen and how the game behaves. And every scene will follow the same life cycle where we want to load in any assets we need, we create our game objects, and then we listen for our player input and react accordingly.

And we'll get more into the game loop in just a second. And for our project, we currently have two Phaser scenes. We have our preload scene and our game scene. So with Phaser, you always have to have at least one scene, but you can have as many as you would like.

How our project is currently set up is our preload scene is going to be responsible for loading in all of the assets that we'll need. We then call this scene.start to go to our game scene and inside here, this is where we render out our background image when we view our game in the browser. Our game scene is going to have all of our core logic for our actual game play. For now, don't worry about all of the code that's inside our scenes.

We'll get more into that later. Our assets folder, this will have all of our static assets that we need for our game. This includes the Phaser library as well as the images and sprite sheets that we'll be using for our project. Finally, under our common folder, we have some boilerplate code that we'll be using in our starter project.

And we'll get to that in a second. So what exactly is Phaser? Phaser is a JavaScript framework that's designed specifically for making games that will run in the browser. It doesn't replace JavaScript, instead it gives you the tools for things games constantly need.

Rendering out graphics, handling input, loading assets, and running all of this in a game loop. For the games you'll create, you're still going to write in JavaScript. Phaser just provides the structure that makes building the games much easier. Every game ever made runs on a concept called the game loop.

A typical game loop is an infinite cycle where we process user input, we then update our game state, and then render out our frames typically running between 30 to 60 times per second. This is the core loop of your game engine. Phaser organizes this loop into something called a scene. And each scene has three important phases.

Preload, this is where we want to load in any assets like images, sounds, and data that we'll need for our game. Then create, this is going to run once after everything is loaded that we need. This is where we'll build our game world. And then finally update, update is going to run over and over again, usually about 60 times per second.

This is where our movement happens, we'll check for input, and our game state is going to change. And we'll keep repeating our update loop until we receive a signal to shut down our scene. If you understand preload, create, and update, you already understand the core of Phaser. Most beginner confusion happens when people don't know when their code runs.

So throughout this series, I'll always point out which part of the life cycle we're working in. And so how our core game loop works with our Phaser scenes is when we define a Phaser scene instance, we can plug in our preload, create, and update methods and internally, Phaser is going to invoke those methods at the appropriate time for our game loop. So to see that our game loop is running, if we add the preload and update methods to our game scene, and we just add a console.log statement, if we open up our developer console and refresh our browser, we should see our console.log statements appear. We'll see preload and create are called one time, and then we can see our update log message is firing over and over again.

That means Phaser is running, the scene is active, and our core game loop is alive. Nothing is moving yet, nothing is on screen, but the foundation of our game is working. So far, we've learned what Phaser is, how a game runs, and how scenes control the game loop. In the next video, we're going to put our first sprite on our screen.

We'll talk about coordinates, positioning, and how the game world maps to the screen. When you're ready, move on to the next lesson and let's keep building.

---

## 02. Coordinates & Positioning

**Duration:** 11:21 · **Video:** https://www.youtube.com/watch?v=Ml9jKIBPjEo

**Goal:** Put sprites on screen and understand X/Y, origin, and asset loading.

### Key concepts

- Images are static textures; sprites can animate — neither has physics by default.
- Load assets in `preload` with keys (`this.load.image`, sprite sheets / texture atlases).
- Create objects in `create` with `this.add.image(x, y, key)` / sprite + optional frame.
- Coordinates start at **top-left (0,0)**; X increases right, Y increases down; negatives exist off-screen.
- Default **origin is the center** — placing at (0,0) centers the object on the corner, so only part shows.
- Center with `scale.width/2`, `scale.height/2`; size with `.setScale()`.
- Texture atlases: JSON frames tell Phaser how to cut the sheet; pass a frame name to show a specific image.

### What to do

- Load background, jar (player), and falling-object atlas in PreloadScene.
- Add centered background; add jar near bottom center; add a falling object with a chosen frame.
- Experiment with origin, position, and `setScale`.

### Transcript

In the last video, we proved that Phaser is running and we talked about how the game loop works. In this video, we're going to put something on the screen for the first time. And more importantly, we're going to answer one of the most confusing questions for beginners. Where does stuff actually appear and why?

If you've ever added a sprite or image and thought, why is this off-screen or why isn't this centered? This video is for you. Let's start simple. In Phaser, an image is just a game object that we're rendering to our screen at a given position.

This type of game object allows us to take an external texture and display it in our game. Another version of our image game object is the sprite game object. So, the main difference between our image and our sprite game objects is our sprite game object can play animations. Our image will be static and neither of these have any physics.

They're just an image with an X and Y value and one of them can play animations. If you understand how the X and Y values work, everything else becomes much easier. Before we can use images or sprite sheets in your game, we need to tell Phaser to load that asset. How this typically works is in one of your Phaser scenes, in the preload method, we want to use our Phaser loader plugin to load in our different assets.

So, in our preload scene, we're calling this.load.image and we're providing a unique key for that asset we're loading and the path to where that file is located. How our code is currently set up is we have our assets in this array here and for each asset, we have a unique key and where that file can be found so Phaser can load it. So, we have assets, images, and then our background PNG, and then our jar PNG. And so, in our preload method, we can also tell Phaser to load other asset types.

Besides our images, we can tell Phaser to load in our other file types. So, this includes our sprite sheets, our texture atlases, audio, video, and much more. And so, when we call one of these methods, our key, this is our unique key, which is a name for where this asset will be placed in our cache. We use that name when we want to create that game object, so that way Phaser knows which image or asset to use for the game object we're creating.

Now, by loading all of our assets in our preload method, this tells Phaser we don't want to start our game until all these assets have been loaded. Once Phaser has loaded in those assets and they're available to use, then we go into our create method. So, once we hit our create method, this is where we can create those game objects. So, in our starting project template, we're already adding an image game object to our game.

And this is our background image. To add our image, we need to specify our X, our Y, and which texture we want to use. So, our texture, this is our key that we used when we called this.load.image. And we need these to match, so Phaser will use the right texture.

As an example, if I change this to to a string that doesn't exist in our cache, now Phaser displays a broken image. Since we loaded in our background image with the key of background, now Phaser knows to display that image for that game object. When we want to create a sprite, we do the same thing. We specify our X and Y, which texture we want to use, and then if we're using a sprite sheet, we can specify our frame.

And we'll get more into sprite sheets later, but for now, if we omit our frame, Phaser will default to using our first frame. Now, for placing our game objects in our game, we have to specify our X and Y values. And this is where the confusion usually starts. So, let's slow this down.

down. So, in Phaser, and in most 2D games, our coordinate system will start in the top left-hand corner of our screen. So, at this point, our X is set to zero and our Y is set to zero. And so, we have a 2D coordinate of 0 0.

Now, as we move to the right, our X value is going to increase. So, we start from zero, we go to one, two, and we keep increasing infinitely in this direction. So, just as an example, if this was 500 pixels, we'd be at X 500. Now, as we move down our screen, this will make our Y value increase.

We start at zero, and then as we move down, we'll just keep increasing. So, now, when I place a game object at 00, I'm saying I want to be here in the top left-hand corner. If I place it at 0 500, I keep my X at zero and increment my Y to 500, and then if I wanted something like 100 100, I would move over 100 pixels, down 100 pixels. And so, this isn't just a Phaser thing.

This is typically something we'd see in most game engines. And so, one thing to keep in mind is for our 2D coordinate system, we don't stop at 00. Our game coordinates extend infinitely in both directions. What this means is when we move left past zero, we'll have a negative value.

And if we move up past zero, we'll have a negative value for our Y. So, to see an example of this in action, come back to our code. We're going to remove this code here for our background. I'm going to place in 0 0.

Now, if we come back to our game, what happens is we only see part of our background. And here's another beginner trap. When you place an image at 00, you might expect the sprite's top left to line up with our screen. But, by default, Phaser positions your game objects by their origin or their center.

So, each game object that we create has a point that's called their origin. That origin is how Phaser positions our game objects in our game. So, when I say I want to place my game object at 00, we're placing our origin or our center at that location. And so, for like our background image like this, when we go to position 00, we only see our bottom right-hand corner of our image.

So, if we want to see our full image, we need to place our origin in the center of our game. game. So, for previous code, that's exactly what we were doing. This scale.width will give us how big our current game is, and we're dividing by two to get the center, and then we're doing the same thing for our height to get the center.

That results in our image game object being centered in our game, and now we can see our full image. And so, this is something important to understand because anytime we want to place a game object in our game, we need to keep in mind where our origin is for that game object. Otherwise, we can have instances where our game object doesn't appear fully on our screen. So, now to manually move our game objects, all we need to do is update those X and Y values.

So, if I take this image game object here, and I change my X and Y to be 50, now my game object is moved in my game. And this leads to one of the most important ideas in game development. Games are just numbers changing over time. Our position is a number, movement is numbers changing every frame, and if you understand that, movement, collisions, and gameplay stop feeling mysterious.

Now, before we wrap up, let's add in some game objects to represent our player and our objects that are going to fall that we'll need to catch. So, if you come back to our project, under our assets folder under images, our jar.png, this is going to be our player where we're going to catch our falling objects. And our sprite sheet, our PNG file, this has all the images representing the objects that will fall and we'll want to catch. So, our project is already set up to load in all these assets.

So, in our preload scene, we're referencing our image assets array, and inside here, we have our image. And then for our texture atlas assets array, this has our sprite sheet, and we're telling Phaser to load these in. So, now, once we get to our game scene, those assets are available and that's how we're rendering out our background. So, now we'll go ahead and add in our player.

So, for our player, we'll do our image game object. So, we're going to do this.add.image. And now we want to center our image just like we do with our background. Instead of referencing this.scale.width every time we want to do that, we'll make a new variable that we can use in our create method.

Let's do const. Let's do our width and our height. We'll grab those from our scale manager. So, now down here, we can just do our width / 2.

We'll do our height / 2. So, now for our jar, we want to be centered. So, we'll do our width / 2. / 2.

And now we want our game object positioned at the bottom of our scene. So, by doing our height, that's going to grab our full height of our scene and then place our game object down here at the bottom. Now, because of our origin, our game object will still be partially visible. Finally, for our texture, we're going to do our asset keys and we want to do our jar object.

So, now if we save, come back to our browser, we should see our new image added to our game. So, I'm going to copy that line of code and now we're going to do our falling object. So, for our falling object, we'll center that in our screen. So, we'll do our height / 2.

And now we want to reference our objects on our asset key. key. So, now if we save and come back to our browser, we're going to see our button game object show up in our game. So, what's happening is when we tell Phaser to load in our sprite sheet here, because we didn't specify a frame, Phaser's going to take our first frame of our sprite sheet and render that out in our game.

How this works is we have this JSON file here that has all of the frames for each of our images. This is instructions to tell Phaser how to cut up that image into multiple smaller images. And so, if we want to display a different image, we just need to pass in the name of the frame that we want to use. So, if I copy this button2.png, come back to our game scene, and pass in one more argument.

This will be our frame. frame. Now, if we come back to our browser, now we're displaying our second button. And now what's really cool is for any of these frames we have defined here, we can display our different image.

If I grab a different frame, come over here, I'll paste it. Now, we're displaying a different game object. object. So, for the time being, I'm just going to do our first frame.

If we go ahead and save and come back to our browser, we should see our button. So, one last thing we'll do is for our game objects, our button's a little bit big. We can change the size of our game objects by using our scale property. How that works is we can call .setScale and now we can provide a value for how much we want to scale our X and Y by.

By default, these are both set to one, which means we're going to render out our image at the original size that we load in. If I want to double my size, I can bump these both to be two. And now my game object will be scaled up and is twice its size. If you only provide one value, Phaser will apply that to both our X and Y.

But then you can also do stuff like this, where you only scale one of the properties. For scaling down, we can provide a value less than one. Now, that's going to shrink our game object. Nice.

In our next video, we're going to make our image game object move on its own. We'll use the update loop to change our numbers over time, and we'll build our first real piece of gameplay. When you're ready, move on to the next lesson, and let's keep building.

---

## 03. Player Movement with Keyboard Input

**Duration:** 7:21 · **Video:** https://www.youtube.com/watch?v=7NZ84qQqMog

**Goal:** Move the player with arrow keys using input polling and delta time — no physics.

### Key concepts

- Movement = changing X/Y over time; keyboard only decides *when* to change numbers.
- `this.input.keyboard.createCursorKeys()` in `create`; poll `isDown` every frame in `update`.
- Store `this.player`, `this.cursorKeys`, and `this.playerSpeed` (init in `init()`).
- `init()` runs before `preload` each time the scene starts — good for resetting values.
- Clamp X so the player stays on screen, accounting for **displayWidth / 2** (origin).
- Use `update(time, delta)` and `speed * delta / 1000` for frame-rate-independent movement.

### What to do

- Wire cursor keys and move left/right by adjusting `player.x`.
- Add left/right screen clamps using half display width.
- Switch speed to pixels-per-second with delta (e.g. ~500).

### Transcript

In the last video, we put a game object on the screen, and we learned how our positioning works. In this video, we're going to make that game object move using our keyboard. No physics, no gravity, no magic, just numbers, input, and control. If you ever feel like movement was confusing or unpredictable, this video will clear that up.

Before we write any code, let's ground this idea. Movement is just changing the position of values over time. If I increase the X value, our game object will move to the right. If I decrease the X value, the game object will move to the left.

That's it. The keyboard just tells us when to change those numbers. Phaser gives us a built-in way to read our keyboard input. Inside our create method, we're going to set this up once.

So, if we do this, input .keyboard we're going to do create cursor keys. This method is going to create an object that'll give us access to our arrow keys on our keyboard, as well as a few other keys. We're not moving anything yet, we're just preparing our input. Think of this like plugging in a controller.

Now, for this object, we'll need to keep track of it on our class, so we'll make a new property. So, we're going to call this cursor keys. Then I'll set equal to that object. Let's add that property to our class.

So, we'll come to the top of our class. We're adding cursor keys. Now, here's an important concept. We don't listen for our key presses once, we want to check our keyboard every frame that our game is running.

That's called input polling. To do that, we need to use our update method. Let's add that method to our class. Now, if you remember, this method is going to be called once for every tick of our game loop, or once every frame.

So, inside this method, this is where we're going to ask questions like, is the left key currently down? And then, based on that answer, we'll change the numbers for our game objects. So, in order to be able to update our game objects, we'll need to create a property to keep track of our player. So, we'll do this.player.

Let's add that to our class. So, now to check for our input, we can do if our cursor keys then on this object, we're going to check our left property. This will be another object, and this has a property called is down. This will tell us if that key is currently being pressed down when our update method runs.

When this key is pressed down, we're going to update our X position for our player, and we're going to decrement it by a value. And now we'll do the same thing if our right key is being pressed down, we'll increase our X value. So, now if we save, we come back over to our game, we press and hold our left or right keys, we'll see our game object is updated, and we can see our player move across our screen. So, instead of hardcoding our movement value, we'll use a variable in our class.

This number will represent how fast our player will be able to move. It's not pixels per second yet, just a value we control. This makes movement easier to tweak later by storing this as a property. So, come to the top of our class, we're going to call this player speed.

speed. Now, to initialize our value, we're going to do this in our init method. So, we'll do init, we'll do this. That player speed equals five.

So, if we come back down to our code, we replace our hardcoded value, and we'll reference our player speed. So, now our init method, this is tied to our Phaser scene life cycle. This works just like our other methods that we previously added, and this is one we haven't talked about yet. So, when our Phaser scene first gets created, Phaser will first call the init method to initialize our scene.

It does this before it calls our preload method, and this method is useful for initializing any values and doing any setup we want to do in our class. It's not where we want to create game objects, we want to do that in our create method. But, this is perfect for things like initializing properties that we want to reset when our Phaser scene starts. So, this method will be invoked each time our Phaser scene starts up.

So, right now our update code works because our update method is going to run for every frame of our game. As long as our key is held down, our number will keep changing. This is why our movement feels continuous. We're not teleporting, we're nudging the position every frame of our game loop.

So, right now our player can leave our screen if we keep holding our left or right keys. That's not great. So, we're going to clamp our position. If our game object goes too far to the left, we'll stop it.

If it goes too far to the right, we'll stop it there as well. To do that, we'll just add code to our update method to keep our player within the bounds of our current game. But, I was going to do an if we'll check our player's position. So, if our X value is less than zero, then we want to reset our player's position.

And now we'll do the same thing for the right side of our screen. So, we're going to do else if if our player's X position is greater than the length of our game, then we'll update our position. So, now if we save and test, we'll see once we reach to a certain position, we can no longer move our player. So, our logic works, but our player is able to still partially go off our screen.

We didn't take into account the origin of our game object. And so, to fix this, we'll want to update our code to include the width of our game object. So, we're going to take our player's X position and we're going to subtract our display width of our game object. Our display width is just how big our game object currently is.

And so, by dividing by two, we'll only subtract half of that. So, then to keep our game object on our screen, we'll want to reset our X value equal to that. So, now if we try moving to our left, once the edge of our jar hits our screen, we no longer leave. So, now for our right-hand side, we want to do the same thing.

So, we're going to copy this value. So, now if our X value plus the half of our width, then we want to take our width and subtract that value. So, now if we move to the right, we'll keep our player within the bounds of our game. One quick note, different computers and devices can run at different frame rates.

To make our movement consistent, we can use delta time. This just means we can scale our movement based on how much time has passed since the last frame. frame. How this works is in our update method, we'll receive two arguments, time and delta.

We'll just log those out. So, we'll do console.log, we'll have time, and we'll have delta. So, now over in our browser, in our developer console, we're going to see our two values get logged. So, time is the amount of time that has passed since our game loop has first started, and then the second argument, our delta, how long has been since our last frame ran.

One thing to note about Phaser is it automatically smooths out our delta value. So, to use our delta, instead of just incrementing by our player speed, we'd want to calculate our new value. So, we do something like const, we'll do move step will be equal to our player speed. And now we want to multiply it by our delta, we'll divide it by a thousand.

So, this will allow us to convert our speed from pixels per frame to pixels per millisecond. So, then we update our X value, we'll just want to add or subtract our move step. So, one thing we'll notice is our game object moves much slower, and that's because we're dividing our value by a thousand. So, now we'd want to update our player speed to be something like 500 pixels, and now have that nice smooth movement.

So, you don't need to master this right now, just know that it exist. We'll use it naturally as we keep building. Now we have a controllable player. In our next video, we'll start spawning our falling objects, and this is where the game really starts to feel like a game.

When you're ready, move on to the next lesson, and we'll keep building.

---

## 04. Spawning Objects Over Time

**Duration:** 9:38 · **Video:** https://www.youtube.com/watch?v=XdUrAfgy5sg

**Goal:** Spawn falling objects on a timer, track them in an array, and clean them up.

### Key concepts

- Game state for many objects = an **array** (`fallingObjects`).
- `spawnFallingObject()`: random X (`Phaser.Math.RND.between`), Y=0, random atlas frame.
- Get frame names via `Object.keys(texture.frames).filter(n => n !== '__BASE')`.
- `Phaser.Utils.Array.GetRandom(frames)` for a random frame.
- `this.time.addEvent({ delay, callback, callbackScope: this, loop: true })` to spawn repeatedly.
- In `update`, move each object down with `y += speed * delta / 1000`.
- Destroy + splice when off-screen; iterate **backwards** when removing in place.
- `depth` controls draw order (player `depth = 1` to appear in front).

### What to do

- Add `spawnFallingObject`, random frames, and `fallingObjects` array.
- Loop a 1s timer to spawn; fall with `fallingObjectSpeed` (~200).
- Clean up off-screen objects; set player depth above falling items.

### Transcript

We have a player that moves. That's great, but it's not a game yet. In this video, we're going to start spawning falling objects. And more importantly, we're going to talk about how games track objects over time.

This is where our game state really starts to matter. So, most games don't deal with one object. They're going to deal with collections of objects. Enemies, bullets, power-ups, obstacles.

The way we manage those is with arrays. An array is just a list of things the gamer is responsible for. And today, falling objects are our first real game state. Let's start by creating a function that will spawn a falling object.

This function will create a game object, position it at the top of our screen, and then give it a falling speed. We're going to separate this into a function because spawning is something we'll want to do repeatedly. So, at the bottom of our game scene, let's add a new method. We're going to call this spawn falling object.

object. So, in this method, we'll want to create one of our falling objects. So, we come into our create method, let's copy our code where we created one of our objects. So, we'll copy this.

We're going to paste that down here. Now, we need to update our X and Y value. For our X value, we'll want this to be a random number between zero and how big our game is. We'll add some padding to keep our object within our scene.

To generate that random number, we can use some of our built-in Phaser utilities. So, we're going to do Phaser .math. RND for random. And now we want to do between.

This allows us to specify a min and max value, and then Phaser is going to return a random number between those two values. So, we're going to do 50, and now we'll reference our scale.width, and we'll subtract 50. So, now for our Y value, let's just set this to be zero. We'll save.

We'll come back up to our create method. Let's replace our line of code here with a call to our function. So, now if we save and refresh our game, we should see we're spawning our object in a different location. So, now when we spawn one of our objects, we'll want to choose one of our random frames from our sprite sheet.

For that, we'll need a way to get a list of all of our possible frame names and choose one of those randomly. To do that, let's add a new property to our class. We're going to call this falling object frames. We'll come down to our create method.

Now, to get our frames, we need to grab our texture from Phaser. To do that, let's do a console log. We're going to do this. That texture is to reference our textures manager.

We use our get method to get a texture. And now we want to grab our texture that is tied to our sprite sheet, which is our asset keys.object. So, now if we save over in our developer console, we'll see we're logging out our texture object. So, this object has a property called frames, and this is an object with all of our frames from our sprite sheet, as well as a base frame that Phaser creates.

So, we'll want to take this object and convert this into an array and subtract out our base frame here. here. Let's reference our falling objects frames property. We're going to set it equal to object.keys.

Object.keys allows us to grab the keys from an object and then create an array from it. So, now we'll want to do our this.textures.get. Our object.keys.frames. And now we want to filter it.

So, we want to filter our array. We're going to filter where our name does not equal underscore underscore base. base. So, now if we update our console log, we'll log out our falling object frames.

Now we have an array with all of our frame names. Nice. So, now what we can do is down in our method, we can choose one of those random frames. For that, we'll use our built-in Phaser utilities.

We're going to do const. We're going to do random frame will be equal to our Phaser utils. utils. We're going to do our array.

We'll do get random. So, now we can pass in our array, and that'll give us a random frame. So, then down here, we can replace our hard-coded frame with our random frame. So, one thing we'll need to do is we need to move our logic for we create our collectible object below where we get our falling object frames.

If we save and come back to our browser and we refresh, we should start seeing different objects getting spawned in our game. Now, here's the important part. We need to remember every falling object that we create. To do that, we need to store them in an array.

That array is going to be our game state, and if that object is in the array, we know it exists in our game, and once it's removed from our array, it'll no longer matter to us. So, to keep track of our objects, we'll add a new property. We're going to call this falling objects. Down in our create method, we'll initialize our array.

So, our falling objects will be equal to an empty array. Now, down in our method, we'll store a reference to our object, and now we add that object to our array. One last thing we'll do is I'm going to console log, and we're going to log out how big our array is. So, we'll do our falling objects.

We'll do our length. And now, we just need a way to spawn our objects over time. For that, we'll use our Phaser time events. How this works is we can tell Phaser to spawn our object after a certain amount of time has passed in our game, and we can have this loop, so then that way we're repeatedly creating our objects.

To do that, we'll do this.time, we'll do add event. event. Now, inside this method, we need to specify an object with the details of our timer event. Our first property is called delay.

So, this is how long we want to wait before we invoke our code. Next is callback, and this is going to be the function that we want to invoke after this delay has passed. So, we're going to reference our spawn falling object method on our class. Next, we're going to specify our callback scope.

Because this method is on our class, we'll pass in this to reference this class. class. Our last property is going to be loop and we'll set it equal to true. So, what this will do is this is going to tell Phaser to wait 1 second.

After a second has passed, we want to invoke our function and then we want this to repeat indefinitely, so we set loop to be true. Now, over in our browser, we're going to start seeing a bunch of game objects starting to spawn and we'll see our array is constantly growing in size. Nice. Nice.

Now that we're spawning our objects, we just need to have them fall down our screen. For that, this will be just like our player, we'll want to update one of our properties in our game object in our update method. Instead of updating our X, we're going to update our Y value to have our game object move down our screen. So, just like our player, we'll want to have a speed for our falling objects.

So, let's add a new property to our class. We'll call this falling object speed. speed. So, down in our init method, we'll set our falling object speed equal to 200.

So, then down in our update method, after the logic for our player, we'll want to loop through our array. So, we're going to do four. We'll do let I equal our falling objects length. We'll subtract one and now if I is greater than or equal to zero, we'll decrement I.

And now inside our loop, we'll get a reference to our object and now we want to update our Y value. So, we do our object.y, we'll do plus equals our falling object speed times our delta on divide by 1,000. So, just like what we did for our player, so now if we save, as we spawn our objects, they'll start falling down our screen. screen.

Now, we just need to add in logic to clean up our game objects once they disappear off our screen. So, right now our array keeps growing indefinitely and eventually this will become a problem. And so, one optimization we can do is we can clean up those objects. To do that, we just want to check our Y value and if that Y value is greater than the height of our game, we can remove that game object.

object. So, if we do our object.y, if it's greater than the height of our game, then we want to destroy that game object. object. And then we want to remove it from our array.

array. So, destroy, this method tells Phaser that we're all done with this game object, and it can clean it up and remove it from our game. This will tell our browser to garbage collect that object, and then it keeps our game running smoothly. So, now we'll see we're spawning our objects consistently, we'll see our array size seems to cap out at four, and now we're not growing indefinitely.

So, one important thing we did here is we iterated over our array backwards. We did that so we could remove our objects in place, so we didn't have to iterate over our array a second time. Finally, one last change we'll do for our game is we're going to update our jar to appear in front of our game objects. So, right now when we spawn our falling objects, they're appearing in front of our player.

By default, when we draw game objects to our scene, they're being drawn in the order that they're created. Because we create our player first, it's always drawn behind our other game objects. If we want our player to appear in front of those game objects, we can update a property called depth on our game object. How this works is that property will tell Phaser that we want to render that game object at a higher level than our other game objects.

By default, all game objects have the same depth of zero, and so by setting our player equal to one, now our game object appears in front of those game objects. And so this is really useful for when you want to do things like UI, and you always want that to appear on top of everything else. Now we have movement, and we have falling objects. In our next video, we'll make them interact.

We'll detect collisions manually using math. No physics engine yet, because if you understand collision at this level, everything else becomes easier.

---

## 05. Collision Detection Without Physics

**Duration:** 4:34 · **Video:** https://www.youtube.com/watch?v=IIaROLpzo0I

**Goal:** Detect jar vs falling-object hits with rectangle math — no physics engine.

### Key concepts

- Collision ≈ "do these two rectangles overlap?"
- `gameObject.getBounds()` returns `{ x, y, width, height }`.
- `Phaser.Geom.Intersects.GetRectangleToRectangle(a, b)` returns overlap points.
- If overlap points `length > 0`, treat as a catch: destroy object and remove from array.
- Same create → update → remove lifecycle as off-screen cleanup; collision just chooses *when* to remove.

### What to do

- Inside the falling-objects loop, after moving, check rectangle intersection with the player.
- On hit: destroy, splice from array, log "Caught item" (score comes next).

### Transcript

Up until now, our objects don't interact. They pass right through each other. In this video, we're going to fix that. And we're going to do it without physics.

No engines, no forces, no magic, just math. Collision detection sounds complicated, but at its core, it's just a question. Do these two shapes overlap? For most beginner games, that shape is a rectangle.

If two rectangles overlap, we say they collided. That's it. So, in Phaser, our game object has what's called a bounding box. That box represents the area it occupies on our screen.

In Phaser, we can ask our game object for its bounds. This gives us a rectangle with an X position, Y position, a width, and a height. This rectangle is just data. No behavior, just numbers.

Once we have the rectangle of that game object, we can then use math to see if those two rectangles overlap or not. Phaser gives us a helper for this. So, if we do console.log, let's log out our player. player.

Now, we use the get bounds method. If we log this out, over in our browser, we have a new object. That object represents that rectangle. We have our width, our height, and our X and Y position of where that rectangle is for our bounding box.

We can use this for both our player and our falling objects to get our two rectangles. So, for our collision logic, we'll want to do that in our update method. So, now to do that in our update method, we'll go into our loop for our falling objects. After we update our object's position, now we'll want to do our check to see if it overlaps with our player.

For that, we can use our built-in Phaser utilities. So, let's do const. We'll do overlap points. We're going to set it equal to Phaser.

Geometry. We'll do intersects. Now, we want to do get rectangle to rectangle. This method expects us to pass two rectangles.

So, we'll do our player. We want to do get bounds. And now for our object, we'll want to do get bounds. Now, we're just going to do a console.log.

We're going to log out those overlapping points. So, now in our browser, we're going to see a bunch of empty arrays being logged. But, once one of our game objects overlaps with our player, then we're going to log out an array, and it's going to have some objects in it. These objects represent the points of where our two rectangles overlap.

So, how this works is for our rectangle, we want to check four things. So, if this represented our jar, and this is our falling object, we want to check to see if our green rectangle, if the right side of it goes past the left side of our jar or other rectangle, we want to check to see if our left side goes past our right side, if our bottom goes past the top, and then if our top went through the bottom. For our falling object game, we only really care about this direction, but the same principle applies. Once we detect that our rectangle overlaps, we have two points where our rectangle is touching.

That array represents those two points of where our rectangles overlap. So, once our array has more than one value, we know that our rectangles are touching. And with that, we know we have a collision. And that's it.

And so, we don't need to write the math ourselves, we can rely on our built-in utility functions of Phaser to get those intersection points. So, now that we have those points, we can just check the array, and so, if our overlapping points, if our length is greater than zero, we know we're overlapping, and so, we would want to remove our object. We'll destroy it, and then we'll remove it from our array. Then, one other thing we'll do, we'll do console log, and we'll just say, "Caught item." item." So, now, as our objects fall, if our object doesn't touch our jar, we don't do anything, but the moment that they overlap, we log our message, and we remove the object from our game.

So, this follows the same lifecycle pattern as before for when our game object moves off our screen. We'll do create, update, and remove. Our collisions don't change our rules, they just decide when a removal will happen. So, what we're doing here, this is important.

Later on, when we look at things like our physics engine, our engine can do this for us automatically. The engine will handle our collision detection, our response, and our edge cases. But, if you don't understand what's happening underneath, they feel like magic. Now, you know it's rectangles, it's math, and it's comparisons.

So, now our game reacts, but there's no feedback yet. In the next video, we'll add scoring and UI. We'll separate our game logic from our presentation, and this will finally start to feel like a real game.

---

## 06. Scoring, UI Text, and Game Feedback

**Duration:** 6:32 · **Video:** https://www.youtube.com/watch?v=U3OdO3CSNz0

**Goal:** Separate game state from UI and show a live score.

### Key concepts

- **Score is state**; on-screen text is only a view of that state — never treat UI as the source of truth.
- `this.score = 0` in `init`; increment on catch (e.g. +10).
- `this.add.text(x, y, string, style)` for UI; style font size, color, stroke.
- Update with `scoreText.setText(...)` after state changes.
- Split prefix ("Score:") and value into two text objects for cleaner updates / localization.
- Pattern: **state changes first, UI reacts second**.

### What to do

- Add `score` and score text UI at top-left.
- On catch, increment score and refresh the value text.

### Transcript

Right now our game works. The player moves, objects fall, and collisions are detected. But something important is missing. There's no feedback.

When we catch an object, nothing really happens. In this video, we're going to fix that by adding scoring and a simple user interface. Before we write any code, I want to explain a concept that will show up in almost every game you build. Game state and user interfaces are not the same thing.

The score variable is a game state. The text on our screen is just a visual representation of that state. If the score changes, the UI updates, but the UI should never be the score itself. This separation helps keep your game logic clean.

So the first step is we need to add a score variable. This is just a number that's going to track how many objects the player has collected. So in our game scene, let's add a new property. We're going to call this score.

score. Down in our init method, we'll initialize it to zero. So then down in our update method, where we have our console log for catching our item, we'll remove that. And now we want to update our score.

So for our score, instead of just doing a single value, we're going to increment it by 10. Later on, this could be modified where different objects are worth different points, but for now, we'll just keep all of our objects the same. So now this property on our class, this is going to be our source of truth. This is how many points our player has at the given time.

The rest of our game is just going to react to it. Now, we need something to display our score. With Phaser, we can use our built-in game objects to do this. We've looked at image and sprite game objects.

Now, we're going to talk about our text game object. Our text game object is a way for us to add text to our game. So in our game scene, let's go to the top of our class. We're going to add a new property to keep track of this game object.

We're going to call this score text game object. So then down at the bottom of our create method, we'll create our game object. So we'll do this. We'll do our score text game object.

We'll do this.add, and now we want to do text. So previously, we used image and sprite to create those game object types. For text game objects, we just use text. Now this follows the same pattern where we want to specify our X and Y.

So we'll do 10 and then 10. And then we specify our string that we want to display. So for this, we're going to do our score prefix with a colon and then our score value. So now if we save, when our browser refreshes, we should see our new text game object.

But it's very small. So for our fourth argument, we can specify an object that allows us to configure the text that we're displaying. This object allows us to do things like our font size, and so we can do like 40 pixels. We can change our color.

So instead of our default white, we'll do 043D. 8C. So now our text is bigger and it's blue. And we can also do things like add a stroke around it, so that way we have an outline.

So we're going to do stroke. For that, we'll do white. So FFF, FFF. And now we're going to define our thickness.

So our stroke thickness will be six. be six. Now we save, we have this nice UI element. So now what we can do is down in our update method, when we update our score, we can update the text that we're displaying.

So we'll do this. Our score text game object, we'll do set text. We'll do our score prefix with our value. So now as we catch our objects, we'll see our text updates with our new score.

Now we have our logic working for updating our score text, we want to make one small enhancement. Right now, we're repeating our prefix of score and we're updating our whole text game object with that each time our score changes. One enhancement we can do is we can break these two things apart. So our prefix is one string and then our score value is another string.

This is a good enhancement because later on, we could do things like add in localization. So then that way we only have to modify our prefix one time, but then our score text wouldn't actually change. So to do that, we'll come back up to our create method and we're going to create a second text game object. So I'm going to copy this block of code here.

We'll go ahead and paste it. For this text game object, we won't need to update it, so let's do a local variable. We're just going to do score text prefix. prefix.

And now we just want to do score with our prefix. And we want to use the same styling. Since we want to use the same styling, we're going to copy that object. We'll remove it.

We'll add that object here. We're going to call this text config. And now we can pass that to both of our game objects. So now that we have our prefix, we need to update our score text game object to be dynamic based on the position of this game object.

So to do that, we'll remove our 10 and our 10. So we'll do our score text prefix. We want to do our X position. And now we want to add in our width.

So what we're doing is we're taking the X position of this game object, we're getting our calculated width, and we're adding that to get our starting X position. So now for our Y, we can do our score text prefix and then our Y value. Now we just need to update our string. So we'll remove our score prefix.

Let's come down to our update method and we'll remove our prefix from there as well. So now we have our score, and as we catch our objects, our score increases. We're only updating this text game object here. here.

And now every time we collect our object, our score increases and our UI reflects it. This pattern is extremely common in game development. Our state will change first and then our UI will react second. Keeping our state separate from our UI makes your game easier to maintain.

For example, later, we could show the score in multiple places. We could have a separate scene for our UI, a HUD. We could have a game over screen where we display the points the player has earned, a leaderboard. All of these can read from the same score variable because the logic and display are not tangled together.

Even a simple score counter adds a lot of feedback. Now the player knows what they're doing right, and the feedback loop is what makes games satisfying. We'll add more polish later, but this is the foundation. Right now, our game can go on forever.

There's no way to win or lose. In the next video, we'll add actual game rules. We'll track missed objects, define a game over condition, and add a way to restart the game. And at that point, we'll have a complete playable loop.

---

## 07. Game Rules, Win Conditions, and Restarting

**Duration:** 7:25 · **Video:** https://www.youtube.com/watch?v=IsnEKi1H-Ys

**Goal:** Add miss limits, game over, lives UI, and click-to-restart.

### Key concepts

- Misses when an object falls off-screen; end when `misses >= maxMisses` (e.g. 3).
- `isGameOver` flag: early-return from `update` when true.
- Store the spawn timer; call `timerEvent.remove()` on game over so spawning stops.
- Show centered "Game Over" text (`setOrigin(0.5)` — text origin defaults to top-left).
- `this.input.once(Phaser.Input.Events.POINTER_DOWN, () => this.scene.restart())`.
- Scene restart resets state cleanly (score, player, objects).
- Lives UI: `maxMisses - misses`, updated on each miss.

### What to do

- Track `misses`, `maxMisses`, `isGameOver`; stop update + timer on game over.
- Add Game Over text and pointer-down restart.
- Add lives display next to score.

### Transcript

Right now our game works pretty well. The player moves, objects fall, collisions increase the score, and we have a simple user interface. But there's one big problem. The game never ends.

You can play forever and nothing meaningful happens. In this video, we're going to add actual game rules. For something to feel like a real game, it needs rules. There has to be a way to succeed and a way to fail.

So in this case, the rule will be simple. The player is trying to catch falling objects. But if they miss too many, the game will end. We already remove objects when they fall off the screen.

Now we'll treat that as a missed catch. So whenever an object reaches the bottom, we'll increase a miss counter. So this will be just like our score variable. At the top of our class, let's add a new property.

We're going to call this misses. Now down in our init method, we'll do this. Our misses will be equal to zero. Now down in our update method, down here where we destroy our game object if it leaves our scene, we'll increase our number of misses.

We'll do this. misses plus equals one. Now outside our for loop, we'll want to see if our number of misses has reached a threshold, and if so, we'll want to end our game. So for that, we'll add another property to our class.

We're going to call this max misses. And while we're here, we'll want a property to keep track if our game is over. So we'll add one more property. We're going to call this is game over.

So now down in our init method, we're going to set our max misses equal to three. three. And then we'll set is game over to be false. false.

So now down in our update method, outside our for loop, we'll do if our number of misses is greater than or equal to our max misses, then we want to end our game. So for ending our game, we're going to add a new method to our class. We're going to call this handle game over. So let's add that method at the bottom of our class.

And the first thing we'll do is we're going to set is game over to be equal to true. true. So now that we're keeping track of when our game should end, we're going to want to add a check to our update method, and if our game is over, then we don't want to run our logic. So, if our game is over, then we want to return early.

So, now if we save, come back to our game, I'm going to catch a few objects. Now, I'm going to let a few objects fall and reach the bottom of our screen, and now we reach our game over state. So, we can no longer move our player, we're no longer moving our objects, but we have one big issue, we're still spawning our objects. So, once our game is over, we want to make sure we shut off our timer event.

To do that, we'll need to keep track of our event. So, I add a new property to our class. I'm going to call this timer event. event.

Now, down in our create method, when we create our event, we want to store that reference. Now, down in our handle game over method, method, we want to reference our timer event, and now we want to do the remove method. Our remove method will tell Phaser to expire our event and remove it from our timer. timer.

So, now what should happen is when our game is over, we should no longer be able to move, our objects stop falling, and now our event no longer fires, so we no longer create new objects. Now, we need feedback for the player. We're going to add a simple text message to the screen that's going to clearly tell the player what happened. So, for that, in our handle game over method, we're going to add a new text game object.

So, we're going to do this.add.text, and now we want to center our text. So, we'll do this.scale.width divided by two. We'll do the same thing for our height, and now we want to display the text game over. over.

So, now for our text, we want to use the same style that we applied to our score. So, let's go to our create method. I'm going to take our variable for our text config, I'm going to move it outside of our create method, and we'll place that outside our class. So, now we have it defined one location, and now we can use that down in our handle game over method.

So, now if we save, once our game is over, we should see our text appear. Nice. However, our text isn't quite centered. What's happening is by default, when you create a text game object in Phaser, our origin for that game object is always in our top left-hand corner.

So, if we want our text to be truly be centered, we need to update our origin to be in the center. So, for that, we can use our set origin method, and we'll pass in 0.5. Finally, we want the player to be able to restart the game. The easiest way to do that is to restart our scene.

And so, we just need to give the player a way to do so. So, for that, we're going to listen for our pointer down event from our input manager. So, if we do this, we'll do input to reference our input plugin. We're going to do once to register an event listener.

Now, we want to do our Phaser, our input, our events. Now, we want to do our pointer down event. event. And then, we have to specify our callback that we want to invoke when this event is fired.

So, when this event is fired, we want to do this.scene.restart. So, what we're doing here is we're telling Phaser, when this event is fired, we want to listen for it one time. When this event is fired, we want to perform this code here, this callback. And all we're doing in our callback is we're telling our scene manager, we want to restart our current scene.

So, how this will work is in our game, once we click on our canvas element, the event will be fired, and then Phaser is going to invoke this code here, which tells us we want to restart our scene. So, now if we play our game, we collect a few objects, we reach our game over state, now our text shows up. If we click, we restart back at the beginning, and our score is reset back to zero. And this works because our scenes manage their own state.

When we restart the scene, Phaser recreates everything. Our player position, score, falling objects, it's a clean reset. Finally, for our game, we're going to add a UI element to represent how many lives the player has left. So, this will be very similar to what we did for our score text game object.

So, we come up to the top of our class, we'll add a new property, we're going to call this lives text game object. Now, down in our create method, let's copy our two blocks of code from where we created our score. We'll paste it. We'll call this lives text prefix.

And then we'll have our lives text game object. We'll update our string, we'll do lives. Let's update our Y position. Let's do 30.

We're going to update our reference here. here. Now, instead of doing our score, we're going to take our max number of misses and subtract our number of misses. Now, if we save, we'll see our new text game object, and we'll see they're overlapping.

So, we're going to update our Y position, we'll do 50. So, now we just need to update our text once our player has a miss. So, we come down to our update method, and now we want to do this, our lives text game object, we'll call set text, we'll take our max misses, and we'll subtract our misses. So, now if we save, if we miss one of our objects, it should go down to two, then we'll go down to one, then once we hit zero, our game is over.

And now, we have a complete game. You can move, catch objects, score points, miss objects, lose, and restart. In the final video of this course, we'll talk about what you learned and where to go next with Phaser.

---

## 08. Where to Go Next (Wrap-Up)

**Duration:** 2:57 · **Video:** https://www.youtube.com/watch?v=BgSDNwg_es8

**Goal:** Review core systems and pick next experiments.

### Key concepts

- You built: scenes, update loop, keyboard input, dynamic spawning, rectangle collisions, score/misses, game over.
- Skipping physics was intentional — learn the math first, then use engines.
- Next ideas: rising difficulty, object types, SFX, animations/particles, title screen / multi-scene flow.
- Further Phaser topics: physics, tilemaps, sprite animation, particles, cameras, multiple scenes.

### What to do

- Modify the finished game (speed, spawn rate, object types).
- Explore Phaser docs and related playlists.

### Transcript

If you followed along through this mini-course, you just built a complete game in Phaser 4. You can move a player, you spawn falling objects, you detect collisions, you track score, and the game ends if the player misses too many objects. That may look simple, but the important part is this: You now understand the core systems that almost every game is built from. Let's quickly look at the core concepts you've learned.

You learned how Phaser organizes games using scenes. Scenes control what appears on the screen and manages your game logic. You use the update loop to move objects and run logic every frame. This is the heartbeat of the game.

You capture keyboard input so the player can move. Input is how the player communicates with your game. You created objects dynamically during gameplay. This is how games create enemies, items, and obstacles.

And you implemented collision detection manually using rectangles. This is important because you now understand the math behind collisions, not just a physics engine. You added score tracking, miss tracking, and game over conditions. And that's what turns mechanics into an actual game.

You might notice something. We didn't use Phaser's physics system, and that was intentional. Physics engines are extremely powerful, but they can also hide how things actually work. By doing our collision detection ourselves, you learned the core idea first.

Once you understand the basics, adding physics becomes much easier. Now comes the fun part. Try modifying the game. Here are some ideas.

Increasing difficulty. You can make objects fall faster over time or spawn more objects the longer the player survives. You could add different object types. So, some objects could give more points, others could remove points, or even end the game instantly.

Add sound effects. Sound makes a huge difference in how a game feels. Try adding sounds when objects are caught or missed. Animations.

You could animate the player or add particle effects when something is collected. And then you can even do things like add a title screen. You could add a transition into the gameplay scene, and that's how many real games structure their flow. If you want to keep learning Phaser, here's some great next topics: physics systems, tile maps and level design, sprite animations, particle effects, camera movement, and multiple scenes.

Those tools let you build much more complex games. The Phaser docs is a great resource, and on my channel, you can find these topics covered in more detail. detail. There's a variety of playlists for learning Phaser, building complete games, and using the Phaser editor.

I recommend checking it out. The most important thing now is to experiment. Don't just copy the tutorials, change things, break things, and add weird ideas. That's how you actually learn game development.

Even small games can teach you a lot. If you made it through this course, you now have a solid starting point with Phaser 4. 4. Keep building, keep experimenting, and most importantly, keep making games.

Thanks for watching.

---

## 09. Updating Phaser 4 to the Latest Version

**Duration:** 4:53 · **Video:** https://www.youtube.com/watch?v=a6BkA9n539Y

**Goal:** Upgrade the Phaser library in this CDN-based starter without breaking imports.

### Key concepts

- NPM Phaser 4 often needs `import * as Phaser from 'phaser'` (not default import) — this course uses **CDN globals** instead.
- Project keeps local `phaser.js` / `phaser.min.js` under assets, plus `phaser.d.ts` for IntelliSense.
- Upgrade by replacing: `assets/.../phaser.js`, `phaser.min.js`, and `source/types/phaser.d.ts` from Phaser GitHub/CDN.
- Repo includes a scripts/`update` shell helper to download a chosen version automatically.

### What to do

- Save latest CDN builds over local Phaser JS files.
- Replace `phaser.d.ts` if you want updated types.
- Confirm new version in the browser Phaser banner (e.g. 4.1).

### Transcript

Hey everyone, if you've been working through the build your first game with Phaser 4 mini course, you may have some questions around how you can upgrade the Phaser library version in this project. So, I've received a number of comments on how you can do this safely and how you can do this without breaking the import for Phaser. So, when Phaser 4 was released, there was a change to the Phaser 4 global import for certain bundlers. Depending on your Phaser setup in your project, you might be importing Phaser from NPM.

And so, you may have had code like this where you do import Phaser from Phaser and other variations. Well, once you upgrade to Phaser 4, this no longer works at this time. You actually need to do a wildcard import instead. You do import asterisk as Phaser from Phaser.

And then this would allow your code to work like it was before. So, for this project, we don't have to do either of those since this is set up to use Phaser from the CDN. And so, Phaser is available as a global import. Where part of the confusion comes in is in our project under our assets folder, we have our JavaScript folder.

Inside here, we have our own copy of the Phaser JavaScript library and the minified version for production. Then in our code under source under lib, we have a phaser.js file where we're just referring to our global Phaser object on the window. Our project is set up this way specifically so we can have our type information while we're working in our JavaScript code. By setting up our project this way, this allows us to do things like in our code when we reference our Phaser scene instance, we actually have intellisense that tells us what our methods do.

Normally, you would have to set this up in a different manner. But with how this project is set up, we're able to use those types that are provided inside our project. And so, back to the original question of how we can actually update our project to use a new version of the Phaser library, it's actually really easy to do. We don't need any build tools, we don't need to change any of our code for the project, we just need to update our Phaser library files.

And so, we just need to swap a couple of these out. There's two main things we need to do. First, we need to get a new version of the Phaser library from the CDN. So, from the Phaser GitHub page, we have some links to the various CDN files.

We'll go to our browser, we paste it in, and now this is our Phaser library from the CDN. We can right click, we can do save as, and now we'll want to save this to our project. So, we'll want to save that to our assets, our JavaScript folder, and we want to replace our phaser.dj s file. So, I'm going to do save, replace, and now for production, we want to do our minified version.

So, we'll do our phaser.min from the same CDN. I'm going to do save as in the same location, assets, JavaScript, we'll replace our phaser.min.js, and now if we save, that's going to update our file. And then finally, if you're using the IntelliSense and you want an updated types file, we need to go into our Phaser code. So, go under types, inside here, we want to go to our phaser.d.ts file, and now if we do raw, this is going to give us that file, and we do right click, we'll do save as, and now for our types, we want to place that in our source folder under types, and we'll replace our phaser.d.ts file.

Want to make sure we have the right extension, we do save, we'll replace our file, and now if we come back to our project, we'll see the only three files we modified was our phaser.dj s, our min, and then our types file. So, now if we start our project, we come back to our browser, go into our developer tools, we'll see now from our Phaser banner, we're now using the latest version of Phaser at this time, so Phaser 4.1. And that's it. So, for this particular project, if you want to update your version of Phaser and change it, you just need to modify those files.

The main files are going to be your phaser.js, your phaser.min.js, and then if you're using the IntelliSense, you want to update your phaser.d.ts file. And finally, if you're comfortable with the command line and you want to make this even easier in the future, the GitHub repo for this project now includes an update script in the scripts folder. In the readme, there's details on how to run the script, and all you need to do is run the shell script and then specify which version of the Phaser library you want to use, and this will download those files and update your project for you automatically. And that's it.

Really hope that clears things up for anyone who's been confused about the version. Both URLs in the description, so you can grab them without having to pause the video. If you have any questions, the comments are open, or you can open a GitHub discussion on the repo. Thank you so much for watching, and I'll see you in the next one.


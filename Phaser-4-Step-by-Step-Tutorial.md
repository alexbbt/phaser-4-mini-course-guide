# Build Your First Game with Phaser 4 — Step-by-Step Tutorial

A written version of the [YouTube mini course](https://www.youtube.com/playlist?list=PLmcXe0-sfoSheQinG8d5JQBiTqiKnBgo7).

Each lesson matches an official GitHub tag from [phaser-4-falling-objects-game](https://github.com/devshareacademy/phaser-4-falling-objects-game). Snippets below are **only the new or changed lines** from that step’s diff (file: `src/scenes/game-scene.js` unless noted).

| Lesson | Video | GitHub tag |
|--------|-------|------------|
| 1 | Project Setup & Core Concepts | `0-initial-project` |
| 2 | Coordinates & Positioning | `1-images` |
| 3 | Player Movement | `2-player-movement` |
| 4 | Spawning Objects | `3-falling-objects` |
| 5 | Collision Detection | `4-collisions` |
| 6 | Scoring & UI | `5-score` |
| 7 | Game Over & Restart | `6-game-over` |
| 8 | Wrap-up | — |
| 9 | Updating Phaser | `phaser-ver-4.1.0` |

---

## Prerequisites

- Modern browser + editor (VS Code works well)
- Course starter at tag `0-initial-project`
- Local server from the project root (folder with `index.html`)

**VS Code:** [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) → Go Live  
**Python:** `python3 -m http.server 8080` → `http://localhost:8080/`  
**Node:** `npx http-server`

Confirm the Phaser banner appears in the browser console.

### Project map

| Path | Role |
|------|------|
| `src/main.js` | Creates `Phaser.Game`, registers scenes |
| `src/scenes/preload-scene.js` | Loads assets → starts `GameScene` |
| `src/scenes/game-scene.js` | Gameplay (almost all edits) |
| `src/common/assets.js` | Asset keys |

Scene lifecycle: `init` (reset) → `preload` (load) → `create` (build once) → `update` (every frame).

---

## 1. Setup & how Phaser works

**Tag:** `0-initial-project` · **Goal:** Run the starter and understand the loop. No gameplay edits yet.

### Already in `main.js`

```js
const game = new Phaser.Game(gameConfig);

game.scene.add(SCENE_KEYS.PRELOAD_SCENE, PreloadScene);
game.scene.add(SCENE_KEYS.GAME_SCENE, GameScene);
game.scene.start(SCENE_KEYS.PRELOAD_SCENE);
```

### Already in `PreloadScene`

```js
preload() {
  IMAGE_ASSETS.forEach((asset) => {
    this.load.image(asset.assetKey, asset.path);
  });
  TEXTURE_ATLAS_ASSETS.forEach((asset) => {
    this.load.atlas(asset.assetKey, asset.textureURL, asset.atlasURL);
  });
}

create() {
  this.scene.start(SCENE_KEYS.GAME_SCENE);
}
```

### Starting `GameScene.create()`

```js
create() {
  this.add.image(this.scale.width / 2, this.scale.height / 2, ASSET_KEYS.BACKGROUND);
}
```

### Try it — temporary logs

```js
preload() {
  console.log('preload');
}

create() {
  console.log('create');
}

update() {
  console.log('update');
}
```

You should see `preload` / `create` once and `update` repeatedly. Remove the logs when done.

**Checkpoint:** Game runs; you know preload → create → update.

---

## 2. Coordinates, assets & first sprites

**Tag:** `1-images` · **Goal:** Add jar + candy; clarify X/Y and origin.

World origin is top-left `(0,0)`. X goes right, Y goes down. Object **origin defaults to center**.

### In `create()` — replace body with

```js
const { width, height } = this.scale;

this.add.image(width / 2, height / 2, ASSET_KEYS.BACKGROUND);
this.add.image(width / 2, height, ASSET_KEYS.JAR);
this.add.image(width / 2, height / 2, ASSET_KEYS.OBJECTS, 'button1.png').setScale(0.75);
```

**Checkpoint:** Background, jar at bottom center, scaled candy in the middle.

---

## 3. Player movement with the keyboard

**Tag:** `2-player-movement` · **Goal:** Move jar with ←/→; clamp to screen; use delta time.

### Add fields on the class

```js
/** @type {Phaser.Types.Input.Keyboard.CursorKeys} handles player input */
#cursorKeys;
/** @type {Phaser.GameObjects.Image} the player (jar) in our game */
#player;
/** @type {number} how fast our player can move in our game */
#playerSpeed;
```

### Add `init()`

```js
init() {
  this.#playerSpeed = 500;
}
```

### In `create()` — new / changed lines

```js
if (!this.input) {
  console.warn('Input plugin is not available');
  return;
}
```

```js
this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR);
```

```js
this.#cursorKeys = this.input.keyboard.createCursorKeys();
```

(Keep the temporary candy line for now; lesson 4 removes it.)

### Add `update(time, delta)`

```js
update(time, delta) {
  const moveStep = this.#playerSpeed * (delta / 1000);
  if (this.#cursorKeys.left.isDown) {
    this.#player.x -= moveStep;
  } else if (this.#cursorKeys.right.isDown) {
    this.#player.x += moveStep;
  }

  if (this.#player.x - this.#player.displayWidth / 2 < 0) {
    this.#player.x = this.#player.displayWidth / 2;
  } else if (this.#player.x + this.#player.displayWidth / 2 > this.scale.width) {
    this.#player.x = this.scale.width - this.#player.displayWidth / 2;
  }
}
```

**Checkpoint:** Smooth left/right movement; jar stays fully on screen.

---

## 4. Spawning falling objects

**Tag:** `3-falling-objects` · **Goal:** Timer spawn, array state, fall, off-screen cleanup.

### Add fields

```js
/** @type {Phaser.GameObjects.Image[]} the falling objects for the player to collect */
#fallingObjects;
/** @type {string[]} the list of frames from the falling object spritesheet that we loaded in */
#fallingObjectFrames;
/** @type {number} how fast the objects will fall */
#fallingObjectsSpeed;
```

### In `init()` — add

```js
this.#fallingObjectsSpeed = 200;
```

### In `create()` — change player line

```js
this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR).setDepth(1);
```

### In `create()` — remove the static candy line

Delete:

```js
this.add.image(width / 2, height / 2, ASSET_KEYS.OBJECTS, 'button1.png').setScale(0.75);
```

### In `create()` — after cursor keys, add

```js
this.#fallingObjects = [];
this.#fallingObjectFrames = Object.keys(this.textures.get(ASSET_KEYS.OBJECTS).frames).filter(
  (name) => name !== '__BASE',
);

this.time.addEvent({
  delay: 1000,
  callback: this.#spawnFallingObject,
  callbackScope: this,
  loop: true,
});
```

### In `update()` — after player clamp, add

```js
for (let i = this.#fallingObjects.length - 1; i >= 0; i--) {
  const obj = this.#fallingObjects[i];
  obj.y += this.#fallingObjectsSpeed * (delta / 1000);

  if (obj.y > this.scale.height) {
    obj.destroy();
    this.#fallingObjects.splice(i, 1);
  }
}
```

### Add method

```js
#spawnFallingObject() {
  const randomFrame = Phaser.Utils.Array.GetRandom(this.#fallingObjectFrames);
  const obj = this.add
    .image(Phaser.Math.RND.between(50, this.scale.width - 50), 0, ASSET_KEYS.OBJECTS, randomFrame)
    .setScale(0.75);
  this.#fallingObjects.push(obj);
}
```

**Checkpoint:** Candies spawn every second, fall, and clean up off-screen.

---

## 5. Collision detection (no physics)

**Tag:** `4-collisions` · **Goal:** Catch when jar and candy rectangles overlap.

### In the falling-object loop — after `obj.y += …`, before the off-screen check

```js
const overlapPoints = Phaser.Geom.Intersects.GetRectangleToRectangle(
  this.#player.getBounds(),
  obj.getBounds(),
);
if (overlapPoints.length > 0) {
  obj.destroy();
  this.#fallingObjects.splice(i, 1);
  console.log('caught item');
}
```

**Checkpoint:** Overlap logs `caught item` and removes the candy.

---

## 6. Scoring & UI text

**Tag:** `5-score` · **Goal:** Score state + on-screen text; update on catch.

### Add fields

```js
#score;
#scoreTextGameObject;
```

### In `init()` — add

```js
this.#score = 0;
```

### At end of `create()` — add HUD

```js
const textConfig = {
  fontSize: '40px',
  color: '#043D8C',
  stroke: '#ffffff',
  strokeThickness: 6,
};
const scoreTextPrefix = this.add.text(10, 10, 'Score:', textConfig);
this.#scoreTextGameObject = this.add.text(
  scoreTextPrefix.x + scoreTextPrefix.width,
  scoreTextPrefix.y,
  `${this.#score}`,
  textConfig,
);
```

### On catch — replace `console.log('caught item')` with

```js
this.#score += 10;
this.#scoreTextGameObject.setText(`${this.#score}`);
```

**Checkpoint:** Each catch adds 10 and updates the score text.

---

## 7. Misses, game over & restart

**Tag:** `6-game-over` · **Goal:** Lives, stop on 3 misses, Game Over, click to restart.

### Move `textConfig` to module scope (above the class)

Cut it out of `create()` and place:

```js
const textConfig = {
  fontSize: '40px',
  color: '#043D8C',
  stroke: '#ffffff',
  strokeThickness: 6,
};
```

### Add fields

```js
/** @type {number} how many times the player has missed collecting the falling objects */
#misses;
/** @type {number} how many misses before the game ends */
#maxMisses;
/** @type {boolean} tracks if the game over */
#isGameOver;
/** @type {Phaser.Time.TimerEvent} timer for spawning objects in the game */
#timerEvent;
/** @type {Phaser.GameObjects.Text} the visual representation of the players lives */
#livesTextGameObject;
```

### In `init()` — add

```js
this.#misses = 0;
this.#maxMisses = 3;
this.#isGameOver = false;
```

### In `create()` — store the timer

Change:

```js
this.time.addEvent({
```

to:

```js
this.#timerEvent = this.time.addEvent({
```

### At end of `create()` — lives HUD

```js
const livesTextPrefix = this.add.text(10, 50, 'Lives:', textConfig);
this.#livesTextGameObject = this.add.text(
  livesTextPrefix.x + livesTextPrefix.width,
  livesTextPrefix.y,
  `${this.#maxMisses - this.#misses}`,
  textConfig,
);
```

### At start of `update()` — early return

```js
if (this.#isGameOver) {
  return;
}
```

### On catch — after updating score, add

```js
continue;
```

### On off-screen miss — after destroy/splice, add

```js
this.#misses += 1;
this.#livesTextGameObject.setText(`${this.#maxMisses - this.#misses}`);
```

### After the falling-object loop — add

```js
if (this.#misses >= this.#maxMisses) {
  this.#handleGameOver();
}
```

### Add method

```js
#handleGameOver() {
  this.#isGameOver = true;
  this.#timerEvent.remove();
  this.add
    .text(this.scale.width / 2, this.scale.height / 2, 'Game Over', textConfig)
    .setOrigin(0.5);
  this.input.once(Phaser.Input.Events.POINTER_DOWN, () => {
    this.scene.restart();
  });
}
```

**Checkpoint:** Three misses → Game Over, no movement/spawns; click restarts cleanly.

---

## 8. Where to go next

You built: scenes, update loop, keyboard input, dynamic spawning, rectangle collisions, score/lives, game over + restart.

Ideas: rising difficulty, item types, SFX, title scene, particles.  
Next Phaser topics: physics, tilemaps, animations, cameras.

---

## 9. Updating Phaser

**Tag:** `phaser-ver-4.1.0`

This project uses local CDN copies, not npm. Replace:

1. `assets/js/phaser.js`
2. `assets/js/phaser.min.js`
3. `src/types/phaser.d.ts` (for IntelliSense)

Then refresh and check the console banner. The repo `scripts/` folder can automate this — see the project README.

If you use npm elsewhere: `import * as Phaser from 'phaser'` (wildcard), not a default import.

---

## Playtest checklist

| Action | Expected |
|--------|----------|
| ← / → | Jar moves; stays on screen |
| Spawns | ~1/sec, random X / frame |
| Catch | Candy gone; score +10 |
| Miss | Lives −1 |
| 3 misses | Game Over; stopped |
| Click | Restart; state reset |

Compare your file anytime against the matching tag, e.g.  
`git show 6-game-over:src/scenes/game-scene.js`

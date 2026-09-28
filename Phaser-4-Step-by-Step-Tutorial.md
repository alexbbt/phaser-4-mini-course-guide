# Build Your First Game with Phaser 4 — Step-by-Step Tutorial

A written version of the [YouTube mini course](https://www.youtube.com/playlist?list=PLmcXe0-sfoSheQinG8d5JQBiTqiKnBgo7). You’ll build a simple catch game: move a jar left/right, catch falling candies, score points, and lose after three misses.

Code matches the official starter: [phaser-4-falling-objects-game](https://github.com/devshareacademy/phaser-4-falling-objects-game). Most edits go in `src/scenes/game-scene.js`. Each step shows **only what to add or change**.

---

## Contents

1. [Setup & how Phaser works](#1-setup--how-phaser-works)
2. [Coordinates, assets & first sprites](#2-coordinates-assets--first-sprites)
3. [Player movement with the keyboard](#3-player-movement-with-the-keyboard)
4. [Spawning falling objects](#4-spawning-falling-objects)
5. [Collision detection (no physics)](#5-collision-detection-no-physics)
6. [Score & UI text](#6-score--ui-text)
7. [Misses, game over & restart](#7-misses-game-over--restart)
8. [Where to go next](#8-where-to-go-next)
9. [Updating Phaser](#9-updating-phaser)

---

## Prerequisites

- A modern browser
- [VS Code](https://code.visualstudio.com/) (or any editor)
- A local web server
- The course starter project unzipped

Open the project folder and start a server from the **project root** (the folder with `index.html`).

**VS Code:** [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) → **Go Live**.

**Python:** `python3 -m http.server 8080` → open `http://localhost:8080/`

**Node:** `npx http-server`

Confirm the canvas loads and the console shows the Phaser banner.

---

## Project map

| Path | Role |
|------|------|
| `index.html` | Loads Phaser + `src/main.js` |
| `src/main.js` | Creates `Phaser.Game`, registers scenes |
| `src/scenes/preload-scene.js` | Loads assets, starts gameplay |
| `src/scenes/game-scene.js` | All gameplay (edit this) |
| `src/common/assets.js` | Asset keys and paths |
| `src/common/scene-keys.js` | Scene name constants |

Scene lifecycle:

| Method | When | For |
|--------|------|-----|
| `init()` | Once per scene start | Reset numbers / flags |
| `preload()` | Once before create | Load assets |
| `create()` | Once after load | Build world, UI, input, timers |
| `update(time, delta)` | Every frame | Movement, input, collisions |

---

## 1. Setup & how Phaser works

### Goal

Understand bootstrap and prove the game loop is alive. You don’t need to rewrite these files — the starter already has them.

### What `main.js` does (read only)

Creates the game and starts preload:

```js
const game = new Phaser.Game(gameConfig);

game.scene.add(SCENE_KEYS.PRELOAD_SCENE, PreloadScene);
game.scene.add(SCENE_KEYS.GAME_SCENE, GameScene);
game.scene.start(SCENE_KEYS.PRELOAD_SCENE);
```

### What `PreloadScene` does (read only)

Loads images/atlas, then jumps to gameplay:

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

### Try it: log the loop

In `game-scene.js`, temporarily add:

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

Refresh: `preload` / `create` once, `update` repeatedly. Remove the logs when done.

**Checkpoint:** Server running, Phaser banner visible, you understand preload → create → update.

---

## 2. Coordinates, assets & first sprites

### Goal

Draw the background, jar, and one candy. Understand X/Y and origin.

### Coordinates (concept)

- World origin is **top-left** `(0, 0)`.
- **X** → right, **Y** → down.
- Default object **origin is the center** — `(0, 0)` parks the *center* on the corner.

### In `create()` — get size + draw sprites

```js
const { width, height } = this.scale;

this.add.image(width / 2, height / 2, ASSET_KEYS.BACKGROUND);
this.add.image(width / 2, height, ASSET_KEYS.JAR);
this.add
  .image(width / 2, height / 2, ASSET_KEYS.OBJECTS, 'button1.png')
  .setScale(0.75);
```

Asset keys come from `src/common/assets.js` (`BACKGROUND`, `JAR`, `OBJECTS`). The key in `load` must match `add.image`.

- Omit the atlas frame → Phaser uses the first frame.
- `.setScale(0.75)` shrinks; `1` = original; `2` = double.

**Checkpoint:** Background fills the view; jar at bottom center; a candy in the middle.

---

## 3. Player movement with the keyboard

### Goal

Move the jar with ← / →, stay on screen, use delta time.

### Add fields on the class

```js
/** @type {Phaser.Types.Input.Keyboard.CursorKeys} */
#cursorKeys;
/** @type {Phaser.GameObjects.Image} */
#player;
/** @type {number} pixels per second */
#playerSpeed;
```

### Add `init()` — reset speed each scene start

```js
init() {
  this.#playerSpeed = 500;
}
```

### In `create()` — keep a player ref + cursor keys

Change the jar line to store it, then set up input:

```js
this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR);
this.#cursorKeys = this.input.keyboard.createCursorKeys();
```

(You can remove the temporary centered candy from lesson 2.)

### Add `update(time, delta)` — move + clamp

```js
update(time, delta) {
  const moveStep = this.#playerSpeed * (delta / 1000);

  if (this.#cursorKeys.left.isDown) {
    this.#player.x -= moveStep;
  } else if (this.#cursorKeys.right.isDown) {
    this.#player.x += moveStep;
  }

  const halfW = this.#player.displayWidth / 2;
  if (this.#player.x - halfW < 0) {
    this.#player.x = halfW;
  } else if (this.#player.x + halfW > this.scale.width) {
    this.#player.x = this.scale.width - halfW;
  }
}
```

Clamp with half width because position is the **center** of the jar.

**Checkpoint:** Smooth left/right movement; jar stays on screen.

---

## 4. Spawning falling objects

### Goal

Spawn on a timer, track in an array, fall downward, clean up off-screen.

### Add fields

```js
/** @type {Phaser.GameObjects.Image[]} */
#fallingObjects;
/** @type {string[]} */
#fallingObjectFrames;
/** @type {number} */
#fallingObjectsSpeed;
```

### In `init()` — add

```js
this.#fallingObjectsSpeed = 200;
```

### In `create()` — draw jar in front, then set up spawn state

```js
this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR).setDepth(1);
```

```js
this.#fallingObjects = [];

this.#fallingObjectFrames = Object.keys(
  this.textures.get(ASSET_KEYS.OBJECTS).frames,
).filter((name) => name !== '__BASE');

this.time.addEvent({
  delay: 1000,
  callback: this.#spawnFallingObject,
  callbackScope: this,
  loop: true,
});
```

### Add method — spawn one candy

```js
#spawnFallingObject() {
  const randomFrame = Phaser.Utils.Array.GetRandom(this.#fallingObjectFrames);
  const x = Phaser.Math.RND.between(50, this.scale.width - 50);

  const obj = this.add
    .image(x, 0, ASSET_KEYS.OBJECTS, randomFrame)
    .setScale(0.75);

  this.#fallingObjects.push(obj);
}
```

### In `update()` — after player movement, fall + cleanup

Loop **backwards** so `splice` is safe:

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

**Checkpoint:** Candies spawn every second, fall, and leave without the array growing forever.

---

## 5. Collision detection (no physics)

### Goal

When candy overlaps jar, catch it (destroy + remove from array).

### Idea

```js
const overlapPoints = Phaser.Geom.Intersects.GetRectangleToRectangle(
  this.#player.getBounds(),
  obj.getBounds(),
);

if (overlapPoints.length > 0) {
  // overlapping
}
```

### In the falling-object loop — after `obj.y += …`, before off-screen check

```js
const overlapPoints = Phaser.Geom.Intersects.GetRectangleToRectangle(
  this.#player.getBounds(),
  obj.getBounds(),
);

if (overlapPoints.length > 0) {
  obj.destroy();
  this.#fallingObjects.splice(i, 1);
  console.log('Caught item');
  continue;
}
```

`continue` skips the miss / off-screen branch for that object.

**Checkpoint:** Catch removes the candy; misses still fall off and clean up.

---

## 6. Score & UI text

### Goal

Score is **state**; text only mirrors it.

### Add fields

```js
/** @type {number} */
#score;
/** @type {Phaser.GameObjects.Text} */
#scoreTextGameObject;
```

### In `init()` — add

```js
this.#score = 0;
```

### At module top (outside the class) — shared style

```js
const textConfig = {
  fontSize: '40px',
  color: '#043D8C',
  stroke: '#ffffff',
  strokeThickness: 6,
};
```

### At end of `create()` — score HUD

```js
const scoreTextPrefix = this.add.text(10, 10, 'Score:', textConfig);
this.#scoreTextGameObject = this.add.text(
  scoreTextPrefix.x + scoreTextPrefix.width,
  scoreTextPrefix.y,
  `${this.#score}`,
  textConfig,
);
```

### On catch — replace the `console.log`

```js
this.#score += 10;
this.#scoreTextGameObject.setText(`${this.#score}`);
```

**Checkpoint:** Each catch adds 10; the on-screen number updates.

---

## 7. Misses, game over & restart

### Goal

Three misses → stop play, show Game Over, click to restart.

### Add fields

```js
/** @type {number} */
#misses;
/** @type {number} */
#maxMisses;
/** @type {boolean} */
#isGameOver;
/** @type {Phaser.Time.TimerEvent} */
#timerEvent;
/** @type {Phaser.GameObjects.Text} */
#livesTextGameObject;
```

### In `init()` — add

```js
this.#misses = 0;
this.#maxMisses = 3;
this.#isGameOver = false;
```

### In `create()` — store the timer (change the existing `addEvent` call)

```js
this.#timerEvent = this.time.addEvent({
  delay: 1000,
  callback: this.#spawnFallingObject,
  callbackScope: this,
  loop: true,
});
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

### At start of `update()` — bail when over

```js
if (this.#isGameOver) {
  return;
}
```

### When an object falls off-screen — count a miss

```js
this.#misses += 1;
this.#livesTextGameObject.setText(`${this.#maxMisses - this.#misses}`);
```

### After the falling-object loop — check game over

```js
if (this.#misses >= this.#maxMisses) {
  this.#handleGameOver();
}
```

### Add method — game over + click to restart

Text defaults to top-left origin; `setOrigin(0.5)` centers it.

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

**Checkpoint:** Three misses stop movement/spawning and show Game Over; click resets everything.

---

## 8. Where to go next

| System | What you built |
|--------|----------------|
| Scenes | Preload → Game |
| Game loop | `update` every frame |
| Input | Cursor keys |
| Dynamic objects | Timer + array |
| Collision | Rectangle overlap |
| State + UI | Score / lives |
| Rules | Miss limit + restart |

Ideas: rising difficulty, good/bad item types, SFX, title scene, particles.

Next Phaser topics: physics, tilemaps, sprite animations, cameras, particles.

---

## 9. Updating Phaser

This starter uses local files under `assets/js/`, not npm.

1. Replace `assets/js/phaser.js` and `assets/js/phaser.min.js` from the [Phaser release](https://github.com/phaserjs/phaser).
2. Replace `src/types/phaser.d.ts` for updated IntelliSense.
3. Refresh and check the console banner.

If you use npm elsewhere, Phaser 4 often needs `import * as Phaser from 'phaser'` instead of a default import. This course project doesn’t.

---

## Controls & playtest checklist

| Action | Expected |
|--------|----------|
| ← / → | Jar moves smoothly, stays on screen |
| Candy spawns | ~once/sec, random X / frame |
| Catch | Candy gone, score +10 |
| Miss | Lives decrease |
| 3 misses | Game Over; no move/spawn |
| Click | Restart; score/lives reset |

Enjoy building — change numbers, break things, add weird ideas.

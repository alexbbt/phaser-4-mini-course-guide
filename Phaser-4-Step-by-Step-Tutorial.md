# Build Your First Game with Phaser 4 — Step-by-Step Tutorial

A written version of the [YouTube mini course](https://www.youtube.com/playlist?list=PLmcXe0-sfoSheQinG8d5JQBiTqiKnBgo7). You’ll build a simple catch game: move a jar left/right, catch falling candies, score points, and lose after three misses.

Code matches the official starter: [phaser-4-falling-objects-game](https://github.com/devshareacademy/phaser-4-falling-objects-game). Private fields (`#player`) are used the same way as the finished project.

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
10. [Complete `GameScene` reference](#10-complete-gamescene-reference)

---

## Prerequisites

- A modern browser
- [VS Code](https://code.visualstudio.com/) (or any editor)
- A local web server (Live Server extension, or Python / Node — see below)
- The course starter project unzipped somewhere convenient

Open the project folder and start a server from the **project root** (the folder that contains `index.html`).

**VS Code:** install [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) → **Go Live**.

**Python:**

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080/`.

**Node:**

```bash
npx http-server
```

Confirm the game canvas loads and the browser console shows the Phaser banner (version number).

---

## Project map

| Path | Role |
|------|------|
| `index.html` | Loads Phaser + `src/main.js` |
| `src/main.js` | Creates `Phaser.Game` and registers scenes |
| `src/scenes/preload-scene.js` | Loads images / atlas, then starts gameplay |
| `src/scenes/game-scene.js` | All gameplay (you edit this most) |
| `src/common/assets.js` | Asset keys and file paths |
| `src/common/scene-keys.js` | Scene name constants |
| `assets/images/` | Background, jar, spritesheet + JSON |

Almost everything interesting happens in **scenes**. Each scene plugs into Phaser’s loop:

| Method | When it runs | What it’s for |
|--------|----------------|---------------|
| `init()` | Once when the scene starts | Reset numbers / flags |
| `preload()` | Once before create | Load images, audio, atlases |
| `create()` | Once after assets are ready | Build world, UI, input, timers |
| `update(time, delta)` | Every frame (~60×/sec) | Move things, poll input, check collisions |

This project splits loading into `PreloadScene`, then starts `GameScene` for play.

---

## 1. Setup & how Phaser works

### Goal

Understand the bootstrap files and prove the game loop is alive.

### Bootstrap: `main.js`

```js
import Phaser from './lib/phaser.js';
import { SCENE_KEYS } from './common/scene-keys.js';
import { GameScene } from './scenes/game-scene.js';
import { PreloadScene } from './scenes/preload-scene.js';

/** @type {Phaser.Types.Core.GameConfig} */
const gameConfig = {
  type: Phaser.AUTO,
  pixelArt: false,
  title: 'Crafty Catch',
  scale: {
    parent: 'game-container',
    width: 1280,
    height: 720,
    autoCenter: Phaser.Scale.CENTER_BOTH,
    mode: Phaser.Scale.FIT,
  },
  backgroundColor: '#000000',
};

const game = new Phaser.Game(gameConfig);

game.scene.add(SCENE_KEYS.PRELOAD_SCENE, PreloadScene);
game.scene.add(SCENE_KEYS.GAME_SCENE, GameScene);
game.scene.start(SCENE_KEYS.PRELOAD_SCENE);
```

`new Phaser.Game(config)` creates the canvas. Scenes are registered, then `PRELOAD_SCENE` starts.

### Preload scene

```js
import Phaser from '../lib/phaser.js';
import { SCENE_KEYS } from '../common/scene-keys.js';
import { IMAGE_ASSETS, TEXTURE_ATLAS_ASSETS } from '../common/assets.js';

export class PreloadScene extends Phaser.Scene {
  constructor() {
    super({ key: SCENE_KEYS.PRELOAD_SCENE });
  }

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

Refresh: `preload` and `create` once, `update` repeatedly. Remove the logs when you’re done.

**Checkpoint:** Server running, Phaser banner in console, you understand preload → create → update.

---

## 2. Coordinates, assets & first sprites

### Goal

Draw the background, jar (player), and one falling candy. Understand X/Y and origin.

### Coordinates

- Origin of the **world** is the **top-left** of the canvas: `(0, 0)`.
- **X** increases to the right; **Y** increases downward.
- By default a game object’s **origin is its center**. Placing an image at `(0, 0)` puts its *center* on the corner — you only see part of it.

Center of the screen:

```js
const { width, height } = this.scale;
const cx = width / 2;
const cy = height / 2;
```

### Asset keys

From `src/common/assets.js`:

```js
export const ASSET_KEYS = Object.freeze({
  BACKGROUND: 'BACKGROUND',
  OBJECTS: 'OBJECTS',
  JAR: 'JAR',
});
```

Keys used in `load.*` must match keys used in `add.image`.

### Build the world in `create()`

Start with a minimal `GameScene`:

```js
import Phaser from '../lib/phaser.js';
import { SCENE_KEYS } from '../common/scene-keys.js';
import { ASSET_KEYS } from '../common/assets.js';

export class GameScene extends Phaser.Scene {
  constructor() {
    super({ key: SCENE_KEYS.GAME_SCENE });
  }

  create() {
    const { width, height } = this.scale;

    // Full-screen background (origin at center → use width/2, height/2)
    this.add.image(width / 2, height / 2, ASSET_KEYS.BACKGROUND);

    // Player jar along the bottom edge
    this.add.image(width / 2, height, ASSET_KEYS.JAR);

    // One candy from the texture atlas (frame name from spritesheet.json)
    this.add
      .image(width / 2, height / 2, ASSET_KEYS.OBJECTS, 'button1.png')
      .setScale(0.75);
  }
}
```

Notes:

- Omitting the atlas frame uses Phaser’s first frame.
- `.setScale(0.75)` shrinks; `1` is original size; `2` doubles.
- Wrong texture key → broken / missing image.

**Checkpoint:** Background fills the view; jar at bottom center; a candy in the middle.

---

## 3. Player movement with the keyboard

### Goal

Move the jar with ← / →, stay on screen, move smoothly with delta time.

### Properties + `init()`

```js
export class GameScene extends Phaser.Scene {
  /** @type {Phaser.Types.Input.Keyboard.CursorKeys} */
  #cursorKeys;
  /** @type {Phaser.GameObjects.Image} */
  #player;
  /** @type {number} pixels per second */
  #playerSpeed;

  constructor() {
    super({ key: SCENE_KEYS.GAME_SCENE });
  }

  init() {
    // Runs every time the scene starts/restarts — reset values here
    this.#playerSpeed = 500;
  }

  create() {
    const { width, height } = this.scale;

    this.add.image(width / 2, height / 2, ASSET_KEYS.BACKGROUND);
    this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR);

    // Arrow keys (+ space/shift helpers Phaser includes)
    this.#cursorKeys = this.input.keyboard.createCursorKeys();
  }

  update(time, delta) {
    // delta = ms since last frame; scale speed to pixels/second
    const moveStep = this.#playerSpeed * (delta / 1000);

    if (this.#cursorKeys.left.isDown) {
      this.#player.x -= moveStep;
    } else if (this.#cursorKeys.right.isDown) {
      this.#player.x += moveStep;
    }

    // Clamp using half display width (origin is center)
    const halfW = this.#player.displayWidth / 2;
    if (this.#player.x - halfW < 0) {
      this.#player.x = halfW;
    } else if (this.#player.x + halfW > this.scale.width) {
      this.#player.x = this.scale.width - halfW;
    }
  }
}
```

Why clamp with `displayWidth / 2`? Because position is the **center**. Stopping at `x = 0` would still let half the jar hang off-screen.

**Checkpoint:** Hold left/right — smooth movement; jar cannot leave the screen.

---

## 4. Spawning falling objects

### Goal

Spawn candies on a timer, track them in an array, move them down, destroy when off-screen.

### More properties

```js
/** @type {Phaser.GameObjects.Image[]} */
#fallingObjects;
/** @type {string[]} */
#fallingObjectFrames;
/** @type {number} */
#fallingObjectsSpeed;
```

In `init()`:

```js
this.#fallingObjectsSpeed = 200;
```

### Frame list + timer in `create()`

```js
create() {
  const { width, height } = this.scale;

  this.add.image(width / 2, height / 2, ASSET_KEYS.BACKGROUND);
  this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR).setDepth(1);
  this.#cursorKeys = this.input.keyboard.createCursorKeys();

  this.#fallingObjects = [];

  // All atlas frame names except Phaser's internal __BASE
  this.#fallingObjectFrames = Object.keys(
    this.textures.get(ASSET_KEYS.OBJECTS).frames,
  ).filter((name) => name !== '__BASE');

  this.time.addEvent({
    delay: 1000, // ms
    callback: this.#spawnFallingObject,
    callbackScope: this,
    loop: true,
  });
}
```

`setDepth(1)` draws the jar above falling items (default depth is `0`).

### Spawn helper

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

### Move + cleanup in `update()`

Loop **backwards** so you can `splice` safely:

```js
update(time, delta) {
  // ... player movement from lesson 3 ...

  for (let i = this.#fallingObjects.length - 1; i >= 0; i--) {
    const obj = this.#fallingObjects[i];
    obj.y += this.#fallingObjectsSpeed * (delta / 1000);

    if (obj.y > this.scale.height) {
      obj.destroy();              // remove from Phaser
      this.#fallingObjects.splice(i, 1); // remove from game state
    }
  }
}
```

The array **is** your game state: if it’s in the list, it exists for gameplay.

**Checkpoint:** Candies spawn every second at random X, fall, and vanish past the bottom without the array growing forever.

---

## 5. Collision detection (no physics)

### Goal

When a candy’s rectangle overlaps the jar’s rectangle, “catch” it (destroy + remove from array).

### Idea

Collision = two axis-aligned rectangles overlap. Phaser can answer that for you:

```js
const overlapPoints = Phaser.Geom.Intersects.GetRectangleToRectangle(
  this.#player.getBounds(),
  obj.getBounds(),
);

if (overlapPoints.length > 0) {
  // overlapping
}
```

`getBounds()` returns `{ x, y, width, height }` for the object’s bounding box.

### Inside the falling-object loop

After updating `obj.y`:

```js
for (let i = this.#fallingObjects.length - 1; i >= 0; i--) {
  const obj = this.#fallingObjects[i];
  obj.y += this.#fallingObjectsSpeed * (delta / 1000);

  const overlapPoints = Phaser.Geom.Intersects.GetRectangleToRectangle(
    this.#player.getBounds(),
    obj.getBounds(),
  );

  if (overlapPoints.length > 0) {
    obj.destroy();
    this.#fallingObjects.splice(i, 1);
    console.log('Caught item');
    continue; // don't also treat as a miss
  }

  if (obj.y > this.scale.height) {
    obj.destroy();
    this.#fallingObjects.splice(i, 1);
  }
}
```

Later videos replace the `console.log` with scoring. Physics engines do the same rectangle (or shape) checks under the hood — you’re learning that math first on purpose.

**Checkpoint:** Catching a candy removes it; missing still lets it fall off and clean up.

---

## 6. Score & UI text

### Goal

Keep score as **state**; show it with text objects that update when state changes.

### Rule

> The number is the truth. The text only mirrors it.

### Properties + init

```js
/** @type {number} */
#score;
/** @type {Phaser.GameObjects.Text} */
#scoreTextGameObject;
```

```js
init() {
  this.#playerSpeed = 500;
  this.#fallingObjectsSpeed = 200;
  this.#score = 0;
}
```

### Shared text style + HUD in `create()`

```js
const textConfig = {
  fontSize: '40px',
  color: '#043D8C',
  stroke: '#ffffff',
  strokeThickness: 6,
};

// at end of create():
const scoreTextPrefix = this.add.text(10, 10, 'Score:', textConfig);
this.#scoreTextGameObject = this.add.text(
  scoreTextPrefix.x + scoreTextPrefix.width,
  scoreTextPrefix.y,
  `${this.#score}`,
  textConfig,
);
```

Splitting “Score:” and the number makes later localization / reuse easier.

### On catch, update state then UI

```js
if (overlapPoints.length > 0) {
  obj.destroy();
  this.#fallingObjects.splice(i, 1);
  this.#score += 10;
  this.#scoreTextGameObject.setText(`${this.#score}`);
  continue;
}
```

**Checkpoint:** Each catch adds 10; the number on screen updates immediately.

---

## 7. Misses, game over & restart

### Goal

Three misses → stop play, show Game Over, click to restart the scene.

### New properties

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

```js
init() {
  this.#playerSpeed = 500;
  this.#fallingObjectsSpeed = 200;
  this.#score = 0;
  this.#misses = 0;
  this.#maxMisses = 3;
  this.#isGameOver = false;
}
```

### Store the timer (so you can stop it)

```js
this.#timerEvent = this.time.addEvent({
  delay: 1000,
  callback: this.#spawnFallingObject,
  callbackScope: this,
  loop: true,
});
```

### Lives UI (next to score)

```js
const livesTextPrefix = this.add.text(10, 50, 'Lives:', textConfig);
this.#livesTextGameObject = this.add.text(
  livesTextPrefix.x + livesTextPrefix.width,
  livesTextPrefix.y,
  `${this.#maxMisses - this.#misses}`,
  textConfig,
);
```

### Update flow

```js
update(time, delta) {
  if (this.#isGameOver) {
    return;
  }

  // ... player movement ...

  for (let i = this.#fallingObjects.length - 1; i >= 0; i--) {
    const obj = this.#fallingObjects[i];
    obj.y += this.#fallingObjectsSpeed * (delta / 1000);

    const overlapPoints = Phaser.Geom.Intersects.GetRectangleToRectangle(
      this.#player.getBounds(),
      obj.getBounds(),
    );

    if (overlapPoints.length > 0) {
      obj.destroy();
      this.#fallingObjects.splice(i, 1);
      this.#score += 10;
      this.#scoreTextGameObject.setText(`${this.#score}`);
      continue;
    }

    if (obj.y > this.scale.height) {
      obj.destroy();
      this.#fallingObjects.splice(i, 1);
      this.#misses += 1;
      this.#livesTextGameObject.setText(`${this.#maxMisses - this.#misses}`);
    }
  }

  if (this.#misses >= this.#maxMisses) {
    this.#handleGameOver();
  }
}
```

### Game over handler

Text objects default to a **top-left** origin — use `setOrigin(0.5)` to center the message.

```js
#handleGameOver() {
  this.#isGameOver = true;
  this.#timerEvent.remove(); // stop spawning

  this.add
    .text(this.scale.width / 2, this.scale.height / 2, 'Game Over', textConfig)
    .setOrigin(0.5);

  this.input.once(Phaser.Input.Events.POINTER_DOWN, () => {
    this.scene.restart(); // full clean reset via init/create
  });
}
```

**Checkpoint:** Three misses freeze movement and spawning, show Game Over; a click restarts score, lives, and objects.

---

## 8. Where to go next

You now have the core systems most games reuse:

| System | What you built |
|--------|----------------|
| Scenes | Preload → Game |
| Game loop | `update` every frame |
| Input | Cursor keys |
| Dynamic objects | Timer + array |
| Collision | Rectangle overlap |
| State + UI | Score / lives |
| Rules | Miss limit + restart |

Ideas to extend:

- Increase fall speed or spawn rate over time
- Different point values (or “bad” items) per frame
- Sound on catch / miss
- Title scene before `GameScene`
- Animations or particles on collect

Next Phaser topics worth learning: arcade/matter physics, tilemaps, sprite animations, cameras, particles.

---

## 9. Updating Phaser

This starter loads Phaser from local CDN copies under `assets/js/`, not npm. To upgrade:

1. Replace `assets/js/phaser.js` and `assets/js/phaser.min.js` with the versions from the [Phaser CDN / GitHub release](https://github.com/phaserjs/phaser).
2. Replace `src/types/phaser.d.ts` if you want updated IntelliSense.
3. Refresh and check the console banner for the new version.

If you use npm elsewhere, Phaser 4 often needs:

```js
import * as Phaser from 'phaser';
```

instead of a default import. This course project doesn’t need that change.

The repo also ships `scripts/` helpers to pull a specific version — see the project README.

---

## 10. Complete `GameScene` reference

Finished scene (matches the course “game over” milestone):

```js
import Phaser from '../lib/phaser.js';
import { SCENE_KEYS } from '../common/scene-keys.js';
import { ASSET_KEYS } from '../common/assets.js';

const textConfig = {
  fontSize: '40px',
  color: '#043D8C',
  stroke: '#ffffff',
  strokeThickness: 6,
};

export class GameScene extends Phaser.Scene {
  /** @type {Phaser.Types.Input.Keyboard.CursorKeys} */
  #cursorKeys;
  /** @type {Phaser.GameObjects.Image} */
  #player;
  /** @type {number} */
  #playerSpeed;
  /** @type {Phaser.GameObjects.Image[]} */
  #fallingObjects;
  /** @type {string[]} */
  #fallingObjectFrames;
  /** @type {number} */
  #fallingObjectsSpeed;
  /** @type {number} */
  #score;
  /** @type {Phaser.GameObjects.Text} */
  #scoreTextGameObject;
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

  constructor() {
    super({ key: SCENE_KEYS.GAME_SCENE });
  }

  init() {
    this.#playerSpeed = 500;
    this.#fallingObjectsSpeed = 200;
    this.#score = 0;
    this.#misses = 0;
    this.#maxMisses = 3;
    this.#isGameOver = false;
  }

  create() {
    if (!this.input) {
      console.warn('Input plugin is not available');
      return;
    }

    const { width, height } = this.scale;

    this.add.image(width / 2, height / 2, ASSET_KEYS.BACKGROUND);
    this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR).setDepth(1);
    this.#cursorKeys = this.input.keyboard.createCursorKeys();

    this.#fallingObjects = [];
    this.#fallingObjectFrames = Object.keys(
      this.textures.get(ASSET_KEYS.OBJECTS).frames,
    ).filter((name) => name !== '__BASE');

    this.#timerEvent = this.time.addEvent({
      delay: 1000,
      callback: this.#spawnFallingObject,
      callbackScope: this,
      loop: true,
    });

    const scoreTextPrefix = this.add.text(10, 10, 'Score:', textConfig);
    this.#scoreTextGameObject = this.add.text(
      scoreTextPrefix.x + scoreTextPrefix.width,
      scoreTextPrefix.y,
      `${this.#score}`,
      textConfig,
    );

    const livesTextPrefix = this.add.text(10, 50, 'Lives:', textConfig);
    this.#livesTextGameObject = this.add.text(
      livesTextPrefix.x + livesTextPrefix.width,
      livesTextPrefix.y,
      `${this.#maxMisses - this.#misses}`,
      textConfig,
    );
  }

  update(time, delta) {
    if (this.#isGameOver) {
      return;
    }

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

    for (let i = this.#fallingObjects.length - 1; i >= 0; i--) {
      const obj = this.#fallingObjects[i];
      obj.y += this.#fallingObjectsSpeed * (delta / 1000);

      const overlapPoints = Phaser.Geom.Intersects.GetRectangleToRectangle(
        this.#player.getBounds(),
        obj.getBounds(),
      );

      if (overlapPoints.length > 0) {
        obj.destroy();
        this.#fallingObjects.splice(i, 1);
        this.#score += 10;
        this.#scoreTextGameObject.setText(`${this.#score}`);
        continue;
      }

      if (obj.y > this.scale.height) {
        obj.destroy();
        this.#fallingObjects.splice(i, 1);
        this.#misses += 1;
        this.#livesTextGameObject.setText(`${this.#maxMisses - this.#misses}`);
      }
    }

    if (this.#misses >= this.#maxMisses) {
      this.#handleGameOver();
    }
  }

  #spawnFallingObject() {
    const randomFrame = Phaser.Utils.Array.GetRandom(this.#fallingObjectFrames);
    const obj = this.add
      .image(
        Phaser.Math.RND.between(50, this.scale.width - 50),
        0,
        ASSET_KEYS.OBJECTS,
        randomFrame,
      )
      .setScale(0.75);
    this.#fallingObjects.push(obj);
  }

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
}
```

Put `textConfig` at module scope (as above) so both HUD and Game Over share the same style.

---

## Controls & playtest checklist

| Action | Expected |
|--------|----------|
| ← / → | Jar moves smoothly, stays on screen |
| Candy spawns | About once per second, random X / frame |
| Catch | Candy disappears, score +10 |
| Miss | Lives decrease |
| 3 misses | “Game Over”, no more movement/spawns |
| Click | Scene restarts, score/lives reset |

Enjoy building — change numbers, break things, and add weird ideas. That’s how the systems stick.

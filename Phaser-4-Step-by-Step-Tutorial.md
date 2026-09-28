# Build Your First Game with Phaser 4 — Step-by-Step Tutorial

Written companion to the [YouTube mini course](https://www.youtube.com/playlist?list=PLmcXe0-sfoSheQinG8d5JQBiTqiKnBgo7).

Code matches official tags on [phaser-4-falling-objects-game](https://github.com/devshareacademy/phaser-4-falling-objects-game). Snippets are **only new/changed lines** in `src/scenes/game-scene.js` unless noted.

| # | Lesson | Tag |
|---|--------|-----|
| 1 | Setup & how Phaser works | `0-initial-project` |
| 2 | Coordinates & sprites | `1-images` |
| 3 | Keyboard movement | `2-player-movement` |
| 4 | Spawning objects | `3-falling-objects` |
| 5 | Collisions (no physics) | `4-collisions` |
| 6 | Score & UI | `5-score` |
| 7 | Game over & restart | `6-game-over` |
| 8 | What you learned / next | — |
| 9 | Updating Phaser | `phaser-ver-4.1.0` |

---

## Prerequisites

1. Unzip / clone the starter at tag `0-initial-project`.
2. Open the project root (folder that contains `index.html`).
3. Start a local server:

| Tool | Command / action |
|------|------------------|
| VS Code | Install [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) → **Go Live** |
| Python | `python3 -m http.server 8080` → open `http://localhost:8080/` |
| Node | `npx http-server` |

4. Confirm the canvas loads and the console shows the Phaser version banner.

### Files you’ll touch

| Path | Role |
|------|------|
| `src/main.js` | Boots `Phaser.Game`, registers scenes |
| `src/scenes/preload-scene.js` | Loads assets, then starts gameplay |
| `src/scenes/game-scene.js` | Almost all gameplay edits |
| `src/common/assets.js` | Texture keys (`BACKGROUND`, `JAR`, `OBJECTS`) |

### Scene lifecycle (mental model)

| Method | Runs | Job |
|--------|------|-----|
| `init()` | Once per scene start/restart | Reset numbers & flags |
| `preload()` | Once before create | Load images / atlases |
| `create()` | Once after load | Build world, input, timers, UI |
| `update(time, delta)` | Every frame (~60×/sec) | Move, poll input, collide, check rules |

**Why split Preload + Game scenes?** Loading stays separate from play logic. Gameplay code doesn’t wait on loaders mixed into the same methods.

---

## 1. Setup & how Phaser works

**Tag:** `0-initial-project`  
**Goal:** Run the starter and see the game loop. No permanent code changes.

### Why this lesson

Phaser doesn’t replace JavaScript — it gives you the pieces every browser game needs: a canvas, asset loading, input, and a game loop. Almost everything you write lives inside a **scene**, and beginner bugs usually come from putting code in the wrong lifecycle method. This lesson is just about seeing that loop run so later movement and collisions have a place to live.

### Steps

1. **Read `main.js` — this is what boots the game**

```js
const game = new Phaser.Game(gameConfig);

game.scene.add(SCENE_KEYS.PRELOAD_SCENE, PreloadScene);
game.scene.add(SCENE_KEYS.GAME_SCENE, GameScene);
game.scene.start(SCENE_KEYS.PRELOAD_SCENE);
```

**Why:** `new Phaser.Game` creates the canvas. Scenes are registered, then Preload starts.

2. **Read `PreloadScene` — load, then hand off**

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

**Why:** Phaser won’t start gameplay until listed assets are cached. Keys here must match keys used later in `add.image`.

3. **Open starting `GameScene.create()`**

```js
create() {
  this.add.image(this.scale.width / 2, this.scale.height / 2, ASSET_KEYS.BACKGROUND);
}
```

**Why:** One centered background proves scenes + textures work before gameplay.

4. **Prove the loop — temporarily add logs in `GameScene`**

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

5. **Refresh → console:** `preload` and `create` once; `update` spam.  
6. **Remove the temporary logs.**

**Checkpoint:** Server works, Phaser banner visible, you can explain preload → create → update.

---

## 2. Coordinates, assets & first sprites

**Tag:** `1-images`  
**Goal:** Place background, jar (player), and one candy.

### Why this lesson

Before anything can move, you need to know where “here” is. In Phaser (and most 2D engines), `(0, 0)` is the **top-left** of the screen: X grows to the right, Y grows downward. The other gotcha is **origin**: by default Phaser positions an image by its *center*, so placing something at `(0, 0)` parks the middle of the sprite on the corner and you only see a quarter of it. Once that clicks, centering with `width / 2` and `height / 2` feels obvious.

### Steps

1. **In `GameScene.create()`, replace the body with:**

```js
const { width, height } = this.scale;

this.add.image(width / 2, height / 2, ASSET_KEYS.BACKGROUND);
this.add.image(width / 2, height, ASSET_KEYS.JAR);
this.add.image(width / 2, height / 2, ASSET_KEYS.OBJECTS, 'button1.png').setScale(0.75);
```

2. **Why each line**

| Line | Why |
|------|-----|
| `const { width, height }` | Avoid repeating `this.scale.*`; center = half size |
| Background at `width/2, height/2` | Origin is center → this fills the view |
| Jar at `width/2, height` | Bottom-center; half jar sits on the edge (OK for now) |
| `OBJECTS` + `'button1.png'` | Atlas frame name from `spritesheet.json` |
| `.setScale(0.75)` | Shrink; `1` = native size |

3. **Refresh** — background, jar, candy.

**Checkpoint:** All three visible; you know why `(0,0)` looked “wrong” before.

---

## 3. Player movement with the keyboard

**Tag:** `2-player-movement`  
**Goal:** Move the jar with ←/→; stay on screen; same speed on every machine.

### Why this lesson

Movement isn’t magic — it’s changing `x` (or `y`) a little bit every frame. The keyboard just tells you when to nudge those numbers. You check keys inside `update` because that method runs continuously; setting something up once in `create` isn’t enough while a key is held. We also scale by `delta` so a 120fps machine doesn’t move twice as fast as a 60fps one, and we clamp using half the jar’s width because position is the sprite’s center.

### Steps

1. **Add class fields** (top of `GameScene`):

```js
/** @type {Phaser.Types.Input.Keyboard.CursorKeys} handles player input */
#cursorKeys;
/** @type {Phaser.GameObjects.Image} the player (jar) in our game */
#player;
/** @type {number} how fast our player can move in our game */
#playerSpeed;
```

**Why private fields:** Keep a stable handle to the jar and input across `create` / `update`.

2. **Add `init()`**

```js
init() {
  this.#playerSpeed = 500;
}
```

**Why `init`:** Runs on every scene start/restart — perfect for resetting values. Not for creating sprites.

3. **At start of `create()`, guard input**

```js
if (!this.input) {
  console.warn('Input plugin is not available');
  return;
}
```

4. **Change jar creation to store a reference**

```js
this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR);
```

5. **After the candy line, wire cursors**

```js
this.#cursorKeys = this.input.keyboard.createCursorKeys();
```

**Why:** Sets up arrow-key state once; you’ll read `.left.isDown` / `.right.isDown` each frame. Keep the static candy for now (lesson 4 removes it).

6. **Add `update(time, delta)`**

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

7. **Refresh** — hold ←/→; jar should not leave the screen.

**Checkpoint:** Smooth movement; full jar stays visible.

---

## 4. Spawning falling objects

**Tag:** `3-falling-objects`  
**Goal:** Spawn on a timer, track in an array, fall, destroy when off-screen.

### Why this lesson

A single candy isn’t a game — you need many objects over time. The usual pattern is an **array** as game state: if it’s in the list, it still exists for gameplay. A Phaser timer calls your spawn function every second so you don’t hand-roll clocks in `update`. When something leaves play, call both `destroy()` (Phaser) and `splice` (your array), and loop the list backwards so removing items doesn’t skip the next one. Raising the jar’s `depth` keeps it drawn above newly spawned candies.

### Steps

1. **Add fields**

```js
/** @type {Phaser.GameObjects.Image[]} the falling objects for the player to collect */
#fallingObjects;
/** @type {string[]} the list of frames from the falling object spritesheet that we loaded in */
#fallingObjectFrames;
/** @type {number} how fast the objects will fall */
#fallingObjectsSpeed;
```

2. **In `init()`, add**

```js
this.#fallingObjectsSpeed = 200;
```

3. **In `create()`, change the player line**

```js
this.#player = this.add.image(width / 2, height, ASSET_KEYS.JAR).setDepth(1);
```

4. **Delete the static candy line**

```js
this.add.image(width / 2, height / 2, ASSET_KEYS.OBJECTS, 'button1.png').setScale(0.75);
```

5. **After cursor keys, add spawn setup**

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

| Bit | Why |
|-----|-----|
| Filter `__BASE` | Phaser adds an internal frame; you don’t want to draw it |
| `delay: 1000` | One second between spawns |
| `callbackScope: this` | So `#spawnFallingObject` keeps class `this` |
| `loop: true` | Keep spawning |

6. **After player clamp in `update()`, add**

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

7. **Add spawn method**

```js
#spawnFallingObject() {
  const randomFrame = Phaser.Utils.Array.GetRandom(this.#fallingObjectFrames);
  const obj = this.add
    .image(Phaser.Math.RND.between(50, this.scale.width - 50), 0, ASSET_KEYS.OBJECTS, randomFrame)
    .setScale(0.75);
  this.#fallingObjects.push(obj);
}
```

**Why padding `50 … width-50`:** Keeps spawns fully on-screen horizontally.

8. **Refresh** — candies appear, fall, vanish past the bottom; array shouldn’t grow forever.

**Checkpoint:** Continuous spawn + fall + cleanup.

---

## 5. Collision detection (no physics)

**Tag:** `4-collisions`  
**Goal:** Catch when jar and candy rectangles overlap.

### Why this lesson

Up to now the jar and candies ignore each other. Collision detection sounds heavy, but the beginner version is a simple question: do these two rectangles overlap? Each game object can hand you a bounding box via `getBounds()`, and Phaser can compare those boxes for you. We’re skipping the physics engine on purpose — once you’ve done this with math, “arcade physics collided” won’t feel like a black box.

### Steps

1. **In the falling-object loop, after `obj.y += …`, before the off-screen `if`, insert:**

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

2. **Refresh** — catch a candy; console logs; candy disappears.

**Checkpoint:** Overlap removes the candy; missed ones still fall off and clean up.

---

## 6. Score & UI text

**Tag:** `5-score`  
**Goal:** Track points in state; show them with text.

### Why this lesson

Catching works, but there’s no feedback yet. Keep a numeric `#score` as the **source of truth**, and treat on-screen text as a mirror of that number — never the other way around. When something is collected, bump the score first, then call `setText`. Splitting the `"Score:"` label from the value makes it easier to restyle or translate later without rewriting the number logic.

### Steps

1. **Add fields**

```js
#score;
#scoreTextGameObject;
```

2. **In `init()`, add**

```js
this.#score = 0;
```

3. **At end of `create()`, add HUD**

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

**Why place value at `prefix.x + prefix.width`:** Sits immediately after the label.

4. **Replace `console.log('caught item')` with**

```js
this.#score += 10;
this.#scoreTextGameObject.setText(`${this.#score}`);
```

5. **Refresh** — catches increment score on screen.

**Checkpoint:** State and UI stay in sync on every catch.

---

## 7. Misses, game over & restart

**Tag:** `6-game-over`  
**Goal:** 3 misses → stop play, show Game Over, click to restart.

### Why this lesson

Without a fail state, the loop never ends — and that doesn’t feel like a game. We’ll treat a candy falling past the bottom as a miss, end after three misses, and freeze gameplay with an `isGameOver` flag. Important detail: also stop the spawn timer, or new objects keep appearing after “Game Over.” Restarting the scene is the easy reset button — Phaser re-runs `init` and `create`, so score, lives, and arrays come back clean. Center the Game Over text with `setOrigin(0.5)` because text defaults to a top-left origin.

### Steps

1. **Move `textConfig` above the class** (cut from inside `create()` so Game Over can reuse it):

```js
const textConfig = {
  fontSize: '40px',
  color: '#043D8C',
  stroke: '#ffffff',
  strokeThickness: 6,
};
```

2. **Add fields**

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

3. **In `init()`, add**

```js
this.#misses = 0;
this.#maxMisses = 3;
this.#isGameOver = false;
```

4. **Store the timer** — change `this.time.addEvent({` to:

```js
this.#timerEvent = this.time.addEvent({
```

5. **After score HUD in `create()`, add lives**

```js
const livesTextPrefix = this.add.text(10, 50, 'Lives:', textConfig);
this.#livesTextGameObject = this.add.text(
  livesTextPrefix.x + livesTextPrefix.width,
  livesTextPrefix.y,
  `${this.#maxMisses - this.#misses}`,
  textConfig,
);
```

**Why `maxMisses - misses`:** Show lives left, not raw miss count.

6. **At the very start of `update()`**

```js
if (this.#isGameOver) {
  return;
}
```

7. **After score update on catch, add `continue`**

```js
continue;
```

**Why:** Don’t also treat a caught object as an off-screen miss in the same frame.

8. **In the off-screen branch, after destroy/splice, add**

```js
this.#misses += 1;
this.#livesTextGameObject.setText(`${this.#maxMisses - this.#misses}`);
```

9. **After the `for` loop, add**

```js
if (this.#misses >= this.#maxMisses) {
  this.#handleGameOver();
}
```

10. **Add `#handleGameOver()`**

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

| Line | Why |
|------|-----|
| `isGameOver = true` | Freezes `update` via early return |
| `timerEvent.remove()` | Stops new spawns |
| `setOrigin(0.5)` | Centers the label |
| `input.once(…)` | One click → restart; listener doesn’t stack |

11. **Refresh** — miss 3 times → Game Over → click → fresh run.

**Checkpoint:** Full loop: move, catch, score, miss, lose, restart.

---

## 8. What you learned / next

You just built a complete playable loop in Phaser 4 — not a huge game, but the same systems most games reuse. The physics engine was skipped on purpose so collisions and state feel understandable first.

### Systems you built

| System | Where |
|--------|--------|
| Scenes | Preload → Game |
| Game loop | `update` |
| Input | Cursor keys |
| Dynamic objects | Timer + array |
| Collision | Rectangle overlap |
| State + UI | Score / lives |
| Rules | Miss limit + restart |

### Try next (pick one)

1. Raise fall speed or spawn rate over time  
2. Different points (or “bad”) items per frame  
3. SFX on catch / miss  
4. Title scene before `GameScene`  
5. Particles on collect  

### Learn next in Phaser

Physics · tilemaps · sprite animation · cameras · particles

---

## 9. Updating Phaser

**Tag:** `phaser-ver-4.1.0`

### Why it matters here

When Phaser 4 ships updates, people often break imports by treating this starter like an npm app. Here Phaser is loaded from **local CDN copies** under `assets/js/`, with a small `phaser.d.ts` for editor IntelliSense — so upgrading is a file swap (or the repo’s update script), not `npm update`.

### Steps

1. Replace `assets/js/phaser.js`  
2. Replace `assets/js/phaser.min.js`  
3. Replace `src/types/phaser.d.ts` (IntelliSense)  
4. Refresh → check console banner for the new version  

Optional: use the repo’s `scripts/` updater (see project README).

**npm projects elsewhere:** use `import * as Phaser from 'phaser'` (wildcard). This course project does not need that change.

---

## Playtest checklist

| Action | Expected |
|--------|----------|
| ← / → | Jar moves; stays on screen |
| Spawn | ~1/sec; random X / frame |
| Catch | Candy gone; score +10 |
| Miss | Lives −1 |
| 3 misses | Game Over; no move/spawn |
| Click | Restart; state reset |

Diff-check anytime:

```bash
git show 6-game-over:src/scenes/game-scene.js
```

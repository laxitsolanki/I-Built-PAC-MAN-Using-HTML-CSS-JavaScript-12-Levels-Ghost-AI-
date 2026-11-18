# 🎮 PAC-MAN PROJECT — MASTER DOCUMENTATION

> **Ye file project ka complete knowledge base hai.**
> Har ek file, folder, function, aur code flow yahan documented hai.
> Koi bhi AI ya developer is file ko padh ke poora project samajh sakta hai.
> Jab bhi project me koi update ho — us section ko yahan bhi update karo.

---

## 📁 PROJECT STRUCTURE (Folder & File Tree)

```
pacman-master/
│
├── index.html                  → Main webpage (Game UI + Hindi layout)
├── favicon.png                 → Browser tab icon
├── README.md                   → Short project description
├── MASTER.md                   → ⭐ YE FILE — Complete documentation
│
└── static/
    ├── font/
    │   └── PressStart2P.ttf    → Retro pixel game font (Google Fonts)
    │
    ├── script/
    │   ├── game.js             → 🔧 Core Game Engine (reusable framework)
    │   └── index.js            → 🎮 Game Logic (Pac-Man specific code)
    │
    └── style/
        └── index.css           → 🎨 Indian Theme CSS (Tricolor design)
```

---

## 🌊 COMPLETE CODE FLOW (Kya se Kya hota hai)

```
Browser/WebView opens index.html
        │
        ▼
CSS load hoti hai (index.css) → Indian theme apply hoti hai
        │
        ▼
game.js load hota hai → Game Engine (Game class) ready hoti hai
        │
        ▼
index.js load hota hai → PressStart2P font load hota hai
        │
        ▼
Font load complete → game.init() call hota hai
        │
        ▼
┌─────────────────────────────────────┐
│         GAME STAGES (Screens)       │
│                                     │
│  Stage 0 → Start Screen            │
│  Stage 1-12 → Level 1 to Level 12  │
│  Stage 13 → End Screen (Win/Lose)   │
└─────────────────────────────────────┘
        │
        ▼
requestAnimationFrame loop shuru hota hai (~60fps)
        │
        ▼
Har frame me:
  1. Canvas clear hota hai
  2. Stage.update() → collision, scoring check
  3. Maps draw hote hain (walls + beans)
  4. Items draw hote hain (player + ghosts)
  5. Next frame request hota hai
```

---

## 📄 FILE 1: `index.html`

### Kya karta hai?
- Game ka main webpage hai
- Canvas element render karta hai jisme game chalti hai
- Hindi UI dikhata hai (controls info, how to play, features)
- WebView aur Browser dono me kaam karta hai

### Important Elements

| Element | ID/Class | Kaam |
|---|---|---|
| `<canvas>` | `id="canvas"` | Game yahan draw hoti hai (960x640) |
| `.app-header` | — | Indian theme header with logo |
| `.canvas-wrap` | — | Canvas ka container |
| `.controls-info` | — | Keyboard/touch controls info |
| `.mod-intro` | — | Game features grid |
| `.mod-howto` | — | Hindi me kaise khelen |
| `.app-footer` | — | Tricolor border footer |

### Script Loading Order (ZAROORI — order mat badlo)
```html
<script src="./static/script/game.js"></script>   <!-- 1st: Engine -->
<script src="./static/script/index.js"></script>  <!-- 2nd: Game Logic -->
```

### WebView Compatibility
- Koi external URL nahi hai (fully offline capable)
- Koi redirect script nahi hai
- `viewport` meta tag set hai mobile ke liye
- `user-scalable=0` — pinch zoom band hai (game ke liye zaroori)

---

## 📄 FILE 2: `static/script/game.js`

### Kya karta hai?
- Ye ek **mini game engine** hai
- Pac-Man specific koi code nahi hai isme
- Reusable framework hai — kisi bhi Canvas game me use ho sakta hai

### Main Class: `Game(id, params)`

```javascript
var game = new Game('canvas');
// 'canvas' = HTML element ka ID
// params = { width: 960, height: 640 }
```

### Game Engine ke 4 Core Objects

#### 1. `Item` — Moving Objects (Player, Ghosts, UI text)
```
Properties:
  x, y          → Canvas position (pixels)
  width, height → Size
  type          → 0=normal, 1=player, 2=NPC/ghost
  status        → 0=inactive, 1=normal, 2=paused, 3=temp, 4=dead
  orientation   → 0=right, 1=down, 2=left, 3=up
  speed         → Movement speed (pixels per frame)
  location      → Map object (agar map se bind ho)
  coord         → Map coordinate {x, y}
  frames        → Kitne frames baad update ho
  times         → Frame counter
  timeout       → Countdown timer
  update()      → Har frame me call hota hai (logic)
  draw()        → Har frame me call hota hai (drawing)
```

#### 2. `Map` — Grid-based Maps (Walls, Beans)
```
Properties:
  x, y          → Canvas me map ka starting point
  size          → Ek cell ka size (20px)
  data[][]      → 2D array (map data)
  x_length      → Columns count
  y_length      → Rows count
  cache         → true = static map (performance optimization)

Methods:
  get(x, y)              → Map cell ki value lo
  set(x, y, value)       → Map cell ki value set karo
  coord2position(cx, cy) → Map coord → Canvas pixel position
  position2coord(x, y)   → Canvas pixel → Map coord
  finder(params)         → 🧠 Pathfinding Algorithm (BFS)
```

#### 3. `Stage` — Game Screens/Levels
```
Properties:
  index    → Stage number
  status   → 0=inactive, 1=running, 2=paused, 3=temp
  maps[]   → Is stage ke maps
  items[]  → Is stage ke items
  timeout  → Stage level countdown

Methods:
  createItem(options)    → Naya item banao
  createMap(options)     → Naya map banao
  resetItems()           → Sab items reset karo
  resetMaps()            → Sab maps reset karo
  reset()                → Poora stage reset karo
  bind(event, callback)  → Keyboard/touch event bind karo
  getItemsByType(type)   → Type se items filter karo
```

#### 4. `Game` — Main Controller
```
Methods:
  createStage(options)  → Naya stage banao
  setStage(index)       → Specific stage par jao
  nextStage()           → Agla stage
  getStages()           → Saare stages ki list
  start()               → Animation loop shuru karo
  stop()                → Animation loop band karo
  init()                → Game initialize karo (stage 0 se)
  getPosition(e)        → Mouse/touch event ka canvas position
```

### 🧠 Pathfinding Algorithm (`Map.finder`)
```
Algorithm: BFS (Breadth-First Search)
Kaam: Ghost ko player tak ka shortest path dhundho

Parameters:
  map   → 2D array (current map state)
  start → {x, y} starting coordinate
  end   → {x, y} target coordinate
  type  → 'path' = full path return karo
           'next' = sirf next possible steps return karo

Special Feature:
  - Map edges wrap hote hain (left se right, top se bottom)
  - value = -1 matlab edge wrap point hai
  - value = 0 matlab walkable cell
  - value = 1 matlab wall (blocked)

Return: Array of coordinates [{x,y}, {x,y}, ...]
```

### Animation Loop (game.start)
```javascript
// ~60fps loop (16ms minimum gap — high refresh rate screens ke liye limit)
requestAnimationFrame(fn)
  → Canvas clear karo
  → Stage timeout countdown
  → Stage.update() call karo
  → Maps draw karo (cache check ke saath)
  → Items draw karo (status check ke saath)
  → Next frame request karo
```

---

## 📄 FILE 3: `static/script/index.js`

### Kya karta hai?
- Pac-Man game ki **saari business logic** yahan hai
- Game engine (game.js) use karke actual game banata hai
- 12 levels, player, 4 ghosts, scoring — sab yahan define hai

### Global Variables
```javascript
_COIGIG[]    → 12 levels ka config array (map data + wall color + power pellet positions)
_COLOR[]     → Ghost colors: ['#F00', '#F93', '#0CF', '#F9C'] (Red, Orange, Cyan, Pink)
_COS[]       → Direction X movement: [1, 0, -1, 0]  (right, down, left, up)
_SIN[]       → Direction Y movement: [0, 1, 0, -1]  (right, down, left, up)
_LIFE        → Player lives: 5 (shuru me)
_SCORE       → Player score: 0 (shuru me)
```

### Map Data Values (2D Array me)
```
0 → Walkable path (yahan dots/beans hain)
1 → Wall (block — nahi ja sakte)
2 → Ghost house (ghosts ka ghar — white border draw hoti hai)
-1 → Edge wrap point (map ke bahar = dusri taraf se aao)
```

### Level Config Structure
```javascript
{
  map: [[...], [...], ...],  // 28x31 grid (28 columns, 31 rows)
  wall_color: '#09f',        // Is level ki wall ka color
  goods: {                   // Power pellets ki positions
    '1,3': 1,                // 'x,y': 1 format
    '26,3': 1,
    '1,23': 1,
    '26,23': 1
  }
}
```

### 3 Main Stages (Screens)

#### Stage 0 — Start Screen
```
Items:
  1. Pac-Man logo animation (mouth open/close)
  2. "Pac-Man" title text
  3. "Press Enter to start" blinking text
  4. Copyright text (clickable)

Event: Enter/Space → game.nextStage() → Level 1 shuru
```

#### Stage 1-12 — Game Levels
```
Maps:
  1. map (wall map)   → Static, cached, wall drawing
  2. beans (dot map)  → Dynamic, dots/pellets drawing

Items:
  1. Score + Level display (top right)
  2. PAUSE text (blinks when paused)
  3. Lives display (bottom right)
  4. Ghost 1 (Red)    → type:2, NPC
  5. Ghost 2 (Orange) → type:2, NPC
  6. Ghost 3 (Cyan)   → type:2, NPC
  7. Ghost 4 (Pink)   → type:2, NPC
  8. Player (Pac-Man) → type:1, player controlled

Stage Update Logic (har frame):
  IF status==1 (normal):
    → Har ghost aur player ke beech distance check karo
    → Distance < 750 (pixels squared) = collision!
      → Ghost status==3 (scared)? → Ghost ko khao (+10 score)
      → Ghost status==1 (normal)? → Player ko khao (stage.status=3)
    → Koi bean nahi bachi? → nextStage() (next level)
  
  IF status==3 (player died):
    → timeout countdown
    → timeout==0: _LIFE--
      → Lives bachi hain? → resetItems() (respawn)
      → Lives khatam? → Last stage (game over screen)
```

#### Stage 13 — End Screen
```
Items:
  1. "YOU WIN!" ya "GAME OVER" text (_LIFE > 0 check)
  2. Final score: _SCORE + (50 * remaining lives)

Event: Enter/Space → _SCORE=0, _LIFE=5, game.setStage(1) → Restart
```

### Player (Pac-Man) Logic
```
Starting position: coord {x:13.5, y:23}
Speed: 2 pixels/frame
Orientation: 2 (left)

update() har frame:
  IF coord.offset == 0 (cell center par hai):
    → control.orientation check karo (player ne key press ki?)
    → Agar us direction me wall nahi → orientation change karo
    → control = {} (clear)
    → Agle cell ki value check karo:
      → value==0: move karo
      → value<0: wrap karo (dusri taraf teleport)
      → value==1: ruko (wall hai)
  ELSE (cell ke beech me hai):
    → Bean check karo current cell par:
      → Bean hai? → _SCORE++, bean hata do
      → Power pellet hai? → Saare ghosts ko scared karo (timeout=450)
    → Move karo current direction me

draw():
  → Yellow circle with mouth animation
  → Mouth angle orientation ke hisaab se rotate hoti hai
  → Death animation: mouth slowly closes (stage.timeout se)
```

### Ghost (NPC) Logic
```
Starting positions: coord {x:12+i, y:14} (i = 0,1,2,3)
Speed: 1 pixel/frame
Frames: 10 (har 10 frames me pathfinding update)

Ghost Status:
  1 = Normal (player ko dhundho — chase mode)
  3 = Scared (player se bhaago — flee mode, timeout=450 frames)
  4 = Dead (ghar wapas jao — return mode)

update() logic:
  IF status==1 (chase):
    → BFS pathfinding: ghost → player
    → Doosre ghosts ko walls maano (collision avoid)
    → path[0] = next step
  
  IF status==3 (scared):
    → BFS pathfinding: player → ghost (reverse!)
    → type='next' (sirf adjacent cells check)
    → Random next step choose karo (flee randomly)
    → timeout==0 → status=1 (normal wapas)
  
  IF status==4 (dead):
    → BFS pathfinding: ghost → spawn point
    → Spawn point par pahuncha? → status=1

Direction calculation:
  vector.x > coord.x → orientation=0 (right)
  vector.x < coord.x → orientation=2 (left)
  vector.y > coord.y → orientation=1 (down)
  vector.y < coord.y → orientation=3 (up)

draw():
  Normal: Colored ghost shape (2 animation frames)
  Scared: Gray color, small eyes
  Scared + timeout<80: Blinking (times%2)
  Dead: Invisible (status==4 me body nahi draw hoti)
```

### Keyboard Controls
```javascript
// Start Screen & End Screen:
Enter (13) / Space (32) → Next stage / Restart

// Game Levels:
Space (32)  → Pause / Resume
ArrowRight (39) → orientation = 0 (right)
ArrowDown  (40) → orientation = 1 (down)
ArrowLeft  (37) → orientation = 2 (left)
ArrowUp    (38) → orientation = 3 (up)
```

### Font Loading
```javascript
// Game tab tak shuru nahi hoti jab tak font load na ho
const myFont = new FontFace('PressStart2P', 'url(./static/font/PressStart2P.ttf)');
myFont.load().then(font => {
    document.fonts.add(font);
    game.init();  // Font ready → Game start
});
```

---

## 📄 FILE 4: `static/style/index.css`

### Kya karta hai?
- Indian Tricolor theme apply karta hai
- Game ko mobile/WebView me responsive banata hai
- Dark background with saffron/green accents

### CSS Variables (Colors)
```css
--saffron: #FF9933    → Indian flag saffron (headers, borders, accents)
--white:   #FFFFFF    → Pure white
--green:   #138808    → Indian flag green (how-to section)
--navy:    #000080    → Navy blue
--dark-bg: #0a0a1a    → Page background (near black)
--card-bg: #12122a    → Card/section background
--text-light: #e0e0e0 → Main text color
--text-muted: #999    → Secondary text color
```

### Layout Sections

| Class | Description |
|---|---|
| `.app-header` | Saffron gradient header, saffron bottom border |
| `.logo-area` | Game icon + title |
| `.header-badge` | "🇮🇳 Made for India" saffron-to-green gradient badge |
| `.mod-panel` | Game canvas container, saffron border, glow effect |
| `.canvas-wrap` | Black background canvas holder |
| `.controls-info` | Keyboard hints bar |
| `.key` | Styled keyboard key badges |
| `.mod-intro` | Feature grid section |
| `.feature-grid` | 2x2 grid of game features |
| `.mod-howto` | Green-bordered how-to-play section |
| `.app-footer` | Tricolor gradient border footer |

### Responsive Breakpoint
```css
@media screen and (max-width: 600px) {
  /* Mobile/WebView adjustments:
     - Smaller header text
     - Canvas fills full width
     - Controls stack vertically
     - Smaller font sizes
  */
}
```

### Canvas Responsive Behavior
```css
.canvas-wrap canvas {
    max-width: 100%;   /* Mobile me full width */
    height: auto;      /* Aspect ratio maintain */
}
/* Game internally runs at 960x640 but CSS scales it down visually */
```

---

## 📄 FILE 5: `static/font/PressStart2P.ttf`

- Retro pixel-style font hai
- Game ke andar Canvas text rendering ke liye use hoti hai
- `index.js` me FontFace API se load hoti hai
- Jab tak ye load na ho, game.init() call nahi hota

---

## 🎮 SCORING SYSTEM

| Action | Points |
|---|---|
| Ek dot (bean) khana | +1 point |
| Scared ghost khana | +10 points |
| Game complete (win) bonus | +50 × remaining lives |

```javascript
// Final score calculation (End screen):
_SCORE + 50 * Math.max(_LIFE - 1, 0)
// _LIFE-1 kyunki current life count nahi hoti
```

---

## 👻 GHOST BEHAVIOR SUMMARY

| Ghost | Color | Index | Behavior |
|---|---|---|---|
| Ghost 1 | Red `#F00` | 0 | Seedha player ko chase karta hai |
| Ghost 2 | Orange `#F93` | 1 | Player ko chase karta hai |
| Ghost 3 | Cyan `#0CF` | 2 | Player ko chase karta hai |
| Ghost 4 | Pink `#F9C` | 3 | Player ko chase karta hai |

> Sab ghosts BFS use karte hain lekin doosre ghosts ko walls maante hain
> Isliye wo alag-alag raston se aate hain aur player ko gherte hain

---

## 📱 ANDROID WEBVIEW INTEGRATION

### Option A — Online URL Load
```java
// MainActivity.java
WebView webView = findViewById(R.id.webView);
WebSettings settings = webView.getSettings();
settings.setJavaScriptEnabled(true);
settings.setDomStorageEnabled(true);
webView.loadUrl("https://aapki-website.com/pacman/");
```

### Option B — Offline Local Files
```java
// Game files ko app/src/main/assets/pacman/ me rakho
WebView webView = findViewById(R.id.webView);
webView.getSettings().setJavaScriptEnabled(true);
webView.getSettings().setAllowFileAccess(true);
webView.loadUrl("file:///android_asset/pacman/index.html");
```

### Option C — Capacitor.js (Recommended)
```bash
npm install @capacitor/core @capacitor/android
npx cap init
npx cap add android
# Game files ko www/ folder me rakho
npx cap sync
npx cap open android
```

### Touch Controls (Mobile ke liye)
> Abhi game me sirf keyboard controls hain.
> Mobile touch swipe support add karna ho to `index.js` me ye add karo:

```javascript
// Touch swipe detection — index.js ke end me add karo
var touchStartX, touchStartY;
document.addEventListener('touchstart', function(e) {
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
});
document.addEventListener('touchend', function(e) {
    var dx = e.changedTouches[0].clientX - touchStartX;
    var dy = e.changedTouches[0].clientY - touchStartY;
    if (Math.abs(dx) > Math.abs(dy)) {
        player.control = { orientation: dx > 0 ? 0 : 2 }; // right : left
    } else {
        player.control = { orientation: dy > 0 ? 1 : 3 }; // down : up
    }
});
```

---

## 🔄 UPDATE LOG (Yahan updates track karo)

| Date | File Changed | Kya Badla |
|---|---|---|
| 2025 | `index.html` | External links/redirects hataye, Hindi UI add ki, WebView compatible banaya |
| 2025 | `index.css` | Indian Tricolor theme apply ki, dark background, responsive mobile layout |
| 2025 | `MASTER.md` | Ye documentation file create ki |

---

## ⚠️ IMPORTANT NOTES (Dhyan Rakho)

1. **Script order mat badlo** — `game.js` pehle, `index.js` baad me load hona chahiye
2. **Font zaroori hai** — `PressStart2P.ttf` delete mat karo, game start nahi hogi
3. **Canvas size** — Game internally 960x640 par run karti hai, CSS se scale hoti hai
4. **Map array size** — Har level ka map 28 columns × 31 rows hona chahiye
5. **Ghost spawn** — Ghosts `coord {x:12+i, y:14}` par spawn hote hain (map center)
6. **Player spawn** — Player `coord {x:13.5, y:23}` par spawn hota hai
7. **Power pellet format** — `goods` object me key format hai `'x,y': 1`
8. **Cache flag** — Wall map me `cache:true` hai (performance ke liye), beans map me nahi

---

## 🚀 NEW LEVEL BANANA HO TO (Quick Guide)

```javascript
// index.js me _COIGIG array me naya object add karo:
{
    map: [
        // 28 columns × 31 rows ka 2D array
        // 1 = wall, 0 = path, 2 = ghost house
        [1,1,1,...],  // Row 0 (top border — saari 1s)
        [1,0,0,...],  // Row 1
        // ...
        [1,1,1,...],  // Row 30 (bottom border — saari 1s)
    ],
    wall_color: '#FF0000',  // Apna color choose karo
    goods: {
        '1,3': 1,    // Power pellet positions (x,y)
        '26,3': 1,
        '1,27': 1,
        '26,27': 1
    }
}
```

---

*Last Updated: 2025 | Project: Pac-Man HTML5 Indian Edition*

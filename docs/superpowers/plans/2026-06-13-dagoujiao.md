# 大狗叫 · 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development to implement. Steps use `- [ ]` syntax for tracking.

**Goal:** 构建大狗叫魔性网站——三个动物模块，狂点触发声音叠加、死亡机制、哈基米解锁。

**Architecture:** Flask 渲染单 HTML → 前端 JS 处理所有交互（Web Audio 合成声音 + localStorage 计数持久化 + 死亡检测 + 解锁逻辑）。

**Tech Stack:** Flask 3.x + gunicorn + vanilla JS + Web Audio API

---

## 文件结构

```
大狗叫/
├── app.py                         ← 保留，不变
├── requirements.txt               ← 保留
├── Procfile                       ← 保留
├── templates/
│   └── index.html                 ← 🔥 重写（核心文件）
├── static/
│   ├── images/
│   │   ├── dog.svg               ← 默认狗图
│   │   ├── dog-dead.svg          ← 死狗图
│   │   ├── chicken.svg           ← 默认鸡图
│   │   ├── chicken-dead.svg      ← 死鸡图
│   │   └── cat.svg               ← 哈基米图
│   └── sounds/
│       ├── dog-bark.mp3           ← 用户替换
│       ├── chicken-cluck.mp3      ← 用户替换
│       └── cat-meow.mp3           ← 用户替换
└── docs/
    └── superpowers/
        ├── specs/2026-06-13-dagoujiao-design.md
        └── plans/2026-06-13-dagoujiao.md
```

---

### Task 1: 重写 index.html（核心）

**Files:**
- Rewrite: `D:\claudecode牛逼666\大狗叫\templates\index.html`

这是唯一的实现任务——所有逻辑在一个 HTML 文件里。

- [ ] **Step 1: 创建 SVG 占位图**

为三种动物 + 死狗/死鸡创建默认 SVG：

```bash
cd D:\claudecode牛逼666\大狗叫\static\images
# 已有 dog.svg, chicken.svg, cat.svg
```

创建死狗 SVG (`dog-dead.svg`)：

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 200">
  <circle cx="100" cy="100" r="80" fill="#333" opacity="0.5"/>
  <text x="100" y="90" text-anchor="middle" font-size="60">💀</text>
  <text x="100" y="140" text-anchor="middle" font-size="16" fill="#888">RIP</text>
</svg>
```

创建死鸡 SVG (`chicken-dead.svg`)：同上，替换文字为「🐔💀」

- [ ] **Step 2: 写 HTML 结构**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8"><meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>大狗叫 🐕</title>
<style>
/* CSS 见下方完整代码 */
</style>
</head>
<body>
<div class="nav" id="navBar">
  <button class="active" onclick="switchTab('dog')">🐕 大狗叫</button>
  <button onclick="switchTab('chicken')">🐔 叮咚鸡</button>
  <button id="navCat" style="display:none" onclick="switchTab('cat')">🐱 哈基米</button>
</div>
<div class="wrap">
  <!-- Dog tab -->
  <div id="tab-dog" class="tab active">
    <div class="animal-wrap">
      <img id="dogImg" src="/static/images/dog.svg" class="animal-img" onclick="handleClick('dog')">
      <div class="bark-stack" id="dogBarkStack"></div>
    </div>
    <div class="counter">大狗叫了 <span id="dogCount">0</span> 次</div>
    <div class="warn hidden" id="dogWarn">大狗叫有点不舒服😰</div>
    <div id="dogDead" class="dead hidden">
      <div class="dead-title">💀 大狗叫被你叫死了</div>
      <button class="reset-btn" onclick="revive('dog')">重新开始</button>
    </div>
  </div>
  <!-- Chicken tab (same structure) -->
  <div id="tab-chicken" class="tab">
    <!-- ... 同狗结构 ... -->
  </div>
  <!-- Cat tab -->
  <div id="tab-cat" class="tab">
    <!-- 无死亡机制，无限点击 -->
  </div>
</div>
</body>
<script>
/* JS 见下方完整代码 */
</script>
</html>
```

- [ ] **Step 3: 写完整 CSS（魔性深色风）**

```css
:root { --bg:#1a1a2e; --gold:#f5c518; --pink:#ff6b9d; --green:#4ade80; --red:#e85d75; }
* { margin:0; padding:0; box-sizing:border-box; }
body { background:var(--bg); color:#fff; font-family:'Segoe UI','Noto Sans SC',sans-serif; min-height:100vh; text-align:center; overflow-x:hidden; }
.nav { display:flex; background:#111; position:sticky; top:0; z-index:100; }
.nav button { flex:1; padding:16px; background:none; border:none; color:#777; font-size:1.1em; cursor:pointer; transition:all .25s; border-bottom:2px solid transparent; font-family:inherit; }
.nav button:hover { color:#aaa; }
.nav button.active { color:var(--gold); border-bottom-color:var(--gold); background:rgba(245,197,24,0.05); }
.wrap { max-width:600px; margin:0 auto; padding:30px 20px; }
.tab { display:none; } .tab.active { display:block; }
.animal-wrap { position:relative; margin:40px auto; }
.animal-img { width:240px; height:240px; object-fit:contain; cursor:pointer; user-select:none; transition:all .1s; -webkit-user-drag:none; }
.animal-img:active { transform:scale(0.9); }
.animal-img.shake { animation:shake 0.3s; }
.animal-img.dead-img { filter:grayscale(1); opacity:0.4; }
@keyframes shake { 0%,100%{transform:translateX(0)} 25%{transform:translateX(-5px)} 75%{transform:translateX(5px)} }
.counter { font-size:2em; font-weight:900; margin:16px 0; }
.warn { color:var(--gold); font-size:1.1em; padding:12px; animation:pulse 0.8s infinite; }
@keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.3} }
.hidden { display:none; }
.dead { padding:40px; }
.dead-title { font-size:2em; font-weight:900; color:var(--red); margin-bottom:20px; }
.reset-btn { padding:14px 40px; background:var(--gold); color:#1a1a24; border:none; border-radius:10px; font-size:1.1em; font-weight:700; cursor:pointer; }
.floating-text { position:absolute; font-size:2em; font-weight:900; pointer-events:none; animation:floatUp 0.8s ease-out forwards; }
@keyframes floatUp { 0%{opacity:1;transform:translateY(0) scale(1)} 100%{opacity:0;transform:translateY(-60px) scale(1.3)} }
.bark-stack { position:absolute; top:20px; left:50%; transform:translateX(-50%); pointer-events:none; }
.unlock-banner { position:fixed; top:0; left:0; right:0; background:linear-gradient(135deg,var(--gold),var(--green)); color:#1a1a24; padding:20px; font-size:1.4em; font-weight:900; z-index:999; animation:slideDown 0.5s; }
@keyframes slideDown { 0%{transform:translateY(-100%)} 100%{transform:translateY(0)} }
@media(max-width:600px){ .animal-img{width:180px;height:180px} .counter{font-size:1.4em} }
```

- [ ] **Step 4: 写完整 JavaScript**

```javascript
// ─── STATE ───
const COUNTS = { dog:0, chicken:0, cat:0 };
let catUnlocked = false;
const RAPID = { dog:{count:0,timer:null,warning:false,dead:false}, chicken:{count:0,timer:null,warning:false,dead:false} };
const RAPID_THRESHOLD = 20;
const RAPID_WINDOW = 1500; // ms
const RECOVER_TIME = 5000; // ms

// ─── AUDIO ───
const ctx = new (window.AudioContext||window.webkitAudioContext)();
const SOUNDS = { dog:null, chicken:null, cat:null };

function synthBark(type) {
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.connect(gain); gain.connect(ctx.destination);
  if (type==='dog') {
    osc.type='sawtooth'; osc.frequency.setValueAtTime(300,ctx.currentTime);
    osc.frequency.linearRampToValueAtTime(80,ctx.currentTime+0.25);
    gain.gain.setValueAtTime(0.3,ctx.currentTime);
    gain.gain.linearRampToValueAtTime(0,ctx.currentTime+0.3);
    osc.start(); osc.stop(ctx.currentTime+0.3);
  } else if (type==='chicken') {
    osc.type='square'; osc.frequency.setValueAtTime(600,ctx.currentTime);
    osc.frequency.setValueAtTime(400,ctx.currentTime+0.04);
    osc.frequency.setValueAtTime(800,ctx.currentTime+0.08);
    osc.frequency.setValueAtTime(500,ctx.currentTime+0.12);
    gain.gain.setValueAtTime(0.15,ctx.currentTime);
    gain.gain.linearRampToValueAtTime(0,ctx.currentTime+0.18);
    osc.start(); osc.stop(ctx.currentTime+0.18);
  } else {
    osc.type='sine'; osc.frequency.setValueAtTime(500,ctx.currentTime);
    osc.frequency.linearRampToValueAtTime(700,ctx.currentTime+0.1);
    gain.gain.setValueAtTime(0.2,ctx.currentTime);
    gain.gain.linearRampToValueAtTime(0,ctx.currentTime+0.2);
    osc.start(); osc.stop(ctx.currentTime+0.2);
  }
}

function playSound(type) {
  if (SOUNDS[type]) { const a=new Audio(SOUNDS[type]); a.volume=0.5; a.play(); }
  else { synthBark(type); }
}

// ─── CORE CLICK ───
function handleClick(type) {
  if (type==='cat') { catClick(); return; }
  if (RAPID[type].dead) return;
  
  COUNTS[type]++; updateCount(type);
  playSound(type);
  spawnFloating(type);
  
  // Rapid-fire detection
  RAPID[type].count++;
  clearTimeout(RAPID[type].timer);
  
  if (RAPID[type].count >= RAPID_THRESHOLD) {
    if (!RAPID[type].warning) {
      // First warning
      RAPID[type].warning = true;
      document.getElementById(type+'Warn').classList.remove('hidden');
    } else {
      // GAME OVER
      killAnimal(type);
      return;
    }
  }
  
  // Reset rapid counter after pause
  RAPID[type].timer = setTimeout(() => {
    RAPID[type].count = 0;
    RAPID[type].warning = false;
    document.getElementById(type+'Warn').classList.add('hidden');
    document.getElementById(type+'Img').classList.remove('shake');
  }, RECOVER_TIME);
  
  // Unlock cat at 100
  if (type==='dog' && COUNTS.dog >= 100 && !catUnlocked) {
    unlockCat();
  }
  
  // Shake animation
  if (RAPID[type].count >= 15) {
    document.getElementById(type+'Img').classList.add('shake');
    setTimeout(()=>document.getElementById(type+'Img').classList.remove('shake'),300);
  }
}

function killAnimal(type) {
  RAPID[type].dead = true;
  document.getElementById(type+'Img').src = `/static/images/${type}-dead.svg`;
  document.getElementById(type+'Img').classList.add('dead-img');
  document.getElementById(type+'Warn').classList.add('hidden');
  document.getElementById(type+'Dead').classList.remove('hidden');
  COUNTS[type] = 0; updateCount(type);
}

function revive(type) {
  RAPID[type].dead = false; RAPID[type].count = 0; RAPID[type].warning = false;
  document.getElementById(type+'Img').src = `/static/images/${type}.svg`;
  document.getElementById(type+'Img').classList.remove('dead-img');
  document.getElementById(type+'Dead').classList.add('hidden');
  updateCount(type);
}

function catClick() {
  COUNTS.cat++; updateCount('cat');
  playSound('cat');
  spawnFloating('cat');
}

function unlockCat() {
  catUnlocked = true;
  document.getElementById('navCat').style.display = 'block';
  const banner = document.createElement('div');
  banner.className = 'unlock-banner';
  banner.innerHTML = '🐱 哈基米登场！隐藏角色解锁！';
  document.body.prepend(banner);
  setTimeout(()=>banner.remove(), 3000);
  localStorage.setItem('dagoujiao_cat', '1');
}

// ─── UI HELPERS ───
function updateCount(type) {
  document.getElementById(type+'Count').textContent = COUNTS[type];
  save();
}

function spawnFloating(type) {
  const texts = { dog:'汪!', chicken:'咯!', cat:'喵~' };
  const colors = { dog:'var(--gold)', chicken:'var(--pink)', cat:'var(--green)' };
  const stack = document.getElementById(type+'BarkStack');
  const el = document.createElement('span');
  el.className = 'floating-text';
  el.textContent = texts[type]; el.style.color = colors[type];
  el.style.position = 'relative';
  stack.appendChild(el);
  setTimeout(()=>el.remove(), 800);
}

function switchTab(type) {
  document.querySelectorAll('.nav button').forEach(b=>b.classList.remove('active'));
  document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
  document.getElementById('tab-'+type).classList.add('active');
  event.target.classList.add('active');
}

// ─── PERSISTENCE ───
function save() {
  localStorage.setItem('dagoujiao_counts', JSON.stringify(COUNTS));
}

function load() {
  try {
    const s = JSON.parse(localStorage.getItem('dagoujiao_counts'));
    if (s) { COUNTS.dog=s.dog||0; COUNTS.chicken=s.chicken||0; COUNTS.cat=s.cat||0; }
    if (localStorage.getItem('dagoujiao_cat')==='1') { catUnlocked=true; document.getElementById('navCat').style.display='block'; }
    updateCount('dog'); updateCount('chicken'); updateCount('cat');
  } catch(e) {}
}

// ─── CUSTOM FILES ───
function tryCustomAudio(type, path) {
  const a = new Audio(); a.oncanplaythrough = ()=>SOUNDS[type]=path;
  a.src = path;
}
tryCustomAudio('dog','/static/sounds/dog-bark.mp3');
tryCustomAudio('chicken','/static/sounds/chicken-cluck.mp3');
tryCustomAudio('cat','/static/sounds/cat-meow.mp3');

// ─── INIT ───
load();
```

- [ ] **Step 5: 验证完整 HTML 文件**

确保 HTML 中三个 tab 结构完整（dog 有完整交互、chicken 同结构、cat 无死亡）。参考 spec 的 Game Over 界面和恢复机制。

- [ ] **Step 6: 本地测试**

```bash
cd D:\claudecode牛逼666\大狗叫
python app.py
# 浏览器打开 http://localhost:5000
# 测试: 点狗20次→警告→继续点→Game Over
# 测试: 点狗100次→哈基米解锁
# 测试: 切换标签、计数持久化
```

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "feat: 大狗叫完整实现 - 三模块/死亡机制/哈基米解锁"
```

---

### Task 2: 部署

- [ ] **Step 1: 推送到 GitHub**

```bash
cd D:\claudecode牛逼666\大狗叫
git remote add origin https://github.com/44552wwwww/dagoujiao.git
git push -u origin master
```

- [ ] **Step 2: Railway 部署**

在 Railway 中 New Project → Deploy from GitHub → 选 `44552wwwww/dagoujiao`
自动检测 Flask → 生成域名

- [ ] **Step 3: 验证线上**

打开 Railway 分配的 URL，测试所有功能。

---

### Task 3: 用户自定义（替换图片/声音）

用户替换以下文件即可自定义：
- `static/images/dog.svg` → 狗图
- `static/images/dog-dead.svg` → 死狗图
- `static/images/chicken.svg` → 鸡图
- `static/images/chicken-dead.svg` → 死鸡图
- `static/images/cat.svg` → 哈基米图
- `static/sounds/dog-bark.mp3` → 狗叫
- `static/sounds/chicken-cluck.mp3` → 鸡叫
- `static/sounds/cat-meow.mp3` → 猫叫

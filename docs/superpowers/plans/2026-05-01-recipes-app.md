# Recipes App Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a shared recipe link collection at food.drewbefree.com — a browsable card grid with search/filter and a password-gated admin for adding, editing, and deleting recipes.

**Architecture:** Pure HTML/CSS/JS on GitHub Pages. Recipes stored in Supabase (`recipes` table). Browse page (`index.html`) is public and read-only; admin page (`admin.html`) is gated by a shared password stored in `config.js` alongside Supabase credentials.

**Tech Stack:** HTML/CSS/JS, Supabase JS v2 (CDN), GitHub Pages

---

## File Map

| File | Purpose |
|------|---------|
| `index.html` | Browse page — search bar, category filter tabs, recipe card grid |
| `admin.html` | Admin page — password gate, recipe list, add/edit/delete form |
| `config.js` | Supabase URL, anon key, admin password (gitignored) |
| `config.example.js` | Template for config.js |
| `.gitignore` | Ignores config.js |

---

## Supabase Setup (do this first, manually)

Run in the Supabase SQL editor for your existing project:

```sql
create table recipes (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  url text,
  category text not null default 'Uncategorized',
  tags text[] default '{}',
  description text default '',
  image_url text default '',
  created_at timestamptz default now()
);

-- Allow public reads, no auth writes (admin enforced client-side via password)
alter table recipes enable row level security;
create policy "Public read" on recipes for select using (true);
create policy "Service write" on recipes for all using (true) with check (true);
```

> Note: This uses anon key for all operations. The admin password is enforced in the browser only — this is appropriate for a personal/family app where data is not sensitive.

---

## Task 1: Initialize repo

**Files:**
- Create: `.gitignore`
- Create: `config.example.js`
- Create: `README.md`

- [ ] **Step 1: Init git repo**

```powershell
cd C:\Users\drewb\Documents\GitHub\recipes
git init
git checkout -b dev
```

- [ ] **Step 2: Create .gitignore**

```
config.js
```

Save to `.gitignore`.

- [ ] **Step 3: Create config.example.js**

```js
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key-here';
const ADMIN_PASSWORD = 'your-admin-password-here';
```

Save to `config.example.js`.

- [ ] **Step 4: Create config.js from example**

Copy `config.example.js` to `config.js` and fill in the real Supabase URL, anon key, and a chosen admin password.

- [ ] **Step 5: Create README.md**

```markdown
# Recipes

Family recipe collection at food.drewbefree.com.

## Setup
1. Copy `config.example.js` to `config.js` and fill in credentials.
2. Run the SQL in `docs/superpowers/plans/2026-05-01-recipes-app.md` in Supabase.
3. Push to GitHub, enable GitHub Pages from `main`.
```

- [ ] **Step 6: Initial commit**

```powershell
git add .gitignore config.example.js README.md docs/
git commit -m "init: scaffold recipes repo"
```

---

## Task 2: Browse page — structure and styles

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create index.html with full HTML, CSS, and placeholder JS**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Recipes · food.drewbefree.com</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🍽️</text></svg>">
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@400;700;900&family=Exo+2:wght@300;400;600&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #050a0e;
    --bg2: #080f14;
    --panel: #0a1520;
    --border: #0d2535;
    --accent: #f97316;
    --accent2: #00ff88;
    --text: #c8e8f0;
    --text-dim: #3a6070;
    --text-mid: #6a9ab0;
    --glow: rgba(249,115,22,0.15);
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { background: var(--bg); color: var(--text); font-family: 'Share Tech Mono', monospace; min-height: 100vh; overflow-x: hidden; }
  body::before { content:''; position:fixed; inset:0; background:repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,0,0,0.07) 2px,rgba(0,0,0,0.07) 4px); pointer-events:none; z-index:1000; }

  .wrap { max-width: 1100px; margin: 0 auto; padding: 0 20px 80px; position: relative; z-index: 1; }

  header { padding: 48px 0 36px; border-bottom: 1px solid var(--border); margin-bottom: 40px; display: flex; align-items: flex-end; justify-content: space-between; flex-wrap: wrap; gap: 16px; }
  .logo-eyebrow { font-size: 0.65rem; color: var(--accent); letter-spacing: 4px; text-transform: uppercase; margin-bottom: 8px; }
  .logo { font-family: 'Orbitron', sans-serif; font-size: clamp(1.8rem,5vw,3rem); font-weight: 900; color: #fff; letter-spacing: 2px; }
  .logo span { color: var(--accent); text-shadow: 0 0 24px rgba(249,115,22,0.5); }
  .logo-sub { font-size: 0.68rem; color: var(--text-mid); letter-spacing: 3px; margin-top: 8px; }
  .header-link { font-size: 0.65rem; color: var(--text-dim); letter-spacing: 2px; text-decoration: none; border: 1px solid var(--border); padding: 6px 14px; border-radius: 2px; transition: color 0.2s, border-color 0.2s; }
  .header-link:hover { color: var(--accent); border-color: var(--accent); }

  .controls { display: flex; gap: 12px; flex-wrap: wrap; align-items: center; margin-bottom: 28px; }
  .search-wrap { flex: 1; min-width: 200px; position: relative; }
  .search-wrap::before { content: '⌕'; position: absolute; left: 12px; top: 50%; transform: translateY(-50%); color: var(--text-dim); font-size: 1rem; pointer-events: none; }
  #search { width: 100%; background: var(--panel); border: 1px solid var(--border); color: var(--text); font-family: 'Share Tech Mono', monospace; font-size: 0.8rem; padding: 10px 12px 10px 34px; border-radius: 2px; outline: none; letter-spacing: 1px; transition: border-color 0.2s; }
  #search:focus { border-color: var(--accent); }
  #search::placeholder { color: var(--text-dim); }

  .cats { display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 32px; }
  .cat-btn { font-family: 'Share Tech Mono', monospace; font-size: 0.62rem; letter-spacing: 2px; text-transform: uppercase; padding: 5px 14px; border: 1px solid var(--border); background: transparent; color: var(--text-dim); border-radius: 2px; cursor: pointer; transition: color 0.2s, border-color 0.2s, background 0.2s; }
  .cat-btn:hover, .cat-btn.active { border-color: var(--accent); color: var(--accent); background: rgba(249,115,22,0.07); }

  .recipe-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 14px; }

  .recipe-card { background: var(--panel); border: 1px solid var(--border); border-radius: 4px; padding: 22px; position: relative; text-decoration: none; color: inherit; display: flex; flex-direction: column; transition: border-color 0.2s, transform 0.2s, box-shadow 0.2s; overflow: hidden; }
  .recipe-card::before { content:''; position:absolute; top:0; left:0; right:0; height:2px; background:linear-gradient(90deg,transparent,var(--accent),transparent); opacity:0; transition:opacity 0.3s; }
  .recipe-card:hover { border-color: var(--accent); transform: translateY(-3px); box-shadow: 0 8px 32px rgba(0,0,0,0.5), 0 0 20px var(--glow); }
  .recipe-card:hover::before { opacity: 1; }

  .rc-category { font-size: 0.58rem; letter-spacing: 3px; text-transform: uppercase; color: var(--accent); margin-bottom: 10px; opacity: 0.8; }
  .rc-title { font-family: 'Orbitron', sans-serif; font-size: 0.85rem; font-weight: 700; color: #fff; letter-spacing: 1px; margin-bottom: 8px; line-height: 1.4; }
  .rc-desc { font-family: 'Exo 2', sans-serif; font-size: 0.78rem; color: var(--text-mid); line-height: 1.6; margin-bottom: 14px; font-weight: 300; flex: 1; }
  .rc-tags { display: flex; flex-wrap: wrap; gap: 5px; margin-bottom: 14px; }
  .rc-tag { font-size: 0.56rem; letter-spacing: 2px; text-transform: uppercase; padding: 2px 7px; border: 1px solid var(--border); color: var(--text-dim); border-radius: 2px; }
  .rc-footer { display: flex; align-items: center; justify-content: space-between; padding-top: 12px; border-top: 1px solid var(--border); margin-top: auto; }
  .rc-launch { font-size: 0.65rem; letter-spacing: 2px; color: var(--accent); text-transform: uppercase; }
  .rc-no-link { font-size: 0.65rem; letter-spacing: 2px; color: var(--text-dim); text-transform: uppercase; }

  .rc-img { width: 100%; height: 120px; object-fit: cover; border-radius: 2px; margin-bottom: 14px; border: 1px solid var(--border); }

  .empty-state { text-align: center; padding: 60px 20px; color: var(--text-dim); font-size: 0.78rem; letter-spacing: 2px; }

  .loading { text-align: center; padding: 60px 20px; color: var(--text-dim); font-size: 0.78rem; letter-spacing: 2px; animation: pulse 1.5s ease-in-out infinite; }
  @keyframes pulse { 0%,100%{opacity:0.4} 50%{opacity:1} }

  footer { padding-top: 20px; border-top: 1px solid var(--border); display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px; font-size: 0.6rem; color: var(--text-dim); letter-spacing: 2px; margin-top: 40px; }

  @media (max-width: 600px) {
    .recipe-grid { grid-template-columns: 1fr; }
    header { padding: 32px 0 24px; }
  }
</style>
</head>
<body>
<div class="wrap">

  <header>
    <div>
      <div class="logo-eyebrow">// RECIPE COLLECTION · ONLINE</div>
      <div class="logo">FOOD<span>.</span>DB</div>
      <div class="logo-sub">DREWBEFREE · ATL, GA</div>
    </div>
    <a href="admin.html" class="header-link">ADMIN ›</a>
  </header>

  <div class="controls">
    <div class="search-wrap">
      <input type="text" id="search" placeholder="Search recipes..." autocomplete="off">
    </div>
  </div>

  <div class="cats" id="cats">
    <button class="cat-btn active" data-cat="All">All</button>
  </div>

  <div class="recipe-grid" id="grid">
    <div class="loading">LOADING RECIPES...</div>
  </div>

  <footer>
    <span>FOOD.DREWBEFREE.COM</span>
    <span id="recipe-count"></span>
  </footer>

</div>

<script src="config.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
const { createClient } = supabase;
const db = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

const CATEGORIES = ['Breakfast','Lunch','Dinner','Desserts','Snacks','Sides','Drinks','Uncategorized'];

let allRecipes = [];
let activeCategory = 'All';

async function loadRecipes() {
  const { data, error } = await db
    .from('recipes')
    .select('*')
    .order('title', { ascending: true });
  if (error) { document.getElementById('grid').innerHTML = '<div class="empty-state">ERROR LOADING RECIPES</div>'; return; }
  allRecipes = data || [];
  buildCategoryTabs();
  render();
}

function buildCategoryTabs() {
  const usedCats = [...new Set(allRecipes.map(r => r.category))].sort();
  const catsEl = document.getElementById('cats');
  catsEl.innerHTML = '<button class="cat-btn active" data-cat="All">All</button>';
  usedCats.forEach(cat => {
    const btn = document.createElement('button');
    btn.className = 'cat-btn';
    btn.dataset.cat = cat;
    btn.textContent = cat;
    btn.addEventListener('click', () => setCategory(cat));
    catsEl.appendChild(btn);
  });
  catsEl.querySelector('[data-cat="All"]').addEventListener('click', () => setCategory('All'));
}

function setCategory(cat) {
  activeCategory = cat;
  document.querySelectorAll('.cat-btn').forEach(b => b.classList.toggle('active', b.dataset.cat === cat));
  render();
}

function render() {
  const query = document.getElementById('search').value.toLowerCase().trim();
  let recipes = allRecipes;
  if (activeCategory !== 'All') recipes = recipes.filter(r => r.category === activeCategory);
  if (query) recipes = recipes.filter(r =>
    r.title.toLowerCase().includes(query) ||
    (r.description || '').toLowerCase().includes(query) ||
    (r.tags || []).some(t => t.toLowerCase().includes(query))
  );

  const grid = document.getElementById('grid');
  document.getElementById('recipe-count').textContent = `${recipes.length} RECIPE${recipes.length !== 1 ? 'S' : ''}`;

  if (recipes.length === 0) {
    grid.innerHTML = '<div class="empty-state">NO RECIPES FOUND</div>';
    return;
  }

  grid.innerHTML = recipes.map(r => `
    <a class="recipe-card" ${r.url ? `href="${r.url}" target="_blank" rel="noopener"` : ''}>
      ${r.image_url ? `<img class="rc-img" src="${r.image_url}" alt="" loading="lazy">` : ''}
      <div class="rc-category">${r.category}</div>
      <div class="rc-title">${r.title}</div>
      ${r.description ? `<div class="rc-desc">${r.description}</div>` : '<div class="rc-desc"></div>'}
      ${r.tags && r.tags.length ? `<div class="rc-tags">${r.tags.map(t => `<span class="rc-tag">${t}</span>`).join('')}</div>` : ''}
      <div class="rc-footer">
        ${r.url ? `<span class="rc-launch">VIEW RECIPE ›</span>` : `<span class="rc-no-link">NO LINK</span>`}
      </div>
    </a>
  `).join('');
}

document.getElementById('search').addEventListener('input', render);
loadRecipes();
</script>
</body>
</html>
```

- [ ] **Step 2: Verify it loads in browser**

Open `index.html` in a browser. You should see the header, search bar, and "LOADING RECIPES..." (it will error without config.js filled in — that's expected). With config.js filled in, you should see recipe cards or "NO RECIPES FOUND" if the table is empty.

- [ ] **Step 3: Commit**

```powershell
git add index.html
git commit -m "feat: add browse page with search, category filter, and card grid"
```

---

## Task 3: Admin page — password gate and recipe list

**Files:**
- Create: `admin.html`

- [ ] **Step 1: Create admin.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Admin · Recipes</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🔧</text></svg>">
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@400;700;900&family=Exo+2:wght@300;400;600&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #050a0e;
    --bg2: #080f14;
    --panel: #0a1520;
    --border: #0d2535;
    --accent: #f97316;
    --accent2: #00ff88;
    --danger: #ef4444;
    --text: #c8e8f0;
    --text-dim: #3a6070;
    --text-mid: #6a9ab0;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { background: var(--bg); color: var(--text); font-family: 'Share Tech Mono', monospace; min-height: 100vh; }
  body::before { content:''; position:fixed; inset:0; background:repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,0,0,0.07) 2px,rgba(0,0,0,0.07) 4px); pointer-events:none; z-index:1000; }

  .wrap { max-width: 900px; margin: 0 auto; padding: 0 20px 80px; position: relative; z-index: 1; }

  /* --- Password gate --- */
  #gate { display: flex; align-items: center; justify-content: center; min-height: 100vh; }
  .gate-box { background: var(--panel); border: 1px solid var(--border); border-radius: 4px; padding: 40px; width: 100%; max-width: 360px; text-align: center; }
  .gate-title { font-family: 'Orbitron', sans-serif; font-size: 1rem; font-weight: 700; color: #fff; letter-spacing: 3px; margin-bottom: 8px; }
  .gate-sub { font-size: 0.62rem; color: var(--text-dim); letter-spacing: 2px; margin-bottom: 28px; }
  .gate-input { width: 100%; background: var(--bg); border: 1px solid var(--border); color: var(--text); font-family: 'Share Tech Mono', monospace; font-size: 0.85rem; padding: 10px 14px; border-radius: 2px; outline: none; text-align: center; letter-spacing: 3px; margin-bottom: 16px; transition: border-color 0.2s; }
  .gate-input:focus { border-color: var(--accent); }
  .gate-btn { width: 100%; background: var(--accent); border: none; color: #fff; font-family: 'Share Tech Mono', monospace; font-size: 0.72rem; letter-spacing: 3px; padding: 11px; border-radius: 2px; cursor: pointer; text-transform: uppercase; transition: opacity 0.2s; }
  .gate-btn:hover { opacity: 0.85; }
  .gate-error { font-size: 0.62rem; color: var(--danger); letter-spacing: 2px; margin-top: 12px; min-height: 18px; }

  /* --- Admin UI (hidden until authenticated) --- */
  #app { display: none; }

  header { padding: 40px 0 28px; border-bottom: 1px solid var(--border); margin-bottom: 32px; display: flex; align-items: flex-end; justify-content: space-between; flex-wrap: wrap; gap: 12px; }
  .logo { font-family: 'Orbitron', sans-serif; font-size: 1.2rem; font-weight: 700; color: #fff; letter-spacing: 2px; }
  .logo span { color: var(--accent); }
  .header-actions { display: flex; gap: 10px; align-items: center; }
  .btn { font-family: 'Share Tech Mono', monospace; font-size: 0.65rem; letter-spacing: 2px; text-transform: uppercase; padding: 7px 16px; border-radius: 2px; cursor: pointer; transition: opacity 0.2s; }
  .btn-primary { background: var(--accent); border: none; color: #fff; }
  .btn-outline { background: transparent; border: 1px solid var(--border); color: var(--text-dim); text-decoration: none; display: inline-flex; align-items: center; }
  .btn:hover { opacity: 0.8; }

  /* --- Form --- */
  .form-panel { background: var(--panel); border: 1px solid var(--border); border-radius: 4px; padding: 28px; margin-bottom: 32px; display: none; }
  .form-panel.open { display: block; }
  .form-title { font-family: 'Orbitron', sans-serif; font-size: 0.8rem; letter-spacing: 2px; color: var(--accent); margin-bottom: 20px; }
  .form-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }
  .form-group { display: flex; flex-direction: column; gap: 6px; }
  .form-group.full { grid-column: 1 / -1; }
  label { font-size: 0.58rem; letter-spacing: 2px; color: var(--text-dim); text-transform: uppercase; }
  input[type=text], textarea, select {
    background: var(--bg); border: 1px solid var(--border); color: var(--text);
    font-family: 'Share Tech Mono', monospace; font-size: 0.8rem; padding: 9px 12px;
    border-radius: 2px; outline: none; width: 100%; letter-spacing: 1px; transition: border-color 0.2s;
  }
  input[type=text]:focus, textarea:focus, select:focus { border-color: var(--accent); }
  textarea { resize: vertical; min-height: 70px; }
  select option { background: var(--panel); }
  .form-actions { display: flex; gap: 10px; margin-top: 20px; }
  .btn-danger { background: transparent; border: 1px solid var(--danger); color: var(--danger); }

  /* --- Recipe list --- */
  .section-label { font-size: 0.62rem; color: var(--text-dim); letter-spacing: 4px; text-transform: uppercase; margin-bottom: 16px; display: flex; align-items: center; gap: 12px; }
  .section-label::after { content:''; flex:1; height:1px; background:var(--border); }

  .recipe-list { display: flex; flex-direction: column; gap: 8px; }
  .recipe-row { background: var(--panel); border: 1px solid var(--border); border-radius: 3px; padding: 14px 18px; display: flex; align-items: center; gap: 14px; }
  .rr-info { flex: 1; min-width: 0; }
  .rr-title { font-size: 0.82rem; color: #fff; letter-spacing: 1px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .rr-meta { font-size: 0.6rem; color: var(--text-dim); letter-spacing: 2px; margin-top: 3px; }
  .rr-actions { display: flex; gap: 8px; flex-shrink: 0; }
  .btn-sm { font-size: 0.58rem; padding: 4px 10px; }

  .empty-state { text-align: center; padding: 40px; color: var(--text-dim); font-size: 0.75rem; letter-spacing: 2px; }
  .loading { text-align: center; padding: 40px; color: var(--text-dim); font-size: 0.75rem; letter-spacing: 2px; animation: pulse 1.5s ease-in-out infinite; }
  @keyframes pulse { 0%,100%{opacity:0.4} 50%{opacity:1} }

  .toast { position: fixed; bottom: 24px; right: 24px; background: var(--accent2); color: #050a0e; font-size: 0.68rem; letter-spacing: 2px; padding: 10px 20px; border-radius: 2px; z-index: 9999; opacity: 0; transition: opacity 0.3s; pointer-events: none; }
  .toast.show { opacity: 1; }

  @media (max-width: 600px) {
    .form-grid { grid-template-columns: 1fr; }
    .form-group.full { grid-column: 1; }
  }
</style>
</head>
<body>

<!-- Password Gate -->
<div id="gate">
  <div class="gate-box">
    <div class="gate-title">ADMIN ACCESS</div>
    <div class="gate-sub">RECIPE COLLECTION · RESTRICTED</div>
    <input type="password" class="gate-input" id="pw-input" placeholder="••••••••" autocomplete="current-password">
    <button class="gate-btn" id="pw-submit">ENTER</button>
    <div class="gate-error" id="pw-error"></div>
  </div>
</div>

<!-- Admin App -->
<div id="app">
<div class="wrap">

  <header>
    <div class="logo">FOOD<span>.</span>DB <span style="font-size:0.65rem;color:var(--text-dim);letter-spacing:3px;">ADMIN</span></div>
    <div class="header-actions">
      <a href="index.html" class="btn btn-outline">← BROWSE</a>
      <button class="btn btn-primary" id="btn-add">+ ADD RECIPE</button>
    </div>
  </header>

  <!-- Add / Edit Form -->
  <div class="form-panel" id="form-panel">
    <div class="form-title" id="form-title">ADD RECIPE</div>
    <div class="form-grid">
      <div class="form-group full">
        <label>Title *</label>
        <input type="text" id="f-title" placeholder="Grandma's Lasagna">
      </div>
      <div class="form-group full">
        <label>URL (link to recipe)</label>
        <input type="text" id="f-url" placeholder="https://...">
      </div>
      <div class="form-group">
        <label>Category</label>
        <select id="f-category">
          <option>Breakfast</option>
          <option>Lunch</option>
          <option>Dinner</option>
          <option>Desserts</option>
          <option>Snacks</option>
          <option>Sides</option>
          <option>Drinks</option>
          <option>Uncategorized</option>
        </select>
      </div>
      <div class="form-group">
        <label>Tags (comma-separated)</label>
        <input type="text" id="f-tags" placeholder="Italian, Pasta, Easy">
      </div>
      <div class="form-group full">
        <label>Description</label>
        <textarea id="f-desc" placeholder="Short note about this recipe..."></textarea>
      </div>
      <div class="form-group full">
        <label>Image URL (optional)</label>
        <input type="text" id="f-image" placeholder="https://...">
      </div>
    </div>
    <div class="form-actions">
      <button class="btn btn-primary" id="btn-save">SAVE</button>
      <button class="btn btn-outline" id="btn-cancel">CANCEL</button>
      <button class="btn btn-danger btn-sm" id="btn-delete" style="display:none;margin-left:auto;">DELETE</button>
    </div>
  </div>

  <div class="section-label">// ALL RECIPES</div>
  <div class="recipe-list" id="recipe-list">
    <div class="loading">LOADING...</div>
  </div>

</div>
</div>

<div class="toast" id="toast"></div>

<script src="config.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
const { createClient } = supabase;
const db = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

// --- Auth ---
const SESSION_KEY = 'recipe_admin_auth';
if (sessionStorage.getItem(SESSION_KEY) === 'true') unlock();

document.getElementById('pw-submit').addEventListener('click', tryAuth);
document.getElementById('pw-input').addEventListener('keydown', e => { if (e.key === 'Enter') tryAuth(); });

function tryAuth() {
  const val = document.getElementById('pw-input').value;
  if (val === ADMIN_PASSWORD) {
    sessionStorage.setItem(SESSION_KEY, 'true');
    unlock();
  } else {
    document.getElementById('pw-error').textContent = 'INCORRECT PASSWORD';
    document.getElementById('pw-input').value = '';
  }
}

function unlock() {
  document.getElementById('gate').style.display = 'none';
  document.getElementById('app').style.display = 'block';
  loadRecipes();
}

// --- Data ---
let recipes = [];
let editingId = null;

async function loadRecipes() {
  const { data, error } = await db.from('recipes').select('*').order('title');
  if (error) { document.getElementById('recipe-list').innerHTML = '<div class="empty-state">ERROR LOADING</div>'; return; }
  recipes = data || [];
  renderList();
}

function renderList() {
  const el = document.getElementById('recipe-list');
  if (recipes.length === 0) { el.innerHTML = '<div class="empty-state">NO RECIPES YET — ADD ONE ABOVE</div>'; return; }
  el.innerHTML = recipes.map(r => `
    <div class="recipe-row">
      <div class="rr-info">
        <div class="rr-title">${r.title}</div>
        <div class="rr-meta">${r.category}${r.url ? ' · ' + r.url.replace(/^https?:\/\//, '').split('/')[0] : ''}</div>
      </div>
      <div class="rr-actions">
        <button class="btn btn-outline btn-sm" onclick="editRecipe('${r.id}')">EDIT</button>
      </div>
    </div>
  `).join('');
}

// --- Form ---
document.getElementById('btn-add').addEventListener('click', () => openForm(null));
document.getElementById('btn-cancel').addEventListener('click', closeForm);
document.getElementById('btn-save').addEventListener('click', saveRecipe);
document.getElementById('btn-delete').addEventListener('click', deleteRecipe);

function openForm(id) {
  editingId = id;
  const panel = document.getElementById('form-panel');
  const deleteBtn = document.getElementById('btn-delete');
  document.getElementById('form-title').textContent = id ? 'EDIT RECIPE' : 'ADD RECIPE';
  deleteBtn.style.display = id ? 'inline-block' : 'none';

  if (id) {
    const r = recipes.find(x => x.id === id);
    document.getElementById('f-title').value = r.title;
    document.getElementById('f-url').value = r.url || '';
    document.getElementById('f-category').value = r.category;
    document.getElementById('f-tags').value = (r.tags || []).join(', ');
    document.getElementById('f-desc').value = r.description || '';
    document.getElementById('f-image').value = r.image_url || '';
  } else {
    document.getElementById('f-title').value = '';
    document.getElementById('f-url').value = '';
    document.getElementById('f-category').value = 'Dinner';
    document.getElementById('f-tags').value = '';
    document.getElementById('f-desc').value = '';
    document.getElementById('f-image').value = '';
  }

  panel.classList.add('open');
  document.getElementById('f-title').focus();
  panel.scrollIntoView({ behavior: 'smooth', block: 'start' });
}

function closeForm() {
  document.getElementById('form-panel').classList.remove('open');
  editingId = null;
}

function getFormData() {
  return {
    title: document.getElementById('f-title').value.trim(),
    url: document.getElementById('f-url').value.trim() || null,
    category: document.getElementById('f-category').value,
    tags: document.getElementById('f-tags').value.split(',').map(t => t.trim()).filter(Boolean),
    description: document.getElementById('f-desc').value.trim(),
    image_url: document.getElementById('f-image').value.trim() || null,
  };
}

async function saveRecipe() {
  const data = getFormData();
  if (!data.title) { alert('Title is required.'); return; }

  let error;
  if (editingId) {
    ({ error } = await db.from('recipes').update(data).eq('id', editingId));
  } else {
    ({ error } = await db.from('recipes').insert(data));
  }

  if (error) { alert('Error saving: ' + error.message); return; }
  showToast(editingId ? 'UPDATED' : 'ADDED');
  closeForm();
  loadRecipes();
}

async function editRecipe(id) {
  openForm(id);
}

async function deleteRecipe() {
  if (!confirm('Delete this recipe?')) return;
  const { error } = await db.from('recipes').delete().eq('id', editingId);
  if (error) { alert('Error deleting: ' + error.message); return; }
  showToast('DELETED');
  closeForm();
  loadRecipes();
}

function showToast(msg) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), 2500);
}
</script>
</body>
</html>
```

- [ ] **Step 2: Verify admin page in browser**

Open `admin.html`. You should see the password gate. Enter the password from `config.js` — it should unlock the admin UI, show the recipe list (empty), and let you add a recipe. Add one test recipe, then confirm it appears on `index.html`.

- [ ] **Step 3: Commit**

```powershell
git add admin.html
git commit -m "feat: add admin page with password gate and recipe CRUD"
```

---

## Task 4: Push to GitHub and add Command Center card

**Files:**
- Modify: `C:\Users\drewb\Documents\GitHub\DrewBeFree-Command-Center\index.html`

- [ ] **Step 1: Create GitHub repo and push**

Create a new repo named `recipes` on GitHub (DrewBeFree/recipes), then:

```powershell
cd C:\Users\drewb\Documents\GitHub\recipes
git remote add origin https://github.com/DrewBeFree/recipes.git
git push -u origin dev
```

Then merge to main:

```powershell
git checkout main
git merge dev
git push origin main
```

- [ ] **Step 2: Enable GitHub Pages**

In the GitHub repo settings → Pages → Source: Deploy from branch → `main` → `/ (root)` → Save.

- [ ] **Step 3: Add APP_007 card to Command Center**

In `DrewBeFree-Command-Center/index.html`, find the closing `</div>` of `.app-grid` (after the soccer-pickup card) and insert this card before it:

```html
    <a class="card" href="https://food.drewbefree.com"
       style="--card-accent:#f97316; --card-glow:rgba(249,115,22,0.1);">
      <div class="card-id">APP_007 // UTILITY</div>
      <span class="card-icon">🍽️</span>
      <div class="card-name">RECIPES</div>
      <div class="card-url">food.drewbefree.com</div>
      <div class="card-desc">Family recipe collection. Searchable card grid by category, with links to recipes hosted on other sites. Admin-editable with shared password.</div>
      <div class="card-tags"><span class="tag">Recipes</span><span class="tag">Supabase</span><span class="tag">Family</span></div>
      <div class="card-footer">
        <span class="card-meta">v1.0 · 2026-05-01</span>
        <span class="card-launch">LAUNCH ›</span>
      </div>
    </a>
```

- [ ] **Step 4: Update Command Center header count**

Find this line:
```
<div class="status-line">APPS <span class="on">6 DEPLOYED</span></div>
```
Change to:
```
<div class="status-line">APPS <span class="on">7 DEPLOYED</span></div>
```

- [ ] **Step 5: Update terminal scan line**

Find the `scan ./apps` line in the `.terminal` div. Add `recipes v1.0` to the end of the app list, matching the existing link style:

```html
· <a href="https://food.drewbefree.com" target="_blank" style="color:inherit;text-decoration:none;border-bottom:1px solid rgba(58,96,112,0.5);transition:color 0.2s" onmouseover="this.style.color='var(--text-mid)'" onmouseout="this.style.color='inherit'">recipes v1.0</a>
```

Also update the count at the start of that line from `6 apps deployed` to `7 apps deployed`.

- [ ] **Step 6: Commit Command Center changes**

```powershell
cd C:\Users\drewb\Documents\GitHub\DrewBeFree-Command-Center
git add index.html
git commit -m "feat: add APP_007 Recipes card, update app count to 7"
git push origin main
```

---

## Notes

- **DNS**: Point `food.drewbefree.com` as a CNAME to `drewbefree.github.io`, then add a `CNAME` file to the `recipes` repo containing `food.drewbefree.com`.
- **Supabase RLS**: The current setup allows anon writes (needed since there's no server-side auth). This is fine for a personal/family app. If you want to lock it down further in future, switch to Supabase Auth with a service role key in a serverless function.
- **Images**: `image_url` accepts any public image URL. For recipes linked from other sites, the original site's thumbnail URL works well.

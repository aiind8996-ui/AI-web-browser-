AI web browser 
<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>AI Web Browser</title>
<style>
  :root {
    --bg:#0f1724; --glass:rgba(255,255,255,0.05);
    --accent:#3b82f6; --text:#e6eef8; --muted:#a0aec0;
  }
  body.light {
    --bg:#f3f4f6; --glass:rgba(0,0,0,0.05);
    --accent:#2563eb; --text:#111827; --muted:#4b5563;
  }
  html,body {
    height:100%;margin:0;
    background:var(--bg);color:var(--text);
    font-family:"Segoe UI",Roboto,sans-serif;
  }
  .browser {display:flex;flex-direction:column;height:100vh;gap:10px;padding:10px;}
  .title-bar {
    width:100%;text-align:center;font-weight:700;
    font-size:18px;background:var(--glass);
    padding:8px;border-radius:8px;
  }
  .top {display:flex;align-items:center;gap:10px;position:relative;}
  .omnibox {flex:1;display:flex;align-items:center;gap:8px;background:var(--glass);padding:6px 8px;border-radius:8px;}
  .omnibox input {flex:1;background:transparent;border:0;outline:0;color:var(--text);font-size:15px;}
  .go {background:var(--accent);color:#fff;border:0;padding:6px 12px;border-radius:6px;cursor:pointer;}
  .settings-btn {background:var(--glass);border:0;color:var(--text);padding:6px 10px;border-radius:6px;cursor:pointer;font-size:15px;}
  .settings-menu {
    display:none;position:absolute;right:0;top:50px;background:rgba(10,15,25,0.95);
    border:1px solid rgba(255,255,255,0.08);border-radius:10px;padding:8px;z-index:1000;
    min-width:180px;backdrop-filter:blur(10px);
  }
  body.light .settings-menu {background:rgba(255,255,255,0.95);}
  .settings-menu button {
    display:block;width:100%;text-align:left;background:transparent;border:0;color:var(--text);
    padding:6px 10px;border-radius:6px;cursor:pointer;font-size:14px;
  }
  .settings-menu button:hover {background:rgba(255,255,255,0.08);}
  .tabs {display:flex;gap:6px;overflow:auto;padding:4px;}
  .tab {padding:6px 10px;border-radius:8px;background:rgba(255,255,255,0.04);cursor:pointer;}
  .tab.active {background:rgba(59,130,246,0.2);color:#fff;}
  .iframe-wrap {flex:1;border-radius:10px;overflow:hidden;border:1px solid rgba(255,255,255,0.05);}
  iframe {width:100%;height:100%;border:0;}
  .bottom {display:flex;gap:10px;align-items:center;padding:8px;background:rgba(255,255,255,0.03);border-radius:10px;flex-wrap:wrap;}
  .list {display:flex;gap:8px;overflow:auto;}
  .chip {padding:6px 10px;border-radius:999px;background:rgba(255,255,255,0.05);font-size:13px;cursor:pointer;white-space:nowrap;}
  .chip:hover {background:rgba(255,255,255,0.08);}
  .small {font-size:12px;color:var(--muted);}
</style>
</head>
<body>
<div class="browser">

  <div class="title-bar">🌐 AI Web Browser</div>

  <div class="top">
    <div class="omnibox">
      <input id="address" placeholder="URL या खोज लिखें..."/>
      <button class="go" id="go">Go</button>
    </div>
    <button class="settings-btn" id="settingsBtn">⚙️</button>
    <div class="settings-menu" id="settingsMenu">
      <button id="back">⬅ Back</button>
      <button id="forward">➡ Forward</button>
      <button id="reload">⟳ Reload</button>
      <button id="home">🏠 Home</button>
      <button id="newtab">➕ New Tab</button>
      <button id="bookmark">★ Bookmark</button>
      <button id="theme">🌓 Theme</button>
    </div>
  </div>

  <div class="tabs" id="tabs"></div>
  <div class="iframe-wrap"><iframe id="viewer" sandbox="allow-same-origin allow-forms allow-scripts allow-popups"></iframe></div>

  <div class="bottom">
    <div>
      <strong class="small">🔖 Bookmarks:</strong>
      <div id="bookmarks" class="list"></div>
    </div>

    <!-- Shortcuts -->
    <div class="list" style="margin-left:20px;">
      <div class="chip" onclick="openExternal('https://www.google.com')">Google</div>
      <div class="chip" onclick="openExternal('https://www.google.com/chrome/')">Chrome</div>
      <div class="chip" onclick="openExternal('https://www.youtube.com')">YouTube</div>
    </div>

    <div style="margin-left:auto;">
      <strong class="small">🕓 History:</strong>
      <div id="history" class="list"></div>
    </div>
  </div>
</div>

<script>
const viewer=document.getElementById('viewer');
const addr=document.getElementById('address');
const tabsEl=document.getElementById('tabs');
const bookmarksEl=document.getElementById('bookmarks');
const historyEl=document.getElementById('history');
const settingsBtn=document.getElementById('settingsBtn');
const settingsMenu=document.getElementById('settingsMenu');
const themeBtn=document.getElementById('theme');
let tabs=[],active=null;

const HOME="https://duckduckgo.com";

// 🔹 open external sites (Google, YouTube, Chrome)
function openExternal(url){ window.open(url, '_blank'); }

function normalize(q){
  q=q.trim();if(!q)return HOME;
  if(!q.includes('.')&&!q.startsWith('http'))return 'https://duckduckgo.com/?q='+encodeURIComponent(q);
  if(/^[a-z]+:\/\//i.test(q))return q;
  return 'https://'+q;
}

function addTab(url){
  const id='t'+Date.now();
  tabs.push({id,url});
  active=id;
  renderTabs();load(url);
}

function renderTabs(){
  tabsEl.innerHTML='';
  tabs.forEach(t=>{
    const el=document.createElement('div');
    el.className='tab'+(t.id===active?' active':'');
    el.textContent=(t.url.length>25?t.url.slice(0,25)+'...':t.url);
    el.onclick=()=>{active=t.id;load(t.url);}
    tabsEl.appendChild(el);
  });
}

function load(q){
  const url=normalize(q);
  addr.value=q;
  viewer.src=url;
  saveHistory(url);
}

function saveHistory(url){
  let h=JSON.parse(localStorage.getItem('aiweb_hist')||'[]');
  h.unshift({url,time:new Date().toLocaleString()});
  h=h.slice(0,20);
  localStorage.setItem('aiweb_hist',JSON.stringify(h));
  renderHistory();
}

function renderHistory(){
  const h=JSON.parse(localStorage.getItem('aiweb_hist')||'[]');
  historyEl.innerHTML='';
  h.forEach(x=>{
    const c=document.createElement('div');
    c.className='chip';
    c.textContent=x.url.slice(0,15);
    c.onclick=()=>{addr.value=x.url;load(x.url);}
    historyEl.appendChild(c);
  });
}

function addBookmark(url){
  let b=JSON.parse(localStorage.getItem('aiweb_book')||'[]');
  if(!b.find(x=>x.url===url)){
    b.unshift({url});localStorage.setItem('aiweb_book',JSON.stringify(b));
  }
  renderBookmarks();
}

function renderBookmarks(){
  const b=JSON.parse(localStorage.getItem('aiweb_book')||'[]');
  bookmarksEl.innerHTML='';
  b.forEach(x=>{
    const c=document.createElement('div');
    c.className='chip';c.textContent=x.url.slice(0,15);
    c.onclick=()=>{addr.value=x.url;load(x.url);}
    bookmarksEl.appendChild(c);
  });
}

settingsBtn.onclick=()=>settingsMenu.style.display=settingsMenu.style.display==="block"?"none":"block";
document.addEventListener('click',e=>{
  if(!settingsMenu.contains(e.target)&&e.target!==settingsBtn)
    settingsMenu.style.display="none";
});

themeBtn.onclick=()=>{
  document.body.classList.toggle('light');
  localStorage.setItem('aiweb_theme',document.body.classList.contains('light')?'light':'dark');
};

document.getElementById('go').onclick=()=>load(addr.value);
document.getElementById('home').onclick=()=>load(HOME);
document.getElementById('newtab').onclick=()=>addTab(HOME);
document.getElementById('bookmark').onclick=()=>addBookmark(addr.value);
document.getElementById('back').onclick=()=>viewer.contentWindow.history.back();
document.getElementById('forward').onclick=()=>viewer.contentWindow.history.forward();
document.getElementById('reload').onclick=()=>viewer.src=viewer.src;
addr.addEventListener('keydown',e=>{if(e.key==='Enter')load(addr.value);});

addTab(HOME);
renderBookmarks();
renderHistory();
</script>
</body>
</html>

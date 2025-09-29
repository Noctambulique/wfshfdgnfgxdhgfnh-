<!--
Bell Madness - Browser clone (single-file)
Project: GitHub-ready single file (index.html)
Author: Generated for user

Instructions:
- Save this content as `index.html` at the root of your GitHub Pages repo.
- Optional: add an LICENSE (MIT) and a README.md describing the project and credits.
- All game data is stored in localStorage; no backend required.

Features included:
- Click/tap interactions to "prank" the neighbor (knock, ring, mailbox...) 
- Coins, hints, and unlockable doorbells
- Reactions system with combinatory triggers
- Simple, responsive UI and animations
- Save/load progress, reset button

Note: This is an original, small implementation inspired by Bell Madness mechanics. It is NOT a byte-for-byte copy of any existing game's source.
-->

<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Bell Madness — Clone léger</title>
  <style>
    :root{
      --bg:#0f1724; --card:#0b1220; --accent:#fbbf24; --muted:#98a8b9; --glass: rgba(255,255,255,0.03);
      --accent-2:#60a5fa; --success:#34d399;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,'Helvetica Neue',Arial;color:#e6eef6;background:linear-gradient(180deg,var(--bg),#07101a);}
    .wrap{max-width:1100px;margin:18px auto;padding:18px;display:grid;grid-template-columns:1fr 360px;gap:18px}
    header{grid-column:1/-1;display:flex;align-items:center;gap:12px}
    h1{font-size:20px;margin:0}
    .main-card{background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);border-radius:12px;padding:16px;box-shadow:0 6px 18px rgba(2,6,23,0.6)}
    .game-area{height:520px;display:flex;flex-direction:column;gap:12px}

    /* Neighbour house area */
    .house{flex:1;background:linear-gradient(180deg,#0b2236,#07202a);border-radius:10px;padding:16px;position:relative;overflow:hidden;display:flex}
    .porch{width:56%;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:8px;padding:8px}
    .door{width:180px;height:260px;border-radius:6px;background:linear-gradient(180deg,#6b3b1b,#3b1d0d);display:flex;align-items:center;justify-content:center;position:relative;box-shadow:inset 0 -8px 30px rgba(0,0,0,0.5)}
    .door .bell{position:absolute;right:14px;top:110px;width:46px;height:46px;border-radius:8px;background:var(--accent);display:flex;align-items:center;justify-content:center;font-weight:700}
    .window{width:44%;display:flex;align-items:center;justify-content:center}
    .reactions-log{position:absolute;left:12px;top:12px;background:var(--glass);backdrop-filter: blur(6px);padding:8px;border-radius:8px;max-width:38%;font-size:13px;color:var(--muted)}

    .controls{display:flex;flex-wrap:wrap;gap:10px}
    .btn{background:linear-gradient(180deg,#0f3a54,#0b2940);border:1px solid rgba(255,255,255,0.04);padding:10px 12px;border-radius:10px;color:#e6eef6;cursor:pointer;min-width:110px;text-align:center}
    .btn:active{transform:translateY(1px)}
    .btn.big{min-width:160px}

    /* Side panel */
    .side{height:100%;display:flex;flex-direction:column;gap:12px}
    .panel{background:var(--card);border-radius:10px;padding:12px}
    .stats{display:flex;gap:8px;align-items:center}
    .coin{display:flex;gap:6px;align-items:center;font-weight:700;color:var(--accent)}
    .inventory{display:flex;gap:8px;flex-wrap:wrap;margin-top:8px}
    .item{padding:6px 8px;border-radius:8px;background:var(--glass);font-size:13px}
    .progress{height:10px;border-radius:999px;background:rgba(255,255,255,0.05);overflow:hidden}
    .progress > i{display:block;height:100%;background:linear-gradient(90deg,var(--accent),var(--accent-2));width:0%}

    footer{grid-column:1/-1;margin-top:6px;color:var(--muted);font-size:13px}

    /* Reaction animations */
    .pop{animation:pop .8s ease}
    @keyframes pop{0%{transform:translateY(0) scale(1);opacity:0}30%{opacity:1;transform:translateY(-18px) scale(1.02)}100%{transform:translateY(-40px) scale(1);opacity:0}}

    /* responsive */
    @media (max-width:980px){.wrap{grid-template-columns:1fr;}.side{order:2}}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <img src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='44' height='44'><rect rx='9' width='100%' height='100%' fill='%230b1220'/><text x='50%' y='55%' font-size='18' text-anchor='middle' fill='%23fbbf24' font-family='Verdana'>BM</text></svg>" alt="logo" style="width:44px;height:44px;border-radius:9px;">
      <div>
        <h1>Bell Madness — Clone léger</h1>
        <div style="color:var(--muted);font-size:13px">But : découvrir des réactions en combinant des actions — sauvegarde automatique</div>
      </div>
    </header>

    <section class="main-card">
      <div class="game-area">
        <div class="house" id="house">
          <div class="reactions-log" id="log">Réactions découvertes : <strong id="foundCount">0</strong></div>
          <div class="porch">
            <div class="door" id="door">
              <div class="bell" id="visibleBell">🔔</div>
            </div>
            <div style="display:flex;gap:8px;margin-top:6px">
              <button class="btn" id="knockBtn">Frapper</button>
              <button class="btn" id="ringBtn">Sonner</button>
            </div>
          </div>
          <div class="window">
            <div style="width:90%;display:flex;flex-direction:column;align-items:center;gap:8px">
              <div style="background:var(--glass);padding:8px;border-radius:8px;width:100%;text-align:center">Boîte aux lettres • Robinet • Fenêtre</div>
              <div style="display:flex;gap:8px;margin-top:8px">
                <button class="btn" id="mailBtn">Boîte aux lettres</button>
                <button class="btn" id="waterBtn">Robinet</button>
                <button class="btn" id="tapWindowBtn">Taper</button>
              </div>
            </div>
          </div>
        </div>

        <div style="display:flex;justify-content:space-between;align-items:center">
          <div style="display:flex;gap:8px;align-items:center">
            <div class="btn" id="useHintBtn">Utiliser indice (<span id="hintsCount">0</span>)</div>
            <div class="btn" id="collectBtn">Collecter pièces</div>
          </div>
          <div style="display:flex;gap:8px;align-items:center">
            <div style="color:var(--muted);font-size:13px">Progrès :</div>
            <div class="progress" style="width:240px"><i id="progBar"></i></div>
          </div>
        </div>
      </div>
    </section>

    <aside class="side">
      <div class="panel">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div class="stats">
            <div class="coin">🪙 <span id="coins">0</span></div>
            <div style="width:10px"></div>
            <div style="color:var(--muted);font-size:13px">Indices : <strong id="hints">0</strong></div>
          </div>
          <div><button class="btn" id="resetBtn">Réinitialiser</button></div>
        </div>

        <div style="margin-top:10px;font-size:13px;color:var(--muted)">Sonnette(s) débloquées :</div>
        <div class="inventory" id="bellsList">
          <!-- doorbells -->
        </div>
      </div>

      <div class="panel">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <strong>Réactions connues</strong>
          <div style="color:var(--muted);font-size:13px">Découvertes : <span id="discoveries">0</span></div>
        </div>
        <div id="knownList" style="margin-top:8px;max-height:200px;overflow:auto;color:var(--muted);font-size:14px"></div>
      </div>

      <div class="panel">
        <strong>Aide / Guide rapide</strong>
        <ul style="margin-top:8px;color:var(--muted);font-size:13px">
          <li>Cliquer sur les actions pour provoquer des réactions.</li>
          <li>Collecter des pièces, acheter indices et nouvelles sonnettes.</li>
          <li>Combinaisons = plus de réactions.</li>
        </ul>
      </div>
    </aside>

    <footer>Version clone — usage éducatif / projet GitHub • Sauvegarde locale via localStorage</footer>
  </div>

  <script>
  // --- Game Data ---
  const ACTIONS = ['knock','ring','mail','water','tapWindow'];

  // Example doorbells / unlockables
  const DOORBELLS = [
    {id:'basic',name:'Sonnette basique',cost:0,unlocked:true,emoji:'🔔'},
    {id:'sneak',name:'Sonnette vicieuse',cost:120,unlocked:false,emoji:'🔕'},
    {id:'loud',name:'Sonnette forte',cost:300,unlocked:false,emoji:'📯'},
    {id:'myst',name:'Sonnette mystère',cost:700,unlocked:false,emoji:'🛎️'}
  ];

  // Reactions dataset. Each reaction has a triggers array (action sequences/subset), a reward coins, description.
  // Triggers may be single action or combos. This is a toy dataset; expand as needed.
  const REACTIONS = [
    {id:'r1',triggers:[['ring','ring']],desc:'Le voisin sort et grogne. Jette une tasse dorée.',coins:15,unlocked:false},
    {id:'r2',triggers:[['knock','mail']],desc:'Découvre une lettre d\'amour, s’énerve.',coins:25,unlocked:false},
    {id:'r3',triggers:[['water','tapWindow']],desc:'Le voisin glisse et la maison éternue (objet tombe).',coins:30,unlocked:false},
    {id:'r4',triggers:[['ring','ring','ring']],desc:'Le voisin devient furieux — devient Méduse (gag).',coins:120,unlocked:false},
    {id:'r5',triggers:[['knock','tapWindow','mail']],desc:'Grande scène : la maison se transforme.',coins:220,unlocked:false},
    {id:'r6',triggers:[['mail']],desc:'Le courrier contient un trésor caché.',coins:8,unlocked:false},
    {id:'r7',triggers:[['knock']],desc:'Toc toc, il regarde par la serrure.',coins:4,unlocked:false},
    {id:'r8',triggers:[['water']],desc:'Le robinet éclate — surprend le voisin.',coins:6,unlocked:false}
  ];

  // --- State ---
  let state = {
    coins: 0,
    hints: 0,
    unlockedBells: DOORBELLS.filter(d=>d.unlocked).map(d=>d.id),
    discoveries: {}, // reactionId: true
    progress: 0,
    lastActions: [],
    selectedBell: 'basic'
  };

  // Local storage keys
  const SAVE_KEY = 'bellmadness_clone_v1';

  // --- UI refs ---
  const coinsEl = document.getElementById('coins');
  const hintsEl = document.getElementById('hints');
  const hintsCountEl = document.getElementById('hintsCount');
  const foundCountEl = document.getElementById('foundCount');
  const progBar = document.getElementById('progBar');
  const knownList = document.getElementById('knownList');
  const bellsList = document.getElementById('bellsList');
  const discoveriesEl = document.getElementById('discoveries');

  // init
  function init(){
    load();
    renderBells();
    updateUI();
    attachHandlers();
    log('Bienvenue — amuse-toi à découvrir les réactions !');
  }

  function save(){localStorage.setItem(SAVE_KEY,JSON.stringify(state));}
  function load(){
    const raw = localStorage.getItem(SAVE_KEY);
    if(raw){
      try{const parsed = JSON.parse(raw); state = Object.assign(state, parsed);}catch(e){console.warn('load failed',e)}
    }
  }

  function attachHandlers(){
    document.getElementById('knockBtn').addEventListener('click',()=>action('knock'));
    document.getElementById('ringBtn').addEventListener('click',()=>action('ring'));
    document.getElementById('mailBtn').addEventListener('click',()=>action('mail'));
    document.getElementById('waterBtn').addEventListener('click',()=>action('water'));
    document.getElementById('tapWindowBtn').addEventListener('click',()=>action('tapWindow'));
    document.getElementById('collectBtn').addEventListener('click',collectCoins);
    document.getElementById('useHintBtn').addEventListener('click',useHint);
    document.getElementById('resetBtn').addEventListener('click',resetGame);
  }

  function action(act){
    // register last actions
    state.lastActions.push(act);
    if(state.lastActions.length>6) state.lastActions.shift();

    // visual feedback
    animateAction(act);

    // reward small coin chance for simple actions
    if(Math.random()<0.35){state.coins += Math.floor(1 + Math.random()*6);} // tiny coin drip

    // Check reactions
    checkReactions();

    // small progress increase
    state.progress = Math.min(100, state.progress + 0.6);
    updateUI();
    save();
  }

  function animateAction(act){
    const door = document.getElementById('door');
    const bubble = document.createElement('div');
    bubble.className='pop';
    bubble.style.position='absolute';
    bubble.style.right='20px';
    bubble.style.bottom='40px';
    bubble.style.padding='8px 10px';
    bubble.style.borderRadius='10px';
    bubble.style.background='rgba(255,255,255,0.06)';
    bubble.style.fontSize='13px';
    bubble.style.color='var(--muted)';
    bubble.textContent = {knock:'Toc',ring:'Ding',mail:'📫',water:'💧',tapWindow:'Tap'}[act] || act;
    door.appendChild(bubble);
    setTimeout(()=>bubble.remove(),900);
  }

  function checkReactions(){
    // test each reaction: if any triggers array is subset of lastActions (order-insensitive)
    REACTIONS.forEach(r=>{
      if(state.discoveries[r.id]) return; // already found
      for(const trig of r.triggers){
        if(isSubset(trig, state.lastActions)){
          discoverReaction(r);
          break;
        }
      }
    });
  }

  function isSubset(need, have){
    // check that 'have' contains all elements of 'need' in any order with at least the multiplicity
    const copy = [...have];
    for(const n of need){
      const idx = copy.indexOf(n);
      if(idx===-1) return false;
      copy.splice(idx,1);
    }
    return true;
  }

  function discoverReaction(r){
    state.discoveries[r.id]=true;
    state.coins += r.coins;
    // small chance to grant a hint
    if(Math.random()<0.2) state.hints++;
    log("Découverte : "+r.desc+" — +"+r.coins+" pièces");
    // increase progress more
    state.progress = Math.min(100, state.progress + Math.min(28, r.coins/4));
    updateUI();
    save();
    // unlock bells based on discoveries
    maybeUnlockBells();
  }

  function maybeUnlockBells(){
    const discoveredCount = Object.keys(state.discoveries).length;
    if(discoveredCount>=2) unlockBell('sneak');
    if(discoveredCount>=4) unlockBell('loud');
    if(discoveredCount>=6) unlockBell('myst');
  }

  function unlockBell(id){
    if(state.unlockedBells.includes(id)) return;
    const bell = DOORBELLS.find(d=>d.id===id);
    if(!bell) return;
    if(state.coins >= bell.cost){
      state.coins -= bell.cost;
      state.unlockedBells.push(id);
      log('Sonnette débloquée : '+bell.name);
      renderBells();
      updateUI();
      save();
    }
  }

  function renderBells(){
    bellsList.innerHTML='';
    for(const b of DOORBELLS){
      const el = document.createElement('div');
      el.className='item';
      const unlocked = state.unlockedBells.includes(b.id);
      el.innerHTML = `${b.emoji} <strong style='margin-left:6px'>${b.name}</strong><div style='font-size:12px;color:var(--muted)'>${unlocked? 'Débloquée' : 'Coûte '+b.cost+' pièces'}</div>`;
      if(!unlocked){
        const buy = document.createElement('button'); buy.className='btn'; buy.style.marginLeft='8px'; buy.style.padding='4px 6px'; buy.textContent='Acheter';
        buy.addEventListener('click',()=>{ if(state.coins>=b.cost){ state.coins -= b.cost; state.unlockedBells.push(b.id); renderBells(); updateUI(); save(); log('Acheté : '+b.name);} else{log('Pas assez de pièces pour '+b.name);} });
        el.appendChild(buy);
      } else {
        // select button
        const sel = document.createElement('button'); sel.className='btn'; sel.style.marginLeft='8px'; sel.style.padding='4px 6px'; sel.textContent = (state.selectedBell===b.id? 'Sélectionnée' : 'Sélectionner');
        sel.addEventListener('click',()=>{state.selectedBell=b.id; renderBells(); save(); updateUI();});
        el.appendChild(sel);
      }
      bellsList.appendChild(el);
    }
  }

  function collectCoins(){
    // small boost based on progress and discoveries
    const bonus = Math.floor(3 + state.progress*0.08 + Object.keys(state.discoveries).length*2 + Math.random()*8);
    state.coins += bonus;
    log('Collecte : +' + bonus + ' pièces');
    // slight progress decay to encourage activity
    state.progress = Math.max(0, state.progress - 6);
    updateUI(); save();
  }

  function useHint(){
    if(state.hints<=0){log('Aucun indice disponible');return}
    state.hints--;
    // reveal a random undiscovered reaction's partial hint
    const undiscovered = REACTIONS.filter(r=>!state.discoveries[r.id]);
    if(undiscovered.length===0){log('Tout est déjà découvert !');return}
    const pick = undiscovered[Math.floor(Math.random()*undiscovered.length)];
    // hint: show one action from its trigger
    const someTrigger = pick.triggers[0];
    const hintAction = someTrigger[Math.floor(Math.random()*someTrigger.length)];
    log('Indice : essaie de combiner « ' + hintAction + ' » avec d\'autres actions.');
    updateUI(); save();
  }

  function resetGame(){
    if(!confirm('Réinitialiser la progression ?')) return;
    localStorage.removeItem(SAVE_KEY);
    location.reload();
  }

  function updateUI(){
    coinsEl.textContent = state.coins;
    hintsEl.textContent = state.hints;
    hintsCountEl.textContent = state.hints;
    foundCountEl.textContent = Object.keys(state.discoveries).length;
    discoverries = Object.keys(state.discoveries).length;
    discoveriesEl.textContent = Object.keys(state.discoveries).length;
    // progress
    progBar.style.width = state.progress + '%';

    // known list
    knownList.innerHTML = '';
    for(const r of REACTIONS){
      if(state.discoveries[r.id]){
        const el = document.createElement('div'); el.style.padding='6px 0'; el.innerHTML = '• <strong>'+r.desc+'</strong> — +' + r.coins + ' pièces'; knownList.appendChild(el);
      }
    }
  }

  function log(msg){
    const logEl = document.getElementById('log');
    const time = new Date().toLocaleTimeString();
    logEl.innerHTML = time + ' — ' + msg + '<br/>' + logEl.innerHTML;
  }

  // auto-save every 5s
  setInterval(save,5000);

  // init
  init();
  </script>
</body>
</html>

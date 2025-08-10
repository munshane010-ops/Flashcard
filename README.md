# Flashcard
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>日本語 Flashcards — Study</title>
<style>
  :root{
    --bg:#0f1724; --card:#0b1220; --accent:#ff7a59; --muted:#9aa6b2; --glass: rgba(255,255,255,0.03);
    --radius:14px; font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  *{box-sizing:border-box}
  body{margin:0;background:linear-gradient(180deg,#071021 0%, #0f1724 100%);color:#e6eef6;min-height:100vh;display:flex;flex-direction:column;align-items:center;padding:28px;}
  header{width:100%;max-width:1100px;display:flex;gap:12px;align-items:center;justify-content:space-between;margin-bottom:20px}
  h1{font-size:20px;margin:0;display:flex;gap:10px;align-items:center}
  .brand-dot{width:12px;height:12px;background:var(--accent);border-radius:50%}
  main{width:100%;max-width:1100px;display:grid;grid-template-columns:360px 1fr;gap:20px;align-items:start}
  .panel{background:var(--card);border-radius:var(--radius);padding:16px;box-shadow:0 6px 20px rgba(2,6,23,0.6);min-height:320px}
  .sidebar{display:flex;flex-direction:column;gap:12px}
  .deck-list{display:flex;flex-direction:column;gap:8px;max-height:380px;overflow:auto;padding-right:6px}
  .deck{padding:10px;border-radius:10px;cursor:pointer;display:flex;justify-content:space-between;align-items:center;gap:8px;background:linear-gradient(180deg, rgba(255,255,255,0.01), transparent)}
  .deck.active{outline:2px solid rgba(255,255,255,0.03);box-shadow:inset 0 0 30px rgba(255,255,255,0.02)}
  button{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 10px;border-radius:10px;color:inherit;cursor:pointer}
  .small{font-size:13px;padding:6px 8px;border-radius:8px}
  .controls{display:flex;gap:8px;flex-wrap:wrap}
  .search{display:flex;gap:8px}
  input[type="text"], textarea, select{
    background:var(--glass);border:1px solid rgba(255,255,255,0.04);color:inherit;padding:8px;border-radius:8px;width:100%;
  }
  .card-area{display:flex;flex-direction:column;gap:14px}
  .flashcard{height:260px;border-radius:12px;display:flex;align-items:center;justify-content:center;position:relative;perspective:1200px;background:transparent}
  .card-inner{width:100%;height:100%;max-width:720px;transition:transform .7s;transform-style:preserve-3d;cursor:pointer}
  .card-inner.flipped{transform:rotateY(180deg)}
  .card-face{position:absolute;inset:0;background:linear-gradient(180deg, rgba(255,255,255,0.01), transparent);border-radius:12px;padding:24px;backface-visibility:hidden;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:12px}
  .front{font-size:34px;font-weight:600}
  .back{transform:rotateY(180deg);font-size:20px;opacity:0.95}
  .meta{display:flex;gap:10px;color:var(--muted);font-size:13px;}
  .actions{display:flex;gap:8px;align-items:center}
  .rating{display:flex;gap:8px}
  .badge{background:rgba(255,255,255,0.03);padding:6px 8px;border-radius:8px;font-size:13px}
  .list{background:linear-gradient(180deg, rgba(255,255,255,0.01), transparent);padding:10px;border-radius:10px;display:flex;flex-direction:column;gap:8px;max-height:300px;overflow:auto}
  footer{width:100%;max-width:1100px;margin-top:18px;color:var(--muted);font-size:13px;text-align:center}
  @media (max-width:900px){
    main{grid-template-columns:1fr; padding-bottom:20px}
    .flashcard{height:220px}
  }
  /* scrollbar */
  .deck-list::-webkit-scrollbar, .list::-webkit-scrollbar{width:8px;height:8px}
  .deck-list::-webkit-scrollbar-thumb, .list::-webkit-scrollbar-thumb{background:rgba(255,255,255,0.03);border-radius:8px}
  .tiny{font-size:12px;color:var(--muted)}
</style>
</head>
<body>
<header>
  <h1><span class="brand-dot"></span> 日本語 Flashcards</h1>
  <div style="display:flex;gap:8px;align-items:center">
    <div class="tiny">Keyboard: ← flip → next • F=Easy G=Good H=Hard</div>
    <button id="newDeckBtn" class="small">＋ New deck</button>
    <button id="importBtn" class="small">Import</button>
    <button id="exportBtn" class="small">Export</button>
  </div>
</header>

<main>
  <aside class="panel sidebar">
    <div>
      <label class="tiny">Search decks / cards</label>
      <div style="display:flex;gap:8px;margin-top:8px">
        <input id="searchInput" type="text" placeholder="Search deck or card (Japanese / English)" />
        <button id="searchBtn" class="small">Search</button>
      </div>
    </div>

    <div style="margin-top:10px;display:flex;gap:8px;align-items:center;justify-content:space-between">
      <strong>Decks</strong>
      <div class="controls">
        <button id="shuffleDecks" class="small">Shuffle</button>
        <button id="studyBtn" class="small">Start Study</button>
      </div>
    </div>

    <div class="deck-list" id="deckList" aria-live="polite"></div>

    <div style="margin-top:8px">
      <label class="tiny">Create card</label>
      <div style="display:flex;flex-direction:column;gap:8px;margin-top:8px">
        <select id="deckSelect"></select>
        <input id="japanese" type="text" placeholder="Japanese (front) — e.g. ありがとう" />
        <input id="reading" type="text" placeholder="Reading (optional) — e.g. ありがとう / arigatou" />
        <textarea id="meaning" placeholder="English meaning (back) — e.g. thank you" rows="2"></textarea>
        <input id="audio" type="text" placeholder="Audio URL (optional)" />
        <div style="display:flex;gap:8px">
          <button id="addCardBtn" class="small">Add card</button>
          <button id="clearForm" class="small">Clear</button>
        </div>
      </div>
    </div>
  </aside>

  <section class="panel card-area" aria-live="polite">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div>
        <div class="meta"><div id="currentDeckName">No deck selected</div><div id="deckStats" class="badge">0 cards</div></div>
      </div>
      <div class="actions">
        <button id="prevBtn" class="small">Prev</button>
        <button id="flipBtn" class="small">Flip</button>
        <button id="nextBtn" class="small">Next</button>
      </div>
    </div>

    <div class="flashcard" id="flashcard" tabindex="0" role="button" aria-pressed="false" title="Click or press space to flip">
      <div class="card-inner" id="cardInner">
        <div class="card-face front">
          <div id="frontText" class="front">Select a deck to start</div>
          <div id="readingText" class="tiny"></div>
        </div>
        <div class="card-face back">
          <div id="backText"></div>
          <div id="cardMeta" class="tiny" style="margin-top:8px"></div>
        </div>
      </div>
    </div>

    <div style="display:flex;gap:8px;align-items:center;justify-content:space-between">
      <div class="rating">
        <button id="easyBtn" class="small">Easy</button>
        <button id="goodBtn" class="small">Good</button>
        <button id="hardBtn" class="small">Hard</button>
      </div>
      <div class="tiny" id="sessionInfo">Ready</div>
    </div>

    <hr style="border:none;height:1px;background:rgba(255,255,255,0.03);margin:12px 0" />

    <div style="display:flex;gap:12px;align-items:flex-start;flex-wrap:wrap">
      <div style="flex:1;min-width:260px">
        <strong>Cards in deck</strong>
        <div class="list" id="cardsList"></div>
      </div>
      <div style="width:280px">
        <strong>Session stats</strong>
        <div style="margin-top:8px" id="sessionStats">
          <div class="tiny">Studied: <span id="studiedCount">0</span></div>
          <div class="tiny">Correct(Easy): <span id="easyCount">0</span></div>
          <div class="tiny">Good: <span id="goodCount">0</span></div>
          <div class="tiny">Hard: <span id="hardCount">0</span></div>
        </div>
      </div>
    </div>
  </section>
</main>

<footer>Made for learning Japanese • Data stored in your browser (localStorage)</footer>

<script>
/*
  Simple flashcards app
  Data model:
  {
    decks: {
      "<deckId>": {
         id, name, createdAt, cards: [{id, front, reading, back, audio, score, lastReviewed, intervalDays}]
      }
    },
    order: [deckId...]
  }
*/
const STORAGE_KEY = 'jp_flashcards_v1';
let store = loadStore();

// UI refs
const deckList = document.getElementById('deckList');
const deckSelect = document.getElementById('deckSelect');
const currentDeckName = document.getElementById('currentDeckName');
const deckStats = document.getElementById('deckStats');
const cardsList = document.getElementById('cardsList');
const frontText = document.getElementById('frontText');
const readingText = document.getElementById('readingText');
const backText = document.getElementById('backText');
const cardInner = document.getElementById('cardInner');
const flashcard = document.getElementById('flashcard');
const sessionInfo = document.getElementById('sessionInfo');

const studiedCountEl = document.getElementById('studiedCount');
const easyCountEl = document.getElementById('easyCount');
const goodCountEl = document.getElementById('goodCount');
const hardCountEl = document.getElementById('hardCount');
const studiedCount = { total:0, easy:0, good:0, hard:0 };

let activeDeckId = null;
let currentIndex = 0;
let sessionQueue = []; // indices of cards for study this session

// initial sample deck (if empty)
if(!store || !store.decks || Object.keys(store.decks).length===0){
  store = {
    decks: {},
    order: []
  };
  const sampleId = createDeckObj('Basic JLPT N5');
  store.decks[sampleId].cards.push(...[
    createCardObj('ありがとう','ありがとう','thank you',''),
    createCardObj('こんにちは','こんにちは','hello / good afternoon',''),
    createCardObj('水','みず','water',''),
    createCardObj('食べる','たべる','to eat','')
  ]);
  store.order.unshift(sampleId);
  saveStore();
}

renderDecks();
renderDeckSelect();
selectDeck(store.order[0]);

// ---------- UI wiring ----------
document.getElementById('newDeckBtn').addEventListener('click', ()=> {
  const name = prompt('New deck name','My Japanese Deck');
  if(name) {
    const id = createDeckObj(name);
    store.order.unshift(id);
    saveStore();
    renderDecks();
    renderDeckSelect();
    selectDeck(id);
  }
});
document.getElementById('addCardBtn').addEventListener('click', addCardFromForm);
document.getElementById('clearForm').addEventListener('click', clearForm);
document.getElementById('studyBtn').addEventListener('click', startStudy);
document.getElementById('shuffleDecks').addEventListener('click', ()=> {
  store.order = shuffleArray(store.order);
  saveStore();
  renderDecks();
});
document.getElementById('flipBtn').addEventListener('click', flipCard);
document.getElementById('prevBtn').addEventListener('click', ()=> navigate(-1));
document.getElementById('nextBtn').addEventListener('click', ()=> navigate(1));
document.getElementById('easyBtn').addEventListener('click', ()=> rateCard('easy'));
document.getElementById('goodBtn').addEventListener('click', ()=> rateCard('good'));
document.getElementById('hardBtn').addEventListener('click', ()=> rateCard('hard'));
document.getElementById('searchBtn').addEventListener('click', handleSearch);
document.getElementById('importBtn').addEventListener('click', importJSON);
document.getElementById('exportBtn').addEventListener('click', exportJSON);
document.getElementById('searchInput').addEventListener('keydown', (e)=>{
  if(e.key === 'Enter') handleSearch();
});

// keyboard controls
document.addEventListener('keydown', (e)=>{
  if(e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
  if(e.key === 'ArrowLeft'){ flipCard(); }
  if(e.key === 'ArrowRight'){ navigate(1); }
  if(e.key === 'f' || e.key === 'F') rateCard('easy');
  if(e.key === 'g' || e.key === 'G') rateCard('good');
  if(e.key === 'h' || e.key === 'H') rateCard('hard');
});

// flashcard click toggles
flashcard.addEventListener('click', flipCard);

// ---------- functions ----------
function loadStore(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY);
    return raw ? JSON.parse(raw) : null;
  }catch(e){ console.error(e); return null; }
}
function saveStore(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(store));
}

function createDeckObj(name){
  const id = 'd_' + Date.now() + '_' + Math.floor(Math.random()*9999);
  store.decks[id] = { id, name, createdAt: new Date().toISOString(), cards: [] };
  return id;
}
function createCardObj(front, reading='', back='', audio=''){
  return {
    id: 'c_' + Date.now() + '_' + Math.floor(Math.random()*9999),
    front: front || '',
    reading: reading || '',
    back: back || '',
    audio: audio || '',
    score: 0, // higher is better familiarity
    lastReviewed: null,
    intervalDays: 0 // for simple SRS
  };
}

function renderDecks(){
  deckList.innerHTML = '';
  store.order.forEach(id=>{
    const deck = store.decks[id];
    if(!deck) return;
    const div = document.createElement('div');
    div.className = 'deck' + (id === activeDeckId ? ' active' : '');
    div.setAttribute('data-id', id);
    div.innerHTML = `<div style="display:flex;flex-direction:column">
                        <strong style="font-size:14px">${escapeHtml(deck.name)}</strong>
                        <span class="tiny">${deck.cards.length} cards</span>
                     </div>
                     <div style="display:flex;flex-direction:column;align-items:flex-end;gap:6px">
                        <button class="tiny" data-action="rename">Rename</button>
                        <button class="tiny" data-action="delete">Delete</button>
                     </div>`;
    deckList.appendChild(div);

    div.addEventListener('click', (e)=>{
      const act = e.target.getAttribute('data-action');
      if(act === 'rename'){ const n = prompt('Rename deck', deck.name); if(n){ deck.name = n; saveStore(); renderDecks(); renderDeckSelect(); if(activeDeckId===id) updateDeckHeader(); } return; }
      if(act === 'delete'){ if(confirm('Delete deck "'+deck.name+'" ? This cannot be undone.')){ delete store.decks[id]; store.order = store.order.filter(x=>x!==id); saveStore(); if(activeDeckId===id){ activeDeckId = null; selectDeck(store.order[0]); } renderDecks(); renderDeckSelect(); } return; }
      selectDeck(id);
    });
  });
}

function renderDeckSelect(){
  deckSelect.innerHTML = '';
  const placeholder = document.createElement('option');
  placeholder.value = '';
  placeholder.textContent = '-- choose deck --';
  deckSelect.appendChild(placeholder);
  Object.values(store.decks).forEach(d=>{
    const opt = document.createElement('option'); opt.value = d.id; opt.textContent = d.name;
    deckSelect.appendChild(opt);
  });
  deckSelect.addEventListener('change', ()=> {
    const id = deckSelect.value;
    if(id) selectDeck(id);
  });
}

function selectDeck(id){
  if(!id || !store.decks[id]) {
    activeDeckId = null;
    currentDeckName.textContent = 'No deck selected';
    deckStats.textContent = '0 cards';
    cardsList.innerHTML = '';
    frontText.textContent = 'Select a deck to start';
    readingText.textContent = '';
    backText.textContent = '';
    return;
  }
  activeDeckId = id;
  currentIndex = 0;
  renderDecks();
  updateDeckHeader();
  renderCardsList();
  prepareSessionQueue();
  showCurrentCard();
}

function updateDeckHeader(){
  const deck = store.decks[activeDeckId];
  currentDeckName.textContent = deck.name;
  deckStats.textContent = `${deck.cards.length} cards`;
}

function renderCardsList(){
  cardsList.innerHTML = '';
  const deck = store.decks[activeDeckId];
  if(!deck) return;
  deck.cards.forEach((c, idx)=>{
    const el = document.createElement('div');
    el.style.display = 'flex';
    el.style.justifyContent = 'space-between';
    el.style.alignItems = 'center';
    el.innerHTML = `<div style="display:flex;flex-direction:column">
                      <div style="font-weight:600">${escapeHtml(c.front)} <span class="tiny"> ${escapeHtml(c.reading)}</span></div>
                      <div class="tiny">${escapeHtml(c.back)}</div>
                    </div>
                    <div style="display:flex;gap:6px;align-items:center">
                      <button class="tiny" data-action="play" title="Play audio">🔊</button>
                      <button class="tiny" data-action="edit">Edit</button>
                      <button class="tiny" data-action="del">Delete</button>
                    </div>`;
    cardsList.appendChild(el);

    el.querySelector('[data-action="play"]').addEventListener('click', (e)=> {
      e.stopPropagation();
      if(c.audio) {
        const au = new Audio(c.audio);
        au.play().catch(()=>alert('Audio failed to play. Maybe cross-origin or invalid URL.'));
      } else alert('No audio URL on this card.');
    });
    el.querySelector('[data-action="edit"]').addEventListener('click', (e)=> {
      e.stopPropagation();
      const nf = prompt('Japanese (front)', c.front);
      if(nf === null) return;
      const nr = prompt('Reading (optional)', c.reading);
      if(nr === null) return;
      const nb = prompt('Meaning (back)', c.back);
      if(nb === null) return;
      const na = prompt('Audio URL (optional)', c.audio);
      if(na === null) return;
      c.front=nf; c.reading=nr; c.back=nb; c.audio=na;
      saveStore();
      renderCardsList();
      showCurrentCard();
    });
    el.querySelector('[data-action="del"]').addEventListener('click', (e)=> {
      e.stopPropagation();
      if(confirm('Delete this card?')) {
        deck.cards.splice(idx,1);
        saveStore();
        renderCardsList();
        updateDeckHeader();
        prepareSessionQueue();
        showCurrentCard();
      }
    });
    el.addEventListener('click', ()=> {
      currentIndex = idx;
      showCurrentCard();
    });
  });
}

function addCardFromForm(){
  const deckId = document.getElementById('deckSelect').value || activeDeckId;
  if(!deckId){ alert('Select a deck first.'); return; }
  const front = document.getElementById('japanese').value.trim();
  const reading = document.getElementById('reading').value.trim();
  const back = document.getElementById('meaning').value.trim();
  const audio = document.getElementById('audio').value.trim();
  if(!front || !back){ alert('Please provide at least Japanese front and English meaning'); return; }
  const card = createCardObj(front, reading, back, audio);
  store.decks[deckId].cards.push(card);
  saveStore();
  renderCardsList();
  updateDeckHeader();
  clearForm();
  prepareSessionQueue();
  alert('Card added!');
}

function clearForm(){
  document.getElementById('japanese').value = '';
  document.getElementById('reading').value = '';
  document.getElementById('meaning').value = '';
  document.getElementById('audio').value = '';
}

function showCurrentCard(){
  const deck = store.decks[activeDeckId];
  if(!deck || deck.cards.length===0){
    frontText.textContent = 'No cards in deck';
    readingText.textContent = '';
    backText.textContent = '';
    sessionInfo.textContent = 'No cards';
    return;
  }
  // clamp index
  if(currentIndex < 0) currentIndex = 0;
  if(currentIndex >= deck.cards.length) currentIndex = deck.cards.length - 1;
  const c = deck.cards[currentIndex];
  frontText.textContent = c.front || '(no front)';
  readingText.textContent = c.reading || '';
  backText.textContent = c.back || '';
  cardInner.classList.remove('flipped');
  updateSessionInfo();
}

function flipCard(){
  cardInner.classList.toggle('flipped');
}

function navigate(dir){
  const deck = store.decks[activeDeckId];
  if(!deck) return;
  currentIndex += dir;
  if(currentIndex < 0) currentIndex = 0;
  if(currentIndex >= deck.cards.length) currentIndex = deck.cards.length - 1;
  showCurrentCard();
}

function startStudy(){
  if(!activeDeckId) { alert('Select a deck first'); return; }
  prepareSessionQueue();
  if(sessionQueue.length === 0){ alert('No cards scheduled for review right now.'); return; }
  currentIndex = sessionQueue[0];
  resetSessionStats();
  showCurrentCard();
  sessionInfo.textContent = `Study started — ${sessionQueue.length} cards`;
}

function prepareSessionQueue(){
  sessionQueue = [];
  studiedCount.total = 0;
  Object.assign(studiedCount, {easy:0,good:0,hard:0});
  const deck = store.decks[activeDeckId];
  if(!deck) return;
  // simple scheduling: pick all cards, but prioritize low-score and long-unseen
  const cardsWithPriority = deck.cards.map((c, idx)=>{
    const ageDays = c.lastReviewed ? Math.max(1, Math.round((Date.now() - new Date(c.lastReviewed))/86400000)) : 365;
    const priority = (5 - (c.score || 0)) * 2 + Math.min(30, ageDays)/30;
    return { idx, priority };
  });
  cardsWithPriority.sort((a,b)=>b.priority-a.priority);
  sessionQueue = cardsWithPriority.map(x=>x.idx);
  // limit to first 100 to avoid huge sessions
  if(sessionQueue.length>100) sessionQueue = sessionQueue.slice(0,100);
  // If a specific currentIndex exists and is in deck, keep it at front
  if(deck.cards.length>0 && currentIndex >=0 && currentIndex < deck.cards.length){
    if(!sessionQueue.includes(currentIndex)) sessionQueue.unshift(currentIndex);
  }
}

function rateCard(level){
  if(!activeDeckId) return;
  const deck = store.decks[activeDeckId];
  if(!deck || deck.cards.length===0) return;
  const c = deck.cards[currentIndex];
  if(!c) return;
  // adjust score and interval using simple rules
  const now = new Date().toISOString();
  if(level === 'easy'){ c.score = Math.min(5, (c.score || 0) + 2); c.intervalDays = Math.max(1, (c.intervalDays || 0) * 2 + 3); studiedCount.easy++; }
  if(level === 'good'){ c.score = Math.min(5, (c.score || 0) + 1); c.intervalDays = Math.max(1, (c.intervalDays || 0) + 2); studiedCount.good++; }
  if(level === 'hard'){ c.score = Math.max(0, (c.score || 0) - 1); c.intervalDays = 1; studiedCount.hard++; }
  c.lastReviewed = now;
  saveStore();
  // update session counts
  studiedCount.total++;
  updateSessionStats();
  // advance to next card in sessionQueue if present
  const pos = sessionQueue.indexOf(currentIndex);
  if(pos >=0) sessionQueue.splice(pos,1);
  if(sessionQueue.length>0){
    currentIndex = sessionQueue[0];
    showCurrentCard();
  } else {
    sessionInfo.textContent = 'Session complete';
    alert('Session complete — good job!');
  }
}

function updateSessionStats(){
  studiedCountEl.textContent = studiedCount.total;
  easyCountEl.textContent = studiedCount.easy;
  goodCountEl.textContent = studiedCount.good;
  hardCountEl.textContent = studiedCount.hard;
  const deck = store.decks[activeDeckId];
  const pos = deck ? (deck.cards.length ? (currentIndex + 1) : 0) : 0;
  sessionInfo.textContent = `Card ${pos} of ${deck ? deck.cards.length : 0}`;
  // card meta on back
  const c = deck && deck.cards[currentIndex];
  if(c){
    document.getElementById('cardMeta').textContent = `Score: ${c.score || 0} • Interval: ${c.intervalDays || 0}d • Last: ${c.lastReviewed ? new Date(c.lastReviewed).toLocaleDateString() : 'never'}`;
  }
}

function resetSessionStats(){ studiedCount.total=0; studiedCount.easy=0; studiedCount.good=0; studiedCount.hard=0; updateSessionStats(); }

function handleSearch(){
  const q = document.getElementById('searchInput').value.trim().toLowerCase();
  if(!q){ renderDecks(); return; }
  // search decks by name or cards by front/back
  const matches = [];
  Object.values(store.decks).forEach(d=>{
    const nameMatch = d.name.toLowerCase().includes(q);
    const cardMatch = d.cards.some(c => (c.front + ' ' + c.back + ' ' + c.reading).toLowerCase().includes(q));
    if(nameMatch || cardMatch) matches.push(d);
  });
  if(matches.length===0){ alert('No matches'); return; }
  // show only matched decks (re-render deckList)
  deckList.innerHTML = '';
  matches.forEach(d=>{
    const el = document.createElement('div');
    el.className = 'deck';
    el.innerHTML = `<div><strong>${escapeHtml(d.name)}</strong><div class="tiny">${d.cards.length} cards</div></div>`;
    deckList.appendChild(el);
    el.addEventListener('click', ()=> selectDeck(d.id));
  });
}

function exportJSON(){
  const payload = JSON.stringify(store, null, 2);
  const blob = new Blob([payload], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = 'jp_flashcards_export.json'; a.click();
  URL.revokeObjectURL(url);
}

function importJSON(){
  const input = document.createElement('input');
  input.type = 'file'; input.accept = 'application/json';
  input.addEventListener('change', (e)=>{
    const f = e.target.files[0];
    if(!f) return;
    const reader = new FileReader();
    reader.onload = ()=> {
      try{
        const data = JSON.parse(reader.result);
        if(!data || !data.decks){ if(confirm('This JSON does not look like a deck export. Replace current data with this file anyway?')){ store = data; saveStore(); renderDecks(); renderDeckSelect(); selectDeck(store.order[0]); } return; }
        // otherwise merge: add decks not present
        let added = 0;
        Object.values(data.decks).forEach(d=>{
          if(!store.decks[d.id]){
            store.decks[d.id] = d; store.order.unshift(d.id); added++;
          } else {
            // if id exists, create new id to avoid collisions
            const newid = createDeckObj(d.name + ' (import)');
            store.decks[newid].cards = d.cards || [];
            added++;
          }
        });
        saveStore();
        renderDecks(); renderDeckSelect(); alert('Import complete. Decks added: ' + added);
      }catch(err){
        alert('Failed to import: ' + err.message);
      }
    };
    reader.readAsText(f);
  });
  input.click();
}

// small helpers
function escapeHtml(s){ if(!s) return ''; return s.replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;'); }
function shuffleArray(a){ for(let i=a.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [a[i],a[j]]=[a[j],a[i]]; } return a; }
function createSampleDeck(){ const id = createDeckObj('Sample'); store.decks[id].cards.push(createCardObj('例','れい','example','')); return id; }

function exportStoreToConsole(){
  console.log('STORE:', store);
}

// expose some for debugging
window._store = store;
window._exportConsole = exportStoreToConsole;
</script>

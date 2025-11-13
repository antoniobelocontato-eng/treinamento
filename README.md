<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Dobble — v35.3 (match amarelo + erro azul + pódio com pontos)</title>
<style>
  :root { --bg:#b91c1c; --panel:#dc2626; --ink:#fff; --ring:#facc15; --card: clamp(260px, 44vw, 420px); }
  *{box-sizing:border-box} html,body{height:100%;margin:0}
  body{background:var(--bg);color:var(--ink);font-family:ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,Arial}
  .wrap{max-width:1200px;margin:0 auto;padding:12px}
  .topbar{display:flex;align-items:start;gap:8px;justify-content:space-between;margin-bottom:8px;flex-wrap:wrap}
  .toplines{display:flex;flex-direction:column;gap:6px}
  .topline{font-weight:800}
  .toplist{white-space:pre-line;line-height:1.45}
  .gear, .btn{appearance:none;border:0;background:#fff;color:#b91c1c;border-radius:10px;padding:8px 12px;font-weight:800;cursor:pointer}
  .gear{padding:6px 10px}
  .frame{background:var(--panel);border:1px solid #ffffff40;border-radius:16px;box-shadow:0 6px 18px #00000040}
  .head{display:flex;align-items:center;justify-content:space-between;padding:8px 10px;border-bottom:1px solid #ffffff40}
  .title{font-weight:800;font-size:16px}.meta{opacity:.9;font-size:13px}
  .grid2{display:grid;grid-template-columns:1fr 1fr;gap:12px}
  .body{padding:10px}
  .cardwrap{position:relative;width:var(--card);margin-inline:auto}
  .cardwrap::before{content:"";display:block;padding-top:100%}
  .cardboard{
    position:absolute; inset:0;
    background:#ffffff14; border:2px solid #ffffff59;
    border-radius:18px;
    overflow:visible;  /* não corta o glow */
  }

  .token{position:absolute;border-radius:50%;background:#fff;border:2px solid #ffffffd9;box-shadow:0 3px 8px #00000038;display:grid;place-items:center;cursor:pointer;transition:.08s; outline: none;}
  .token:active{transform:scale(.985)}
  .token img{width:98%;height:98%;object-fit:contain;border-radius:50%}

  /* MATCH amarelo brilhante (nos dois lados) */
  .is-match{
    border-color: var(--ring) !important;
    box-shadow:
      0 0 0 8px rgba(250,204,21,.35),
      0 0 24px 12px rgba(250,204,21,.45),
      0 0 60px 18px rgba(250,204,21,.33);
    position: relative;
    filter: drop-shadow(0 0 8px rgba(250,204,21,.9));
  }
  .is-match::after{
    content:""; position:absolute; inset:-6px;
    border:4px solid var(--ring); border-radius:50%;
    box-shadow:0 0 18px rgba(250,204,21,.75); pointer-events:none;
  }

  /* ERRO: primeiro clique errado vira azul e bloqueia nova tentativa */
  .is-wrong{
    border-color:#60a5fa !important;
    position:relative;
    box-shadow:0 0 0 6px rgba(96,165,250,.25), 0 0 20px rgba(96,165,250,.45);
  }
  .is-wrong::after{
    content:""; position:absolute; inset:-6px;
    border:4px solid #60a5fa; border-radius:50%;
    box-shadow:0 0 14px rgba(96,165,250,.7); pointer-events:none;
  }

  .controls{margin-top:12px;display:flex;gap:8px;align-items:center;justify-content:space-between;flex-wrap:wrap}
  .btn[disabled]{opacity:.5;cursor:not-allowed}

  .roundbox{margin-top:10px;background:#ffffff14;border:1px solid #ffffff40;border-radius:12px;padding:8px}
  .rankrow{display:flex;gap:8px;align-items:center;flex-wrap:wrap;max-height:28vh;overflow-y:auto;padding-right:4px}
  .rankpill{display:flex;align-items:center;gap:6px;background:#ffffff1a;border:1px solid #ffffff40;border-radius:999px;padding:4px 8px;white-space:nowrap;font-size:13px}

  .overlay{position:fixed;inset:0;background:#0009;display:none;align-items:center;justify-content:center;z-index:50}
  .modal{width:min(920px,92vw);background:#fff;color:#222;border-radius:14px;padding:14px}
  .list{display:flex;flex-direction:column;gap:6px;margin-top:10px;max-height:60vh;overflow-y:auto;padding-right:6px}
  .li{display:flex;align-items:center;gap:8px;background:#f7f7f7;border:1px solid #e5e7eb;border-radius:8px;padding:6px 10px}
  .li.missed{background:#fff1f2;border-color:#fecdd3}
  .idx{font-weight:800;min-width:140px;text-align:left}.name{font-weight:700}.tag{font-size:12px;opacity:.75}
  .hint{font-size:12px;opacity:.8;margin-top:6px}

  /* PÓDIO */
  .podium{display:flex;align-items:flex-end;gap:12px;height:240px;margin:12px 0}
  .step{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:flex-end}
  .pod-name{
    font-weight:900;
    margin-bottom:6px;
    text-align:center;
    font-size:22px;
    line-height:1.15;
    text-shadow:0 2px 4px rgba(0,0,0,.25);
  }
  .pod-name.first{
    color:#b45309;
    background:#fef08a;
    padding:4px 10px;
    border-radius:999px;
    border:2px solid #f59e0b;
  }
  .block{width:100%;border-radius:12px 12px 0 0;position:relative;display:grid;place-items:center}
  .first .block{height:200px;background:#fde68a;border:1px solid #f59e0b}
  .second .block{height:160px;background:#e5e7eb;border:1px solid #9ca3af}
  .third .block{height:140px;background:#fed7aa;border:1px solid #fb923c}
  .trophy{width:72px;height:72px}
  .pod-list{font-size:14px;line-height:1.5;margin-top:8px;white-space:pre-line}

  /* Contagem como botão no topo */
  #countBadge{
    display:none;background:#111;color:#fff;border:1px solid #ffffff30;
    padding:10px 16px;font-size:18px;border-radius:12px;line-height:1;
  }

  @media (max-width: 860px){.grid2{grid-template-columns:1fr} :root{--card:min(92vw,68vh)}}
  @media (max-width: 420px){:root{--card:min(94vw,62vh)}}
</style>
</head>
<body>
<div class="wrap">
  <div class="topbar">
    <div class="toplines">
      <div class="topline">Ranking geral acumulado :</div>
      <div class="toplist" id="topListText">—</div>
    </div>
    <div style="display:flex; gap:8px; align-items:center; flex-wrap:wrap">
      <button class="btn" id="countBadge" disabled>⏳ 30s</button>
      <button class="btn" id="playSoundBtn" style="display:none">▶️ Som</button>
      <button class="gear" id="embedBtn" title="Embutir/atualizar figuras (gera novo HTML)">⚙️ Figuras</button>
      <audio id="roundAudio" preload="auto" style="display:none"></audio>
    </div>
  </div>

  <div class="grid2">
    <div class="frame">
      <div class="head"><div class="title">Carta da Mesa</div><div class="meta" id="roomInfo"></div></div>
      <div class="body"><div class="cardwrap"><div id="center" class="cardboard"></div></div></div>
    </div>
    <div class="frame">
      <div class="head"><div class="title" id="playerHeader">Jogador</div><div class="meta" id="roleInfo"></div></div>
      <div class="body"><div class="cardwrap"><div id="mine" class="cardboard"></div></div></div>
    </div>
  </div>

  <div class="controls">
    <div class="meta">Rodada: <span id="rnum">1</span> / 21</div>
    <div class="row" style="display:flex; gap:8px; flex-wrap:wrap; align-items:center;">
      <button class="btn" id="resetScores" title="Host">Reset ranking</button>
      <button class="btn" id="showPodium" title="Host">Ranking geral acumulado</button>
      <button class="btn" id="next" title="Host">Avançar</button>
      <label class="btn" for="roundAudioFile" title="Carregar música própria da rodada (opcional)">🎵 Música da rodada</label>
      <input id="roundAudioFile" type="file" accept="audio/*" style="display:none">
    </div>
  </div>

  <div class="roundbox">
    <div class="meta" style="margin-bottom:6px;">Ranking da rodada (sala):</div>
    <div class="rankrow" id="roundRank"><span style="opacity:.7;">Aguardando acerto…</span></div>
  </div>
</div>

<!-- Lobby -->
<div class="overlay" id="lobbyOverlay" style="display:flex;">
  <div class="modal">
    <h3 style="margin:0 0 10px 0;">Dobble — Entrar na Sala</h3>
    <div class="field">
      <label>Seu nome</label><br>
      <input id="playerName" type="text" placeholder="Ex.: Antonio" style="width:100%; padding:8px 10px; border:1px solid #e5e7eb; border-radius:8px;">
    </div>
    <div class="field" style="margin-top:10px;">
      <label>Para ser um host, digite a senha (CLARO) e crie uma sala com código:</label><br>
      <input id="hostPwd" type="password" placeholder="Senha do host (opcional)" style="width:100%; padding:8px 10px; border:1px solid #e5e7eb; border-radius:8px;">
      <div style="margin-top:8px; display:flex; gap:8px; align-items:center;">
        <button class="btn" id="genRoomBtn" style="background:#0ea5e9;color:#fff;">Gerar sala com código</button>
      </div>
    </div>
    <hr style="margin:12px 0;">
    <div class="field">
      <label>Entrar pelo código da sala:</label><br>
      <div style="display:flex; gap:8px; align-items:center; flex-wrap:wrap;">
        <input id="joinCode" placeholder="Ex.: H7Q2M" style="padding:8px;border:1px solid #e5e7eb;border-radius:8px;">
        <button class="btn" id="joinCodeBtn" style="background:#16a34a;color:#fff;">Entrar pelo código</button>
        <span class="hint">Ao digitar (5+), você entra automaticamente.</span>
      </div>
    </div>
  </div>
</div>

<!-- Resultados da rodada -->
<div class="overlay" id="resultsOverlay" style="display:none;">
  <div class="modal">
    <h3 style="margin:0 0 10px 0;">Ranking da rodada (sala)</h3>
    <div class="meta" id="resultsRoundLabel">Rodada —</div>
    <div class="list" id="resultsList"></div>
    <div style="display:flex; justify-content:flex-end; gap:8px; margin-top:12px;">
      <button class="btn" id="closeResultsBtn">Fechar</button>
      <button class="btn" id="continueBtn">Continuar</button>
    </div>
  </div>
</div>

<!-- Pódio -->
<div class="overlay" id="podiumOverlay" style="display:none;">
  <div class="modal">
    <h3 style="margin:0 0 10px 0;">Ranking geral acumulado — Pódio</h3>
    <div class="podium">
      <div class="step second"><div class="pod-name" id="pod2name">—</div><div class="block"><svg class="trophy" viewBox="0 0 64 64"><path fill="#9ca3af" d="M16 8h32v8h6a10 10 0 0 1-10 10H40a8 8 0 0 1-8 8 8 8 0 0 1-8-8H20A10 10 0 0 1 10 16h6z"/><rect x="24" y="42" width="16" height="6" fill="#9ca3af"/><rect x="20" y="48" width="24" height="6" fill="#9ca3af"/></svg></div></div>
      <div class="step first"><div class="pod-name first" id="pod1name">—</div><div class="block"><svg class="trophy" viewBox="0 0 64 64"><path fill="#f59e0b" d="M16 8h32v8h6a10 10 0 0 1-10 10H40a8 8 0 0 1-8 8 8 8 0 0 1-8-8H20A10 10 0 0 1 10 16h6z"/><rect x="24" y="42" width="16" height="6" fill="#f59e0b"/><rect x="20" y="48" width="24" height="6" fill="#f59e0b"/></svg></div></div>
      <div class="step third"><div class="pod-name" id="pod3name">—</div><div class="block"><svg class="trophy" viewBox="0 0 64 64"><path fill="#fb923c" d="M16 8h32v8h6a10 10 0 0 1-10 10H40a8 8 0 0 1-8 8 8 8 0 0 1-8-8H20A10 10 0 0 1 10 16h6z"/><rect x="24" y="42" width="16" height="6" fill="#fb923c"/><rect x="20" y="48" width="24" height="6" fill="#fb923c"/></svg></div></div>
    </div>
    <div class="pod-list" id="podListText"></div>
    <div style="display:flex; gap:8px; justify-content:flex-end; align-items:center; margin-top:12px;">
      <button class="btn" id="closePodiumBtn">Fechar</button>
    </div>
  </div>
</div>

<!-- Resultado Final -->
<div class="overlay" id="finalOverlay" style="display:none;">
  <div class="modal">
    <h3 style="margin:0 0 10px 0;">Resultado Final — Ranking geral acumulado</h3>
    <div id="finalCongrats" style="font-weight:800; font-size:18px; margin-bottom:8px;"></div>
    <div class="list" id="finalList"></div>
    <div style="display:flex; justify-content:flex-end; gap:8px; margin-top:12px;">
      <button class="btn" id="closeFinalBtn">Fechar</button>
    </div>
  </div>
</div>

<!-- Embutir Figuras -->
<div class="overlay" id="embedOverlay" style="display:none;">
  <div class="modal">
    <h3 style="margin:0 0 10px 0;">Embutir figuras (gera novo HTML com base64)</h3>
    <p class="hint">Selecione <b>21 imagens</b> (PNG/JPG) na ordem dos slides 1…21.</p>
    <input id="files" type="file" accept=".png,.jpg,.jpeg,.PNG,.JPG" multiple />
    <div id="embedStatus" class="hint" style="margin-top:6px;"></div>
    <div style="display:flex; gap:8px; justify-content:flex-end; margin-top:12px;">
      <button class="btn" id="cancelEmbed">Cancelar</button>
      <button class="btn" id="genEmbedHtml" style="background:#16a34a;color:#fff;">Gerar HTML com imagens embutidas</button>
    </div>
  </div>
</div>

<script src="https://www.gstatic.com/firebasejs/10.12.3/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.3/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.3/firebase-database-compat.js"></script>
<script>
/* ===== Firebase ===== */
const firebaseConfig = {
  apiKey: "AIzaSyB8fPk7QW9ARm_3B12w6J0g-3_kUJkxkq4",
  authDomain: "somos-tc.firebaseapp.com",
  databaseURL: "https://somos-tc-default-rtdb.firebaseio.com",
  projectId: "somos-tc",
  storageBucket: "somos-tc.firebasestorage.app",
  messagingSenderId: "561006578743",
  appId: "1:561006578743:web:9b93eaea9942d4b90383b2",
  measurementId: "G-ZSQNWW7NEZ"
};
firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db = firebase.database();

let serverOffset = 0;
db.ref(".info/serverTimeOffset").on("value", s => { serverOffset = s.val() || 0; });
const nowServer = () => Date.now() + serverOffset;

/* ===== Áudio (arquivo + synth fallback) ===== */
const AC = window.AudioContext || window.webkitAudioContext;
let audioCtx=null, masterGain=null;
function ensureAudio(){
  if(!AC) return;
  if(!audioCtx){
    audioCtx=new AC();
    masterGain=audioCtx.createGain();
    masterGain.gain.value=0.9;
    masterGain.connect(audioCtx.destination);
  }
  if(audioCtx.state==='suspended') audioCtx.resume();
}
['pointerdown','keydown','touchstart'].forEach(ev=>window.addEventListener(ev, ()=>{ ensureAudio(); }, {passive:true}));

let synth = { running:false, kickInt:null, hatInt:null, bassOsc:null, bassGain:null };
function startSynth(){
  if(!audioCtx) return;
  stopSynth();
  synth.running=true;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.type='sawtooth'; osc.frequency.value=110; gain.gain.value=0.05;
  osc.connect(gain).connect(masterGain); osc.start();
  let step=0;
  synth.kickInt = setInterval(()=>{
    const t=audioCtx.currentTime;
    const g=audioCtx.createGain(); const o=audioCtx.createOscillator();
    o.type='sine'; o.frequency.setValueAtTime(80,t);
    g.gain.setValueAtTime(0.6,t); g.gain.exponentialRampToValueAtTime(0.001,t+0.2);
    o.connect(g).connect(masterGain); o.start(t); o.stop(t+0.21);
    const f=[110,146.8,98,123.5][step%4]; osc.frequency.setValueAtTime(f,t); step++;
  }, 500);
  synth.hatInt = setInterval(()=>{
    const t=audioCtx.currentTime;
    const g=audioCtx.createGain(); const o=audioCtx.createOscillator();
    o.type='triangle'; o.frequency.setValueAtTime(6000,t);
    g.gain.setValueAtTime(0.08,t); g.gain.exponentialRampToValueAtTime(0.001,t+0.05);
    o.connect(g).connect(masterGain); o.start(t); o.stop(t+0.051);
  }, 250);
  synth.bassOsc=osc; synth.bassGain=gain;
}
function stopSynth(){
  if(synth.kickInt){ clearInterval(synth.kickInt); synth.kickInt=null; }
  if(synth.hatInt){ clearInterval(synth.hatInt); synth.hatInt=null; }
  try{ if(synth.bassOsc) synth.bassOsc.stop(); }catch(_){}
  try{ if(synth.bassGain) synth.bassGain.disconnect(); }catch(_){}
  try{ if(synth.bassOsc) synth.bassOsc.disconnect(); }catch(_){}
  synth.running=false; synth.bassOsc=null; synth.bassGain=null;
}

/* ===== Baralho (q=4 ⇒ 21 símbolos, 5 por carta) ===== */
function svgPlaceholder(n){
  const svg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
    <rect width="100%" height="100%" fill="white"/>
    <circle cx="256" cy="256" r="230" fill="#ef4444"/>
    <text x="50%" y="54%" text-anchor="middle" font-size="220" font-family="Segoe UI, Arial, sans-serif" fill="white" font-weight="700">${n}</text>
  </svg>`;
  return "data:image/svg+xml;base64,"+btoa(unescape(encodeURIComponent(svg)));
}

/* >>> AQUI troquei os números por imagens (arquivos locais) <<< */
const SYMBOL_EMBED = [
  "slide01.png","slide02.png","slide03.png","slide04.png","slide05.png",
  "slide06.png","slide07.png","slide08.png","slide09.png","slide10.png",
  "slide11.png","slide12.png","slide13.png","slide14.png","slide15.png",
  "slide16.png","slide17.png","slide18.png","slide19.png","slide20.png",
  "slide21.png"
];

const q=4;
function gf4mul(a,b){return [[0,0,0,0],[0,1,2,3],[0,2,3,1],[0,3,1,2]][a][b];}
function gf4add(a,b){return a^b;}
function pid(x,y){return x*q+y;}
function infm(m){return q*q + m;}
const INF=q*q+q;
const symbols = Array.from({length:q*q+q+1},(_,i)=>({id:i,label:'Slide'+(i+1),img:SYMBOL_EMBED[i]||''}));
const cards=[];
for(let m=0;m<q;m++){
  for(let b=0;b<q;b++){
    cards.push([pid(0,gf4add(gf4mul(m,0),b)), pid(1,gf4add(gf4mul(m,1),b)), pid(2,gf4add(gf4mul(m,2),b)), pid(3,gf4add(gf4mul(m,3),b)), infm(m)]);
  }
}
for(let b=0;b<q;b++) cards.push([pid(b,0), pid(b,1), pid(b,2), pid(b,3), INF]);
cards.push([INF, infm(0), infm(1), infm(2), infm(3)]);
const symToCards = symbols.reduce((acc,s)=> (acc[s.id]=[], acc), {});
cards.forEach((c,ci)=>c.forEach(sid=>symToCards[sid].push(ci)));
const rounds=[]; let last=null;
for(let i=0;i<symbols.length;i++){
  const sym=i; const cl=symToCards[sym];
  let a=cl[i%cl.length], b=cl[(i+1)%cl.length];
  if(a===b) b=cl[(i+2)%cl.length];
  let pair=[a,b].sort().join('-');
  if(last && pair===last){ b=cl[(i+3)%cl.length]; pair=[a,b].sort().join('-'); }
  rounds.push({center:a, mine:b, match:sym}); last=pair;
}
const DECK = {order:q, symbols, cards, rounds};

/* ===== Estado ===== */
let uid=null, me=null, isHost=false, room=null, roomRef=null;
let playersCache={}, scoresCache={}, roundIdx=0, clickedInRound=false, hostUid=null;
let isRoundLocked=false;

const qs=(s)=>document.querySelector(s);
function clear(el){ while(el.firstChild) el.removeChild(el.firstChild); }
async function ensureAuth(){ if(auth.currentUser && auth.currentUser.uid){ uid=auth.currentUser.uid; return uid; } await auth.signInAnonymously(); return await new Promise(res=>auth.onAuthStateChanged(u=>{ if(u){uid=u.uid;res(uid);} })); }
function genCode(){ const s='ABCDEFGHJKMNPQRSTUVWXYZ23456789'; let out=''; for(let i=0;i<5;i++) out+=s[Math.floor(Math.random()*s.length)]; return out; }

/* ===== Lobby ===== */
document.getElementById('genRoomBtn').addEventListener('click', async ()=>{
  const pwd=(qs('#hostPwd').value||'').trim();
  if(pwd!=='CLARO') return alert('Para gerar uma nova sala, digite a senha do host (CLARO).');
  const code=genCode();
  await enterRoom(code,{forceHost:true});
  alert('Nova sala criada: '+code+'. Compartilhe este código.');
});
document.getElementById('joinCodeBtn').addEventListener('click', async ()=>{
  const code=(qs('#joinCode').value||'').trim().toUpperCase();
  if(!code) return alert('Digite um código de sala.');
  await enterRoom(code,{forceJoin:true});
});
(function(){
  const el=document.getElementById('joinCode'); let t=null;
  const go=()=>{ const v=(el.value||'').trim().toUpperCase(); el.value=v; if(t) clearTimeout(t); if(v.length>=5) t=setTimeout(()=>enterRoom(v,{forceJoin:true}),300); };
  el.addEventListener('input',go); el.addEventListener('keyup',e=>{ if(e.key==='Enter') go(); });
})();

async function enterRoom(roomId, opts={}){
  const baseName=(qs('#playerName').value||'').trim();
  if(!baseName) return alert('Digite seu nome');
  await ensureAuth();
  roomRef=db.ref('rooms/'+roomId);
  const wantHost=opts.forceHost || ((qs('#hostPwd').value||'').trim()==='CLARO');
  if(wantHost){
    const ok=await roomRef.child('host').transaction(cur=>cur||uid).then(r=>r.committed && r.snapshot.val()===uid);
    if(!ok) return alert('Esta sala já possui host. Entre como participante.');
    await roomRef.transaction(cur=>cur||({
      createdAt:firebase.database.ServerValue.TIMESTAMP, host:uid,
      roundIdx:0, rounds:DECK.rounds, scores:{}, state:'playing',
      firstWinner:{}, roundWinners:{}, guesses:{}, roundLock:false, countdown:null, showPodium:null, showResults:null, highlight:null
    }));
    isHost=true;
  } else { isHost=false; }

  const pSnap=await roomRef.child('players').get();
  const players=pSnap.val()||{};
  const existing=new Set(Object.values(players).map(p=>p.name));
  let name=baseName;
  if(existing.has(name)){ let n=2; while(existing.has(`${baseName} (${n})`)) n++; name=`${baseName} (${n})`; }
  me={name};
  await roomRef.child('players/'+uid).set(me);

  room=roomId;
  qs('#roomInfo').textContent='Sala: '+room;
  qs('#roleInfo').textContent=isHost?'Host':'Jogador';
  qs('#playerHeader').textContent='Jogador — '+me.name;
  document.getElementById('lobbyOverlay').style.display='none';

  document.getElementById('next').disabled = !isHost;
  document.getElementById('resetScores').disabled = !isHost;
  document.getElementById('showPodium').disabled = !isHost;

  subscribeRoom(); renderRound();
}

function subscribeRoom(){
  roomRef.child('host').on('value', s=>{hostUid=s.val()||null;});
  roomRef.child('state').on('value', s=>{ if(s.val()==='finished') showFinalOverlay(); });
  roomRef.child('roundIdx').on('value', async s=>{
    const v=s.val(); roundIdx=(typeof v==='number')?v:0;
    qs('#rnum').textContent=String(roundIdx+1);
    clickedInRound=false;
    renderRound();
  });
  roomRef.child('players').on('value', s=>{playersCache=s.val()||{};});
  roomRef.child('scores').on('value', s=>{scoresCache=s.val()||{}; updateTop3FromScores();});
  roomRef.child('roundWinners').on('value', s=>{
    const all=s.val()||{};
    const winnersObj=all[String(roundIdx)]||all[roundIdx]||{};
    renderRoundRank(winnersObj);
  });
  roomRef.child('roundLock').on('value', s=>{ isRoundLocked = !!s.val(); });

  /* Contagem (áudio + badge) */
  roomRef.child('countdown').on('value', s=>{
    const data = s.val();
    if(data && typeof data.duration==='number' && typeof data.startAt==='number'){
      startCountdownBadgeUI(data);
    }else{ stopCountdownBadgeUI(); }
  });

  roomRef.child('showResults').on('value', s=>{
    const v=s.val();
    if(v && typeof v.round==='number') showResultsModal(v.round);
    else hideResultsModal();
  });
  roomRef.child('showPodium').on('value', s=>{
    const v=s.val();
    if(v){ updatePodium(); showPodiumOverlay(); }
    else { hidePodiumOverlay(); }
  });
  roomRef.child('highlight').on('value', s=>{
    const all=s.val()||{};
    const h = all[String(roundIdx)] || all[roundIdx];
    if(h && typeof h.sid==='number') applyHighlight(h.sid);
  });
}

/* ===== Render das cartas (5 círculos) ===== */
let currentCenterEls = {};
let currentMineEls = {};

function renderRound(){
  const r=(DECK.rounds[roundIdx]||{center:0,mine:1,match:0});
  const center=DECK.cards[r.center], mine=DECK.cards[r.mine], common=r.match;
  const centerEl=document.getElementById('center'), mineEl=document.getElementById('mine'); clear(centerEl); clear(mineEl);
  currentCenterEls = {}; currentMineEls = {};

  const BASE_POS=[[28,26,44],[72,26,44],[50,50,40],[28,74,44],[72,74,44]];
  const SAFETY=2;
  const hypot=(a,b)=>Math.hypot(a,b), clamp=(x,a,b)=>Math.max(a,Math.min(b,x));
  function computeScale(posList){
    let s=1.0;
    for(const [x,y,sz] of posList){
      const r=sz/2;
      s=Math.min(s,(x-SAFETY)/r,(100-x-SAFETY)/r,(y-SAFETY)/r,(100-y-SAFETY)/r);
    }
    for(let i=0;i<posList.length;i++) for(let j=i+1;j<posList.length;j++){
      const [x1,y1,sz1]=posList[i],[x2,y2,sz2]=posList[j];
      const d=hypot(x2-x1,y2-y1), rsum=(sz1+sz2)/2;
      s=Math.min(s,(d-SAFETY)/rsum);
    }
    return clamp(s,0.62,1.0);
  }
  function assignPositions(ids,pref){
    const used=new Set(), out={};
    if(pref) for(const sid of ids){ const p=pref[sid]; if(p!=null && !used.has(p)){ out[sid]=p; used.add(p); } }
    for(const sid of ids) if(out[sid]==null) for(let i=0;i<BASE_POS.length;i++) if(!used.has(i)){ out[sid]=i; used.add(i); break; }
    return out;
  }
  function placeToken(cardEl,sid,posIdx,scale,clickable,onClick){
    const [x,y,sz0]=BASE_POS[posIdx], sz=sz0*scale;
    const t=document.createElement('button'); t.className='token';
    t.dataset.sid = String(sid);
    t.style.width=sz+'%'; t.style.height=sz+'%';
    t.style.left=`calc(${x}% - ${sz/2}%)`; t.style.top=`calc(${y}% - ${sz/2}%)`;
    const sym=symbols.find(s=>s.id===sid);
    const img=document.createElement('img'); img.alt=sym.label; img.src = sym.img || '';
    t.appendChild(img);
    if(clickable && onClick) t.addEventListener('click', onClick);
    cardEl.appendChild(t); return t;
  }

  const mapCenter=assignPositions(center,null);
  const mapMine=assignPositions(mine, {[common]:(mapCenter[common]+1)%5});
  const scaleCenter=computeScale(center.map(sid=>BASE_POS[mapCenter[sid]]));
  const scaleMine=computeScale(mine.map(sid=>BASE_POS[mapMine[sid]]));

  // mesa (não clica)
  center.forEach(sid=>{
    const el = placeToken(centerEl,sid,mapCenter[sid],scaleCenter,false,null);
    currentCenterEls[sid]=el;
  });

  // jogador (sem handler aqui; adiciono abaixo)
  mine.forEach(sid=>{
    const el = placeToken(mineEl,sid,mapMine[sid],scaleMine,false,null);
    currentMineEls[sid]=el;
  });

  /* único listener oficial de clique do jogador */
  Array.from(mineEl.children).forEach(btn=>{
    btn.addEventListener('click', e=>{
      if (clickedInRound || isRoundLocked) return;
      const r=(DECK.rounds[roundIdx]||{match:0});
      const sid = Number(e.currentTarget.dataset.sid);
      clickedInRound = true; // trava a rodada para este jogador

      if (sid!==r.match){
        // clique errado: azul e encerra a tentativa
        e.currentTarget.classList.add('is-wrong');
        return;
      }

      // correto: amarelo nos dois lados
      applyHighlight(r.match);

      // registra no ranking da rodada
      const myEntryRef = roomRef.child('roundWinners/'+roundIdx+'/'+uid);
      myEntryRef.set({name:(me?.name||'Jogador'), ts: firebase.database.ServerValue.TIMESTAMP}).then(()=>{

        // apenas o 1º ganha ponto + highlight global
        const firstRef=roomRef.child('firstWinner/'+roundIdx);
        firstRef.transaction(cur => cur || {uid, name:(me?.name||'Jogador'), ts: firebase.database.ServerValue.TIMESTAMP}, async (wErr, wCommitted, snap)=>{
          if (wErr || !wCommitted) return;
          const v=snap.val(); if (!v || v.uid !== uid) return;

          await roomRef.child('scores/'+uid).transaction(x => (x||0)+1);
          await roomRef.child('highlight/'+roundIdx).set({sid:r.match, by:uid, at: firebase.database.ServerValue.TIMESTAMP});
        });

      });
    }, {once:false});
  });

  // ranking ao carregar
  roomRef.child('roundWinners/'+roundIdx).once('value').then(s=> renderRoundRank(s.val()||{}));
}

/* ===== Highlight amarelo (garantido nos dois lados) ===== */
function applyHighlight(sid, tries=0){
  const a = currentCenterEls[sid];
  const b = currentMineEls[sid];
  if (a) a.classList.add('is-match');
  if (b) b.classList.add('is-match');
  if ((!a || !b) && tries < 8) setTimeout(() => applyHighlight(sid, tries+1), 50);
}

/* ===== Topo — mostrar "pontos" ===== */
function updateTop3FromScores(){
  const entries = Object.entries(scoresCache||{})
    .map(([id,pts])=>({uid:id, pts:Number(pts)||0}))
    .filter(e=>e.pts>0);

  const arr = entries.map(e=>({
    uid:e.uid,
    name:(playersCache?.[e.uid]?.name)||'??',
    pts:e.pts
  })).sort((a,b)=> b.pts - a.pts || a.name.localeCompare(b.name));

  const t = document.getElementById('topListText');
  const lines = [];
  if(arr[0]) lines.push(`1° Lugar ${arr[0].name} – ${arr[0].pts} pontos`);
  if(arr[1]) lines.push(`2° Lugar ${arr[1].name} – ${arr[1].pts} pontos`);
  if(arr[2]) lines.push(`3° Lugar ${arr[2].name} – ${arr[2].pts} pontos`);
  t.textContent = lines.length ? lines.join("\n") : '—';
}

/* ===== Ranking da rodada ===== */
function renderRoundRank(winnersObj){
  const el=document.getElementById('roundRank'); el.innerHTML='';
  const arr=Object.entries(winnersObj||{}).map(([uid,w])=>({uid,...w})).sort((a,b)=>(a.ts||0)-(b.ts||0));
  arr.forEach((w,i)=>{
    const pill=document.createElement('div'); pill.className='rankpill';
    const tag=i===0?'Pontuou':'Não pontuou';
    pill.innerHTML=`<strong>${i+1}º</strong> ${w.name} <span class="tag">${tag}</span>`;
    el.appendChild(pill);
  });
  const clicked=new Set(arr.map(w=>w.uid));
  Object.keys(playersCache||{}).filter(u=>!clicked.has(u))
    .map(u=>({name:playersCache[u]?.name||'??'}))
    .sort((a,b)=>a.name.localeCompare(b.name))
    .forEach(nc=>{
      const pill=document.createElement('div'); pill.className='rankpill';
      pill.innerHTML = `<strong>sem classificação</strong> ${nc.name}`;
      el.appendChild(pill);
    });
  if(!arr.length && !Object.keys(playersCache||{}).length)
    el.innerHTML='<span style="opacity:.7;">Aguardando acerto…</span>';
}

/* ===== Botões ===== */
document.getElementById('next').addEventListener('click', async ()=>{
  if(!isHost) return alert('Somente o host pode mostrar os resultados.');
  const current=roundIdx;
  await roomRef.child('showResults').set({ round: current, at: firebase.database.ServerValue.TIMESTAMP });
});
document.getElementById('resetScores').addEventListener('click', async ()=>{
  if(!isHost) return alert('Somente o host pode resetar.');
  if(!confirm('Zerar o ranking acumulado desta sala?')) return;
  await roomRef.update({
    scores:{}, roundWinners:{}, guesses:{}, showResults:null, firstWinner:{}, roundIdx:0, state:'playing', roundLock:false, countdown:null, highlight:null
  });
});
document.getElementById('showPodium').addEventListener('click', async ()=>{
  if(!isHost) return alert('Somente o host pode abrir o pódio.');
  await roomRef.child('showPodium').set({ at: firebase.database.ServerValue.TIMESTAMP });
});
document.getElementById('closePodiumBtn').addEventListener('click', async ()=>{
  if(isHost) await roomRef.child('showPodium').set(null);
  else document.getElementById('podiumOverlay').style.display='none';
});
document.getElementById('rnum').textContent='1';

/* ===== Modal + contagem ===== */
async function showResultsModal(roundNumber){
  const overlay=document.getElementById('resultsOverlay');
  const listEl=document.getElementById('resultsList');
  const label=document.getElementById('resultsRoundLabel');
  label.textContent='Rodada '+(roundNumber+1);

  const [playersSnap,winnersSnap]=await Promise.all([
    roomRef.child('players').get(),
    roomRef.child('roundWinners/'+roundNumber).get()
  ]);
  const players=playersSnap.val()||{};
  const winnersObj=winnersSnap.val()||{};
  const winnersArr=Object.entries(winnersObj).map(([uid,w])=>({uid,...w})).sort((a,b)=>(a.ts||0)-(b.ts||0));
  const clicked=new Set(winnersArr.map(w=>w.uid));

  listEl.innerHTML=''; let idx=1;
  winnersArr.forEach(w=>{
    const div=document.createElement('div'); div.className='li';
    const tag=idx===1?'Pontuou':'Não pontuou';
    div.innerHTML=`<div class="idx">${idx++}º</div><div class="name">${w.name}</div><div class="tag">${tag}</div>`;
    listEl.appendChild(div);
  });
  Object.entries(players)
    .filter(([u,_])=>!clicked.has(u))
    .map(([u,v])=>({name:v.name}))
    .sort((a,b)=>a.name.localeCompare(b.name))
    .forEach(nc=>{
      const div=document.createElement('div'); div.className='li missed';
      div.innerHTML=`<div class="idx">sem classificação</div><div class="name">${nc.name}</div>`;
      listEl.appendChild(div);
    });

  const contBtn=document.getElementById('continueBtn');
  const closeBtn=document.getElementById('closeResultsBtn');
  contBtn.style.display = isHost ? 'inline-block' : 'none';
  overlay.style.display='flex';
  closeBtn.onclick = ()=>{ overlay.style.display='none'; };

  contBtn.onclick=async()=>{
    const lastIndex=(DECK.rounds.length-1);
    const nextIndex = Math.min((roundIdx||0)+1, lastIndex);
    await Promise.all([
      roomRef.child('guesses/'+roundNumber).set(null),
      roomRef.child('showResults').set(null),
      roomRef.child('highlight/'+roundNumber).set(null)
    ]);
    if(nextIndex > lastIndex){
      await roomRef.child('state').set('finished');
      return;
    }
    await roomRef.update({ roundIdx: nextIndex, roundLock:false });
    await roomRef.child('countdown').set({
      round: nextIndex, duration: 30,
      startAt: firebase.database.ServerValue.TIMESTAMP
    });
  };
}
function hideResultsModal(){ document.getElementById('resultsOverlay').style.display='none'; }

/* ===== Contagem como BOTÃO no topo + áudio ===== */
let countTimer=null, countEndAt=0, currentCountdownRound=null;
const countBadge = document.getElementById('countBadge');
const roundFileInput = document.getElementById('roundAudioFile');
const playSoundBtn = document.getElementById('playSoundBtn');
const roundAudioEl = document.getElementById('roundAudio');
let customRoundAudioURL = null;

roundFileInput.addEventListener('change', (ev)=>{
  const f = ev.target.files && ev.target.files[0];
  if(!f) return;
  if(customRoundAudioURL) URL.revokeObjectURL(customRoundAudioURL);
  customRoundAudioURL = URL.createObjectURL(f);
  roundAudioEl.src = customRoundAudioURL;
  roundAudioEl.loop = true;
  roundAudioEl.load();
});
playSoundBtn.addEventListener('click', async ()=>{
  ensureAudio();
  try{
    if(roundAudioEl && roundAudioEl.src){ await roundAudioEl.play(); }
    else { startSynth(); }
    playSoundBtn.style.display='none';
  }catch(e){}
});
function startCountdownBadgeUI(data){
  currentCountdownRound = data.round;
  countBadge.style.display='inline-block';
  playSoundBtn.style.display='none';

  const tryPlay = async () => {
    ensureAudio();
    if (roundAudioEl && roundAudioEl.src) {
      try { roundAudioEl.pause(); } catch(_){}
      try { roundAudioEl.currentTime = 0; } catch(_){}
      try { await roundAudioEl.play(); }
      catch(e){ playSoundBtn.style.display='inline-block'; }
    } else {
      startSynth();
    }
  };
  tryPlay();

  const startAt = data.startAt;
  const dur = (data.duration||30)*1000;
  countEndAt = startAt + dur;

  if(countTimer) clearInterval(countTimer);
  const tick=async ()=>{
    const remainMs = Math.max(0, countEndAt - nowServer());
    const s = Math.ceil(remainMs/1000);
    countBadge.textContent = `⏳ ${s}s`;
    if(remainMs<=0){
      stopCountdownBadgeUI();
      if(isHost){
        const sr = await roomRef.child('showResults').get();
        if(!sr.val()){
          await roomRef.update({ roundLock:true });
          await roomRef.child('showResults').set({ round: currentCountdownRound, at: firebase.database.ServerValue.TIMESTAMP });
        }
      }
    }
  };
  tick(); countTimer=setInterval(tick, 200);
}
function stopCountdownBadgeUI(){
  if(countTimer){ clearInterval(countTimer); countTimer=null; }
  countBadge.style.display='none';
  playSoundBtn.style.display='none';
  try{ if(roundAudioEl){ roundAudioEl.pause(); } }catch(_){}
  stopSynth();
}

/* ===== Pódio — “Lugar Nome (x pontos)” ===== */
function computeFinalRankingArray(){
  const players=Object.entries(playersCache||{})
    .filter(([u,_])=>!hostUid || u!==hostUid)
    .map(([u,v])=>({uid:u, name:v?.name||'??'}));
  const arr=players.map(p=>({uid:p.uid, name:p.name, pts:Number((scoresCache||{})[p.uid]||0)}));
  arr.sort((a,b)=> b.pts - a.pts || a.name.localeCompare(b.name));
  return arr;
}
function updatePodium(){
  // monta ranking (host fora)
  const players = Object.entries(playersCache||{})
    .filter(([u,_]) => !hostUid || u !== hostUid)
    .map(([u,v]) => ({ uid:u, name:v?.name || '??', pts:Number((scoresCache||{})[u]||0) }));

  players.sort((a,b)=> b.pts - a.pts || a.name.localeCompare(b.name));

  const p1 = players[0], p2 = players[1], p3 = players[2];

  // textos acima de cada pilar
  const pod1 = document.getElementById('pod1name');
  const pod2 = document.getElementById('pod2name');
  const pod3 = document.getElementById('pod3name');

  pod1.textContent = p1 ? `1° Lugar ${p1.name} (${p1.pts} pontos)` : '—';
  pod2.textContent = p2 ? `2° Lugar ${p2.name} (${p2.pts} pontos)` : '—';
  pod3.textContent = p3 ? `3° Lugar ${p3.name} (${p3.pts} pontos)` : '—';

  // lista em texto (abaixo do pódio) – útil pra conferência
  const txt = [];
  if(p1) txt.push(`1° Lugar ${p1.name} – ${p1.pts} pontos`);
  if(p2) txt.push(`2° Lugar ${p2.name} – ${p2.pts} pontos`);
  if(p3) txt.push(`3° Lugar ${p3.name} – ${p3.pts} pontos`);
  document.getElementById('podListText').textContent = txt.join("\n");
}
function showPodiumOverlay(){ document.getElementById('podiumOverlay').style.display='flex'; }
function hidePodiumOverlay(){ document.getElementById('podiumOverlay').style.display='none'; }

function showFinalOverlay(){
  const overlay=document.getElementById('finalOverlay');
  const listEl=document.getElementById('finalList');
  const congrats=document.getElementById('finalCongrats');
  const arr=computeFinalRankingArray(); listEl.innerHTML='';
  congrats.textContent = arr.length ? `Parabéns ${arr[0].name} pelo 1° lugar` : '';
  let pos=1;
  arr.forEach(p=>{
    const div=document.createElement('div'); div.className='li';
    div.innerHTML=`<div class="idx">${pos++}º</div><div class="name">${p.name}</div><div class="tag">${p.pts} pontos</div>`;
    listEl.appendChild(div);
  });
  overlay.style.display='flex';
  document.getElementById('closeFinalBtn').onclick=()=>{ overlay.style.display='none'; };
}

/* ===== Embutir Figuras ===== */
const embedBtn = document.getElementById('embedBtn');
const embedOverlay = document.getElementById('embedOverlay');
const cancelEmbed = document.getElementById('cancelEmbed');
const genEmbedHtml = document.getElementById('genEmbedHtml');
const filesInput = document.getElementById('files');
const embedStatus = document.getElementById('embedStatus');

embedBtn.onclick = ()=>{ embedOverlay.style.display='flex'; embedStatus.textContent=''; filesInput.value=''; };
cancelEmbed.onclick = ()=>{ embedOverlay.style.display='none'; };
function readFilesAsDataURIs(files){
  return Promise.all(Array.from(files).map(f=> new Promise(res=>{
    const fr=new FileReader(); fr.onload=()=>res(fr.result); fr.readAsDataURL(f);
  })));
}
genEmbedHtml.onclick = async ()=>{
  const files = filesInput.files;
  if(!files || files.length===0){ alert('Selecione 21 imagens (PNG/JPG).'); return; }
  embedStatus.textContent = 'Convertendo imagens…';
  const arr = await readFilesAsDataURIs(files);
  const data = Array(21).fill(0).map((_,i)=> arr[i] || SYMBOL_EMBED[i]);
  const html = buildInlineHtml(data);
  const blob = new Blob([html], {type:'text/html;charset=utf-8'});
  const a = document.createElement('a');
  a.download = 'dobble_v35_inline_figs.html';
  a.href = URL.createObjectURL(blob);
  a.click();
  URL.revokeObjectURL(a.href);
  embedStatus.textContent = 'Pronto! Um novo HTML foi baixado com as imagens embutidas.';
};
function buildInlineHtml(dataUris){
  const src = document.documentElement.outerHTML
    .replace(/const SYMBOL_EMBED = [\s\S]*?;\n/, 'const SYMBOL_EMBED = '+JSON.stringify(dataUris)+';\n');
  return '<!doctype html>\n' + src;
}
</script>
</body>
</html>



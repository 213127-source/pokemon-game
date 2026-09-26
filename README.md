<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pokémon: Legend Hunter</title>
<style>
:root { --primary: #ffcb05; --secondary: #2a75bb; --danger: #ff3e3e; --legendary: #9b59b6; }
* { box-sizing: border-box; }
body {
    font-family: 'Helvetica Neue', Arial, sans-serif;
    background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
    margin: 0; display: flex; justify-content: center; align-items: center;
    min-height: 100vh; color: #333;
}
#app {
    width: 100%; max-width: 640px; background: white; border-radius: 15px;
    box-shadow: 0 20px 50px rgba(0,0,0,0.5); overflow: hidden;
    height: 95vh; display: flex; flex-direction: column; position: relative;
}
header {
    background: var(--secondary); color: white; padding: 12px 15px;
    display: flex; justify-content: space-between; align-items: center; font-weight: bold; z-index: 10;
}
.rank-badge { background: var(--primary); color: #333; padding: 4px 12px; border-radius: 15px; border: 2px solid #c7a008; }
.phase-badge { background: var(--legendary); color: white; padding: 4px 10px; border-radius: 15px; font-size: 0.8rem; margin-left: 5px; }

/* 🎵 BGMボタンを最前面に固定 */
#music-control {
    position: absolute; top: 10px; right: 10px; z-index: 999;
}
.music-btn {
    background: rgba(0,0,0,0.7); color: white; border: 2px solid white;
    padding: 8px 15px; border-radius: 20px; font-size: 0.9rem; cursor: pointer;
    font-weight: bold; transition: 0.2s;
}
.music-btn:hover { background: rgba(0,0,0,0.9); transform: scale(1.05); }
.music-btn.on { background: #4caf50; border-color: #4caf50; }

#loading-screen {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255,255,255,0.98); display: flex; flex-direction: column;
    justify-content: center; align-items: center; z-index: 100;
}
.loader { border: 5px solid #f3f3f3; border-top: 5px solid var(--secondary); border-radius: 50%; width: 50px; height: 50px; animation: spin 1s linear infinite; margin-bottom: 15px; }
@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

.screen { display: none; flex: 1; flex-direction: column; padding: 15px; overflow-y: auto; }
.screen.active { display: flex; }

.search-bar { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 8px; margin-bottom: 15px; font-size: 16px; }
.poke-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(75px, 1fr)); gap: 8px; }
.poke-card { border: 1px solid #eee; border-radius: 8px; padding: 6px; text-align: center; cursor: pointer; transition: 0.2s; background: #fff; position: relative; }
.poke-card:hover:not(.locked) { background: #eef; transform: translateY(-2px); box-shadow: 0 3px 8px rgba(0,0,0,0.1); }
.poke-card.locked { opacity: 0.5; cursor: not-allowed; filter: grayscale(100%); }
.poke-card.locked::after { content: "🔒"; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 24px; }
.poke-card img { width: 55px; height: 55px; image-rendering: pixelated; }
.poke-id { font-size: 0.65em; color: #888; }
.poke-name { font-size: 0.72em; font-weight: bold; margin-top: 2px; }

.battle-field {
    flex: 1; background: linear-gradient(to bottom, #87CEEB 60%, #90EE90 60%);
    border-radius: 10px; position: relative; margin-bottom: 10px;
    display: flex; justify-content: space-between; align-items: flex-end;
    padding: 15px 30px; min-height: 200px; transition: background 1s;
}
.battle-field.boss-bg { background: linear-gradient(to bottom, #2c0000 60%, #4a0000 60%); animation: bossShake 0.5s infinite; }
.battle-field.legend-bg { background: linear-gradient(to bottom, #FFD700 60%, #FF8C00 60%); }
@keyframes bossShake { 0%,100%{transform:translate(0)} 25%{transform:translate(-3px,2px)} 75%{transform:translate(3px,-2px)} }

.fighter { text-align: center; width: 130px; position: relative; }
.fighter img { width: 100px; height: 100px; filter: drop-shadow(0 5px 5px rgba(0,0,0,0.3)); transition: 0.3s; }
.fighter.mega img { filter: drop-shadow(0 0 15px gold); }
.fighter.dynamax img { transform: scale(1.3); filter: drop-shadow(0 0 20px red); }
.fighter.tera img { filter: drop-shadow(0 0 15px cyan); }
.fighter.stella img { filter: drop-shadow(0 0 20px #ff00ff) drop-shadow(0 0 30px #00ffff); }

.hp-box { background: rgba(0,0,0,0.6); color: white; padding: 3px 6px; border-radius: 4px; font-size: 0.75em; margin-top: 5px; }
.hp-bar-bg { width: 100%; height: 6px; background: #333; border-radius: 3px; margin-top: 3px; overflow: hidden; }
.hp-bar-fill { height: 100%; background: #4caf50; width: 100%; transition: width 0.3s; }
.status-effects { display: flex; gap: 3px; justify-content: center; margin-top: 3px; flex-wrap: wrap; }
.status-tag { font-size: 0.6em; padding: 1px 5px; border-radius: 8px; color: white; font-weight: bold; }
.tag-mega { background: gold; color: black; }
.tag-dynamax { background: #ff3e3e; }
.tag-tera { background: #00ced1; }
.tag-stella { background: linear-gradient(45deg, #ff00ff, #00ffff); }

.controls { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; margin-bottom: 8px; }
.special-controls { display: grid; grid-template-columns: repeat(auto-fit, minmax(100px, 1fr)); gap: 6px; margin-bottom: 8px; }
button { padding: 12px 8px; border: none; border-radius: 8px; font-size: 0.9rem; font-weight: bold; cursor: pointer; color: white; transition: 0.1s; }
button:active:not(:disabled) { transform: scale(0.97); }
button:disabled { opacity: 0.4; cursor: not-allowed; }
.btn-atk { background: #ff6b6b; }
.btn-heal { background: #4caf50; }
.btn-def { background: #2196f3; }
.btn-ball { background: #ff3e3e; border: 2px solid white; }
.btn-next { background: var(--primary); color: #333; grid-column: span 2; }
.btn-tera { background: linear-gradient(45deg, #00ced1, #1e90ff); }
.btn-z { background: linear-gradient(45deg, #9b59b6, #8e44ad); }
.btn-dynamax { background: linear-gradient(45deg, #ff3e3e, #c0392b); }
.btn-mega { background: linear-gradient(45deg, #f39c12, #e67e22); }
.btn-stella { background: linear-gradient(45deg, #ff00ff, #00ffff); color: white; }

.log-area { height: 90px; background: #222; color: #fff; padding: 8px; border-radius: 8px; font-size: 0.85em; overflow-y: auto; font-family: monospace; }

#ending-screen { background: #000; color: white; justify-content: center; align-items: center; text-align: center; position: relative; overflow: hidden; }
#ending-screen h1 { font-size: 2.5rem; background: linear-gradient(45deg, gold, #ff6b6b, #00ced1); -webkit-background-clip: text; -webkit-text-fill-color: transparent; margin-bottom: 20px; z-index: 2; }
.caught-list { display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; margin: 20px 0; z-index: 2; max-height: 300px; overflow-y: auto; }
.caught-poke { background: rgba(255,255,255,0.1); padding: 8px; border-radius: 8px; border: 1px solid gold; animation: fadeIn 0.5s ease-out; }
.caught-poke img { width: 50px; height: 50px; }

@keyframes shake { 0%{transform:translateX(0)} 25%{transform:translateX(-5px)} 75%{transform:translateX(5px)} 100%{transform:translateX(0)} }
.shake { animation: shake 0.4s; }
@keyframes flash { 0%,100%{opacity:1} 50%{opacity:0.3} }
.flash { animation: flash 0.3s 3; }
@keyframes fadeIn { from{opacity:0;transform:translateY(20px)} to{opacity:1;transform:translateY(0)} }
</style>
</head>
<body>

<div id="app">
    <!-- 🎵 音楽コントロールボタン -->
    <div id="music-control">
        <button class="music-btn" onclick="toggleMusic()">🎵 BGM: OFF</button>
    </div>
    
    <header>
        <span>POKÉMON LEAGUE</span>
        <div>
            <span id="rank-display" class="rank-badge">RANK Z</span>
            <span id="phase-display" class="phase-badge" style="display:none;"></span>
        </div>
    </header>

    <div id="loading-screen">
        <div class="loader"></div>
        <div id="loading-text">ポケモンデータを読み込み中... (1025匹)</div>
    </div>

    <div id="select-screen" class="screen">
        <h3>パートナー選択 <small style="font-size:0.6em; color:#666;">※伝説・幻は選択できません</small></h3>
        <input type="text" id="search-input" class="search-bar" placeholder="名前または番号で検索...">
        <div id="pokemon-list" class="poke-grid"></div>
    </div>

    <div id="battle-screen" class="screen">
        <div class="battle-field" id="battle-field">
            <div class="fighter" id="enemy-fighter">
                <div class="name" id="enemy-name">Enemy</div>
                <img id="enemy-img" src="" alt="">
                <div class="hp-box"><span id="enemy-hp-text">100/100</span></div>
                <div class="hp-bar-bg"><div id="enemy-hp-bar" class="hp-bar-fill"></div></div>
                <div class="status-effects" id="enemy-status"></div>
            </div>
            <div class="fighter" id="player-fighter">
                <div class="name" id="player-name">Player</div>
                <img id="player-img" src="" alt="">
                <div class="hp-box"><span id="player-hp-text">100/100</span></div>
                <div class="hp-bar-bg"><div id="player-hp-bar" class="hp-bar-fill"></div></div>
                <div class="status-effects" id="player-status"></div>
            </div>
        </div>
        <div class="log-area" id="battle-log">バトル開始！</div>
        <div class="special-controls" id="special-controls"></div>
        <div class="controls" id="main-controls">
            <button class="btn-atk" onclick="action('attack')">こうげき</button>
            <button class="btn-heal" onclick="action('heal')">かいふく</button>
            <button class="btn-def" onclick="action('defend')">ぼうぎょ</button>
            <button class="btn-ball" id="ball-btn" onclick="action('ball')" style="display:none;"> モンスターボール</button>
        </div>
        <button id="next-btn" class="btn-next" onclick="nextRank()" style="display:none;">次のランクへ >></button>
    </div>

    <div id="ending-screen" class="screen">
        <h1> LEGEND MASTER 🏆</h1>
        <p style="font-size:1.1rem; z-index:2;">あなたは全ての伝説を制覇した！</p>
        <p style="z-index:2;">捕まえた伝説・幻ポケモン：</p>
        <div class="caught-list" id="caught-list"></div>
        <button class="btn-next" style="z-index:2; width:200px;" onclick="location.reload()">もう一度挑戦する</button>
    </div>
</div>

<script>
const TOTAL_POKEMON = 1025;
const RANKS = ['Z','Y','X','W','V','U','T','S','R','Q','P','O','N','M','L','K','J','I','H','G','F','E','D','C','B','A'];
const LEGENDARY_IDS = [144,145,146,150,151,243,244,245,249,250,251,377,378,379,380,381,382,383,384,385,386,480,481,482,483,484,485,486,487,488,489,490,491,492,493,494,638,639,640,641,642,643,644,645,646,647,648,649,716,717,718,719,720,721,785,786,787,788,789,790,791,792,800,801,802,807,808,809,888,889,890,891,892,893,894,895,896,897,898,1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1011,1012,1013,1014,1015,1016,1017,1018,1019,1020,1021,1022,1023,1024,1025];
const JP_NAMES = {144:'フリーザー',145:'サンダー',146:'ファイヤー',150:'ミュウツー',151:'ミュウ',243:'ライコウ',244:'エンテイ',245:'スイクン',249:'ルギア',250:'ホウオウ',251:'セレビィ',377:'レジロック',378:'レジアイス',379:'レジスチル',380:'ラティアス',381:'ラティオス',382:'カイオーガ',383:'グラードン',384:'レックウザ',385:'ジラーチ',386:'デオキシス',480:'ユクシー',481:'エムリット',482:'アグノム',483:'ディアルガ',484:'パルキア',485:'ヒードラン',486:'レジギガス',487:'ギラティナ',488:'クレセリア',489:'フィオネ',490:'マナフィ',491:'ダークライ',492:'シェイミ',493:'アルセウス',494:'ビクティニ',638:'テラキオン',639:'ランドロス',640:'ビリジオン',641:'トルネロス',642:'ボルトロス',643:'レシラム',644:'ゼクロム',645:'ランドロス',646:'キュレム',647:'ケルディオ',648:'メロエッタ',649:'ゲノセクト',716:'ゼルネアス',717:'イベルタル',718:'ジガルデ',719:'ディアンシー',720:'フーパ',721:'ボルケニオン',785:'カプ・コケコ',786:'カプ・テテフ',787:'カプ・ブルル',788:'カプ・レヒレ',789:'コスモッグ',790:'コスモウム',791:'ソルガレオ',792:'ルナアーラ',800:'ネクロズマ',801:'マギアナ',802:'マルシャドウ',807:'ゼラオラ',808:'メルタン',809:'メルメタル',888:'ザシアン',889:'ザマゼンタ',890:'ムゲンダイナ',891:'ダクマ',892:'ウーラオス',893:'ザルード',894:'レジエレキ',895:'レジドラゴ',896:'ブリザポス',897:'レイスポス',898:'バドレックス',1001:'ココルリ',1002:'パオジアン',1003:'チヨンジェン',1004:'イーシン',1005:'ウネアマルシ',1006:'テツノブジン',1007:'コーライドン',1008:'ミライドン',1014:'オーガポン',1016:'キテツナ',1017:'ウガツホムラ',1018:'リキキリン',1019:'ブラッキュン',1020:'カムツゴ',1021:'テツノイサハ',1022:'テツノカシラ',1023:'テツノツブテ',1024:'テラパゴス',1025:'オーガポン'};

let allPokemon = []; let player = null; let enemy = null; let currentRankIdx = 0; let isPlayerTurn = true;
let gamePhase = 'select'; let caughtLegendaries = []; let legendTourIndex = 0;
let musicPlaying = false; let audioCtx = null; let musicInterval = null; let currentMusicType = '';

let playerState = { terastalUsed:false, zMoveUsed:false, dynamaxUsed:false, megaUsed:false, stellaUsed:false, isMega:false, isDynamax:false, isTera:false, isStella:false, defending:false };

const els = { loading:document.getElementById('loading-screen'), loadText:document.getElementById('loading-text'), selectScreen:document.getElementById('select-screen'), battleScreen:document.getElementById('battle-screen'), endingScreen:document.getElementById('ending-screen'), list:document.getElementById('pokemon-list'), search:document.getElementById('search-input'), rank:document.getElementById('rank-display'), phase:document.getElementById('phase-display'), log:document.getElementById('battle-log'), nextBtn:document.getElementById('next-btn'), ballBtn:document.getElementById('ball-btn'), field:document.getElementById('battle-field'), specialControls:document.getElementById('special-controls') };

async function init() {
    try {
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${TOTAL_POKEMON}`);
        const data = await response.json();
        allPokemon = data.results.map((p, index) => {
            const id = index + 1;
            return { id, name:p.name, jpName:JP_NAMES[id]||capitalize(p.name), isLegendary:LEGENDARY_IDS.includes(id), hp:Math.floor(40+(id*0.05)+(Math.random()*20)), atk:Math.floor(35+(id*0.05)+(Math.random()*20)) };
        });
        renderList(allPokemon); els.loading.style.display='none'; els.selectScreen.classList.add('active');
        els.search.addEventListener('input', (e) => { const t=e.target.value.toLowerCase(); renderList(allPokemon.filter(p=>p.name.toLowerCase().includes(t)||p.jpName.includes(t)||p.id.toString()===t)); });
    } catch(err) { els.loadText.innerText="エラー: データを読み込めませんでした"; }
}

function renderList(list) {
    els.list.innerHTML=''; list.forEach(p => {
        const div=document.createElement('div'); div.className='poke-card';
        if(p.isLegendary){div.classList.add('locked');div.title="伝説・幻ポケモンはボス戦後にのみ選択可能です";} else {div.style.border='1px solid #ddd';}
        div.innerHTML=`<img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${p.id}.png" loading="lazy"><div class="poke-id">No.${p.id}</div><div class="poke-name">${p.jpName}</div>`;
        if(!p.isLegendary) div.onclick=()=>selectPokemon(p);
        els.list.appendChild(div);
    });
}
function capitalize(s){return s.charAt(0).toUpperCase()+s.slice(1);}

function selectPokemon(p) {
    if(confirm(`${p.jpName} を パートナーにしますか？`)) {
        player={...p, baseMaxHp:p.hp*4, maxHp:p.hp*4, currentHp:p.hp*4, baseAtk:p.atk*2, atk:p.atk*2}; startGame();
    }
}
function startGame() { els.selectScreen.classList.remove('active'); els.battleScreen.classList.add('active'); currentRankIdx=0; gamePhase='battle'; startBattle(); }

// ==================== MUSIC SYSTEM ====================
function toggleMusic() {
    if(!audioCtx) audioCtx=new(window.AudioContext||window.webkitAudioContext)();
    musicPlaying=!musicPlaying;
    const btn=document.querySelector('.music-btn');
    btn.innerText=musicPlaying?"🎵 BGM: ON":"🎵 BGM: OFF";
    btn.classList.toggle('on', musicPlaying);
    
    // 確認音（ピッ！）
    if(musicPlaying && audioCtx.state==='suspended') audioCtx.resume();
    if(musicPlaying) {
        playTone(880, 0.1, 'sine', 0.2);
        changeMusic(currentMusicType || 'gentle');
    } else {
        stopMusic();
    }
}

function playTone(freq,dur,type='sine',vol=0.1) {
    if(!audioCtx||!musicPlaying) return;
    const osc=audioCtx.createOscillator(), gain=audioCtx.createGain();
    osc.type=type; osc.frequency.setValueAtTime(freq,audioCtx.currentTime);
    gain.gain.setValueAtTime(vol,audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01,audioCtx.currentTime+dur);
    osc.connect(gain); gain.connect(audioCtx.destination); osc.start(); osc.stop(audioCtx.currentTime+dur);
}

function stopMusic() { if(musicInterval){clearInterval(musicInterval);musicInterval=null;} }

function changeMusic(type) {
    if(currentMusicType===type || !musicPlaying) return;
    currentMusicType=type; stopMusic();
    
    if(type==='gentle') { // ランクZ〜N: 優しいピアノ風
        const m=[261,293,329,349,392,349,329,293]; let i=0;
        musicInterval=setInterval(()=>{playTone(m[i],0.6,'sine',0.08);if(i%4===0)playTone(m[i]/2,1.2,'triangle',0.04);i=(i+1)%m.length;},600);
    } else if(type==='intense') { // ランクM〜A: ちょっと激しいロック風
        const m=[196,220,246,261,246,220,196,174]; let i=0;
        musicInterval=setInterval(()=>{playTone(m[i],0.3,'square',0.06);if(i%2===0)playTone(m[i]/2,0.6,'sawtooth',0.04);i=(i+1)%m.length;},300);
    } else if(type==='boss') { // ボス戦: めちゃくちゃ激しいメタル風
        const m=[146,164,174,196,174,164,146,130]; let i=0;
        musicInterval=setInterval(()=>{playTone(m[i],0.15,'sawtooth',0.08);playTone(m[i]*2,0.15,'square',0.05);if(i%4===0)playTone(m[i]/2,0.3,'triangle',0.06);i=(i+1)%m.length;},150);
    } else if(type==='ending') {
        const m=[392,440,493,523,493,440,392,349,329,349,392,440,392,349,329,293]; let i=0;
        musicInterval=setInterval(()=>{playTone(m[i],0.4,'sine',0.1);if(i%4===0)playTone(m[i]/2,0.8,'triangle',0.05);i=(i+1)%m.length;},500);
    }
}

// ==================== BATTLE ====================
function startBattle() {
    const isBoss=currentRankIdx>=RANKS.length;
    if(gamePhase==='legendTour'){
        els.rank.innerText="LEGEND"; els.phase.innerText=`${legendTourIndex+1}/${LEGENDARY_IDS.length}`; els.phase.style.display='inline';
        els.field.classList.add('legend-bg'); els.field.classList.remove('boss-bg');
    } else if(isBoss){
        els.rank.innerText="BOSS"; els.phase.innerText="FINAL BOSS"; els.phase.style.display='inline';
        gamePhase='boss'; els.field.classList.add('boss-bg'); els.field.classList.remove('legend-bg');
        changeMusic('boss'); // ボス戦で激しい音楽に変更
    } else {
        els.rank.innerText=`RANK ${RANKS[currentRankIdx]}`; els.phase.style.display='none';
        els.field.classList.remove('boss-bg','legend-bg');
        
        // 音楽切り替え判定
        if(currentRankIdx===13) { // ランクN (index 13) でメッセージ＋音楽変更
            log("🎵 音楽が変わります！");
            changeMusic('intense');
        } else if(currentRankIdx<=13) {
            changeMusic('gentle');
        }
    }

    els.nextBtn.style.display='none'; els.ballBtn.style.display=(gamePhase==='legendTour')?'block':'none';
    isPlayerTurn=true; 
    if(gamePhase!=='legendTour'){player.maxHp=player.baseMaxHp;player.atk=player.baseAtk;player.currentHp=player.baseMaxHp;}
    playerState={terastalUsed:false,zMoveUsed:false,dynamaxUsed:false,megaUsed:false,stellaUsed:false,isMega:false,isDynamax:false,isTera:false,isStella:false,defending:false};
    els.log.innerHTML='';

    if(gamePhase==='boss'){
        enemy={name:"MEGA ミュウツー",id:150,isLegendary:true,maxHp:5000,currentHp:5000,atk:350};
        log(`⚠️ 最終ボス MEGAミュウツー が現れた！`);
    } else if(gamePhase==='legendTour'){
        const lid=LEGENDARY_IDS[legendTourIndex], bd=allPokemon.find(p=>p.id===lid);
        enemy={name:bd.jpName,id:bd.id,isLegendary:true,maxHp:2000,currentHp:2000,atk:250};
        log(`野生の伝説ポケモン ${enemy.name} が現れた！ 捕まえよう！`);
    } else {
        const mid=Math.min(TOTAL_POKEMON,100+(currentRankIdx*40)), rid=Math.floor(Math.random()*mid)+1;
        const bd=allPokemon.find(p=>p.id===rid)||allPokemon[0], sc=1+(currentRankIdx*0.15);
        enemy={name:bd.jpName,id:bd.id,isLegendary:false,maxHp:Math.floor(bd.hp*3*sc),currentHp:Math.floor(bd.hp*3*sc),atk:Math.floor(bd.atk*1.5*sc)};
        log(`野生の ${enemy.name} が飛び出してきた！`);
    }
    updateUI(); renderSpecialControls();
}

function updateUI() {
    document.getElementById('player-name').innerText=player.jpName;
    const pi=document.getElementById('player-img'); pi.src=`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/back/${player.id}.png`;
    const pf=document.getElementById('player-fighter'); pf.classList.remove('mega','dynamax','tera','stella');
    if(playerState.isStella)pf.classList.add('stella');else if(playerState.isTera)pf.classList.add('tera');else if(playerState.isMega)pf.classList.add('mega');else if(playerState.isDynamax)pf.classList.add('dynamax');
    updateHp('player',player); renderStatus('player-status',playerState);
    document.getElementById('enemy-name').innerText=enemy.name;
    document.getElementById('enemy-img').src=`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${enemy.id}.png`;
    updateHp('enemy',enemy);
}
function renderStatus(id,s){const e=document.getElementById(id);e.innerHTML='';if(s.isMega)e.innerHTML+='<span class="status-tag tag-mega">MEGA</span>';if(s.isDynamax)e.innerHTML+='<span class="status-tag tag-dynamax">DYNAMAX</span>';if(s.isStella)e.innerHTML+='<span class="status-tag tag-stella">ステラ</span>';else if(s.isTera)e.innerHTML+='<span class="status-tag tag-tera">テラスタル</span>';}
function updateHp(w,d){const p=Math.max(0,(d.currentHp/d.maxHp)*100);document.getElementById(`${w}-hp-bar`).style.width=`${p}%`;document.getElementById(`${w}-hp-text`).innerText=`${Math.ceil(d.currentHp)}/${d.maxHp}`;const b=document.getElementById(`${w}-hp-bar`);if(p<20)b.style.background='#ff3e3e';else if(p<50)b.style.background='#ffcb05';else b.style.background='#4caf50';}
function log(m){els.log.innerHTML+=`<div>> ${m}</div>`;els.log.scrollTop=els.log.scrollHeight;}

function renderSpecialControls(){
    els.specialControls.innerHTML=''; const r=currentRankIdx;
    if(!playerState.terastalUsed)addSpecialBtn('テラスタル','btn-tera',useTerastal);
    if(r>=5&&!playerState.zMoveUsed)addSpecialBtn('Z技','btn-z',useZMove);
    if(r>=10&&!playerState.dynamaxUsed)addSpecialBtn('ダイマックス','btn-dynamax',useDynamax);
    if(r>=17&&!playerState.megaUsed)addSpecialBtn('メガシンカ','btn-mega',useMega);
    if(r>=25&&!playerState.stellaUsed)addSpecialBtn('ステラタイプ','btn-stella',useStella);
}
function addSpecialBtn(l,c,f){const b=document.createElement('button');b.className=c;b.innerText=l;b.onclick=()=>{if(!isPlayerTurn)return;f();};els.specialControls.appendChild(b);}
function useTerastal(){if(playerState.terastalUsed)return;playerState.terastalUsed=true;playerState.isTera=true;player.atk=Math.floor(player.baseAtk*1.5);log(`✨ ${player.jpName} は テラスタルした！`);animFlash('player-fighter');updateUI();renderSpecialControls();}
function useZMove(){if(playerState.zMoveUsed||!isPlayerTurn)return;playerState.zMoveUsed=true;const d=Math.floor(player.atk*3);enemy.currentHp-=d;log(`💥 Z技発動！ ${d}の大ダメージ！`);animHit('enemy-fighter');updateUI();renderSpecialControls();if(enemy.currentHp<=0)winRound();else setTimeout(enemyTurn,1000);isPlayerTurn=false;}
function useDynamax(){if(playerState.dynamaxUsed||!isPlayerTurn)return;playerState.dynamaxUsed=true;playerState.isDynamax=true;player.maxHp=Math.floor(player.baseMaxHp*2);player.currentHp=player.maxHp;player.atk=Math.floor(player.baseAtk*1.5);log(`🔥 ダイマックス！ HPと攻撃力がUP！`);animFlash('player-fighter');updateUI();renderSpecialControls();}
function useMega(){if(playerState.megaUsed||!isPlayerTurn)return;playerState.megaUsed=true;playerState.isMega=true;player.atk=Math.floor(player.baseAtk*2);player.maxHp=Math.floor(player.baseMaxHp*1.3);player.currentHp=Math.min(player.currentHp+100,player.maxHp);log(`💫 メガシンカ！ 全ステータスUP！`);animFlash('player-fighter');updateUI();renderSpecialControls();}
function useStella(){if(playerState.stellaUsed||!isPlayerTurn)return;playerState.stellaUsed=true;playerState.isStella=true;playerState.isTera=false;player.atk=Math.floor(player.baseAtk*2.5);player.maxHp=Math.floor(player.baseMaxHp*1.5);player.currentHp=player.maxHp;log(`🌟 ステラタイプ！ 最強の力！`);animFlash('player-fighter');updateUI();renderSpecialControls();}

function action(type){
    if(!isPlayerTurn)return; isPlayerTurn=false; playerState.defending=false; let d=0,m="";
    if(type==='attack'){const c=Math.random()<0.1;d=Math.floor(player.atk*(0.8+Math.random()*0.4)*(c?1.5:1));enemy.currentHp-=d;m=`${player.jpName} の こうげき！ ${d}ダメージ！${c?'(会心!)':''}`;animHit('enemy-fighter');}
    else if(type==='heal'){const h=Math.floor(player.maxHp*0.3);player.currentHp=Math.min(player.maxHp,player.currentHp+h);m=`${player.jpName} は 体力を回復した！ (+${h})`;}
    else if(type==='defend'){playerState.defending=true;m=`${player.jpName} は 身を守っている！`;}
    else if(type==='ball'){if(gamePhase==='legendTour'){log(`🎉 ${enemy.name} を 捕まえた！`);caughtLegendaries.push(enemy);setTimeout(()=>{legendTourIndex++;if(legendTourIndex>=LEGENDARY_IDS.length)showEnding();else startBattle();},1500);return;}}
    log(m); updateUI(); if(enemy.currentHp<=0)winRound(); else setTimeout(enemyTurn,1000);
}
function enemyTurn(){
    if(enemy.currentHp<=0)return;
    const db=Math.floor(enemy.atk*(0.8+Math.random()*0.4)), fd=playerState.defending?Math.floor(db/2):db;
    player.currentHp-=fd; log(`${enemy.name} の こうげき！ ${fd}ダメージ！${playerState.defending?'(軽減)':''}`); animHit('player-fighter'); updateUI();
    if(player.currentHp<=0){log(`${player.jpName} は たおれてしまった...`);setTimeout(()=>{alert("ゲームオーバー...");location.reload();},1000);}
    else isPlayerTurn=true;
}
function animHit(id){const e=document.getElementById(id);e.classList.add('shake');setTimeout(()=>e.classList.remove('shake'),400);}
function animFlash(id){const e=document.getElementById(id);e.classList.add('flash');setTimeout(()=>e.classList.remove('flash'),1000);}

function winRound(){
    log(`${enemy.name} を たおした！`);
    if(gamePhase==='boss'){log(`🎊 ボスを撃破！ 伝説捕獲ツアー開始！`);changeMusic('ending');setTimeout(()=>{gamePhase='legendTour';legendTourIndex=0;startBattle();},2000);}
    else if(gamePhase==='legendTour'){log(`${enemy.name} は 消え去った... 次の伝説へ！`);caughtLegendaries.push({name:enemy.name+"(逃)",id:enemy.id});setTimeout(()=>{legendTourIndex++;if(legendTourIndex>=LEGENDARY_IDS.length)showEnding();else startBattle();},1500);}
    else els.nextBtn.style.display='block';
}
function nextRank(){currentRankIdx++;startBattle();}

function showEnding(){
    els.battleScreen.classList.remove('active'); els.endingScreen.classList.add('active'); changeMusic('ending');
    const l=document.getElementById('caught-list'); l.innerHTML='';
    caughtLegendaries.forEach(p=>{const d=document.createElement('div');d.className='caught-poke';d.innerHTML=`<img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${p.id}.png"><div>${p.name||p.jpName}</div>`;l.appendChild(d);});
}

init();
</script>
</body>
</html>

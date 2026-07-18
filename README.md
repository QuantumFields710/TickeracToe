<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>TickeracToe</title>
<!--TODO:
Updates for v1.3
- Place Os first
- Settings
- Active game cancelation/resigning
- Remove scoring with Xs for original style
- Review last move via settings
- Win/lose/score affects togglable via settings
Long term / v1.4
- Basic temporary ELO scoring
- AI opponent
- Scoring placements glow red/blue based on team instead of green
- Change display
- Connect 4?
- Hive integration?
-->
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
:root{
 --cell-color: #e2e8f0;
 --b-color: #0d86a8ff;
 --p-x-color: #ff5e62;
 --p-o-color: #001aff;
}
body {font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;margin: 0;padding: 20px;background-color: #f0f4f8;color: #1a202c;transition: background-color 0.3s, color 0.3s; }h1 {text-align: center;color: #0b0f19;text-shadow: 0 0 10px rgba(0, 0, 0, 0.1); }
.stat {text-align: center;margin: 20px auto;font-size: 1.2rem;font-weight: bold;min-height: 1.5rem;line-height: 1.6; }
.HC-container {
 display: grid;gap: 20px;margin: 30px auto;justify-content: center;
 background: var(--b-color);
 padding: 20px;
/* margin-left: 450px;
 margin-right: 450px; */
 border: 1px solid #cbd5e1; }
.gc {
display: flex;
justify-content: center;
gap: 30px;
margin-top: 40px;
max-width: 600px;
margin-left: auto;
margin-right: auto;
}
.mc{
flex: 1;
display: flex;
flex-direction: column;
align-items: center;
gap: 12px;
text-align: center;
}
.t-btn {
background-color: #6cc0f7;
color: black;
padding: 12px 20px;
border: none;
border-radius: 5px;
cursor: pointer;
font-size: 16px;
font-weight: bold;
width: 100%;
max-width: 200px;
transition: all 0.2s ease;
box-shadow: 0 4px 6px rgba(0,0,0,0.3);
}
.t-btn:hover {
background-color: #316182;
color: white;
transform: translateY(-2px);
}
#exclamation {
  font-size: 20px;
  position: absolute;
  top: 10px;
  right: 20px;
  background-color: #cef6f8ff;
}
.bigBox {
text-align: center;
max-width: 400px;
margin: 10px auto;
padding: 15px;
border-radius: 5px;
background: rgba(108, 192, 247, 0.1);
}
.bigBox input {
padding: 8px;
font-size: 14px;
border-radius: 4px;
border: 1px solid #cbd5e1;
margin-right: 5px;
}
.bigBox button {
padding: 8px 12px;
font-size: 14px;
cursor: pointer;
background-color: #6cc0f7;
border: none;
border-radius: 4px;
font-weight: bold;
}
.grid-3d-3 { grid-template-columns: repeat(3, auto); }
.grid-original {
display: grid;
grid-template-columns: repeat(7, auto);
gap: 20px;
justify-content: center;
grid-template-areas:
". . . b01 . . . "
". . . b02 . . . "
". . b03 b04 b05 . . "
"b06 b07 b08 . b09 b10 b11"
". . b12 b13 b14 . . "
". . . b15 . . . "
". . . b16 . . . ";
}
.boardmat {
background: transparent;
padding: 0;
border: none;
position: relative;
}
.tic-tac-toe-board {
display: grid;
gap: 4px;
}
.board-size-3 { grid-template-columns: repeat(3, 35px); }
.cell {
background-color: var(--cell-color);
border: 1px solid #cbd5e1;
color: #1a202c;
border-radius: 4px;
display: flex;
align-items: center;
justify-content: center;
font-weight: bold;
cursor: pointer;
user-select: none;
transition: all 0.1s ease;
}
.cell:hover {
background-color:#cbd5e1;
border-color: #6cc0f7;
}
.board-size-3 .cell {
width: 35px;
height: 35px;
font-size: 1.2rem;
}
.cell.blocked {
background-color: transparent !important;
pointer-events: none;
cursor: default;
border: none;
}
.cell.player-X { color: var(--p-x-color); }
.cell.player-O { color: var(--p-o-color); }
.cell.winning-cell {
background-color: #103a20;
border-color: #2ecc71;
box-shadow: 0 0 10px #2ecc71;
}
.hidden { display: none !important; }
.hi-list {
display: flex;
flex-direction: column;
gap: 10px;
max-width: 400px;
margin: 20px auto;
text-align: left;
}
.hi-item {
background: #0d86a8ff;
border: 1px solid #cbd5e1;
padding: 15px;
border-radius: 6px;
display: flex;
justify-content: space-between;
align-items: center;
}
.hi-item-info {
display: flex;
flex-direction: column;
gap: 4px;
}
.hi-item-title {
font-weight: bold;
color: #6cc0f7;
}
.join-btn {
background-color: #2ecc71;
color: #0b0f19;
border: none;
padding: 8px 15px;
border-radius: 4px;
font-weight: bold;
cursor: pointer;
transition: transform 0.1s;
}
.join-btn:hover {
transform: scale(1.05);
}
#settings1 {
background-color: #f0f4f8;
border: none;
position: absolute;
top: 10px;
left: 10px;
cursor: pointer;
font-size: 32px;
}
#YouclickedSettings {
border: 1px solid #cbd5e1;
border-radius: 12px;
background-color: #ffffff;
padding: 25px;
width: 90%;
max-width: 320px;
box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
box-sizing: border-box;
}
#YouclickedSettings::backdrop {
background: rgba(11, 15, 25, 0.4);
backdrop-filter: blur(3px);
}
.settings-header {
display: flex;
justify-content: space-between;
align-items: center;
margin-bottom: 20px;
border-bottom: 1px solid #cbd5e1;
padding-bottom: 10px;
}
.settings-header h3 {
margin: 0;
font-size: 20px;
color: #0b0f19;
}
#closingTime {
background: none;
border: none;
cursor: pointer;
padding: 5px;
display: flex;
align-items: center;
justify-content: center;
border-radius: 50%;
transition: background-color 0.2s;
}#closingTime:hover {
background-color: rgba(0, 0, 0, 0.08);
}
.settings-row {
display: flex;
justify-content: space-between;
align-items: center;
margin-bottom: 15px;
}.settings-row label {
font-size: 18px;
font-weight: 500;
color: black;
}.colors {
box-sizing: border-box;
width: 45px;
height: 30px;
padding: 0;
border: none;
border-radius: 4px;
cursor: pointer;
background: none;
}
#san{
 font-size: 22px;
 font-family: sans-serif;
 color: #ff5e62;
 background-color: #6cc0f7;
 padding: 5px;
 margin: 2px;
 cursor: pointer;
 border: 1px solid #6cbff77a;
 border-radius: 4px;
 box-shadow: 0 0 10px #2ecc71;
}
#ser{
 font-size: 24px;
 font-family: serif;
 color: #ff5e62;
 background-color: #6cc0f7;
 padding: 5px;
 margin: 2px;
 cursor: pointer;
   border: 1px solid #6cbff77a;
 border-radius: 4px;
 box-shadow: none;
}
#symp{
 font-size: 18px;
}
#cc {
 font-size: 18px;
 background-color: #c5f0ecff;
 border: none;
 border-radius: 4px;
}
#cc:hover{
 cursor: pointer;
 background-color: #9ae5ddff
}
#hide {
 display: none;
 flex-direction: column;
}
.firstbtn{
  background-color: #6cc0f7;
  border: none;
  border-radius: 5px;
  font-size: 18px;
   box-shadow: none;
}
.firstbtn:hover{
  cursor: pointer;
  background-color: #457290ff;
  color: white;
  transform: translateY(-1px);
}
#local{
  margin: 20px;
  padding: 11px;
  width: 185px;
}
</style>
</head>
<body>
<!--There's probably random "playcode" here isn't there :(-->
<!--It's impossible to remove-->
<h1>TickeracToe</h1>
<div class="bigBox" id="pm">
<label id="greeting" style="font-weight: bold; display: block; margin-bottom: 8px;">Set your name (cannot be a duplicate)</label>
<input type="text" id="username-input" placeholder="Set a name">
<button onclick="newnames()">Save</button>
</div>
<button id="settings1"><svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#1f1f1f"><path d="m370-80-16-128q-13-5-24.5-12T307-235l-119 50L78-375l103-78q-1-7-1-13.5v-27q0-6.5 1-13.5L78-585l110-190 119 50q11-8 23-15t24-12l16-128h220l16 128q13 5 24.5 12t22.5 15l119-50 110 190-103 78q1 7 1 13.5v27q0 6.5-2 13.5l103 78-110 190-118-50q-11 8-23 15t-24 12L590-80H370Zm70-80h79l14-106q31-8 57.5-23.5T639-327l99 41 39-68-86-65q5-14 7-29.5t2-31.5q0-16-2-31.5t-7-29.5l86-65-39-68-99 42q-22-23-48.5-38.5T533-694l-13-106h-79l-14 106q-31 8-57.5 23.5T321-633l-99-41-39 68 86 64q-5 15-7 30t-2 32q0 16 2 31t7 30l-86 65 39 68 99-42q22 23 48.5 38.5T427-266l13 106Zm42-180q58 0 99-41t41-99q0-58-41-99t-99-41q-59 0-99.5 41T342-480q0 58 40.5 99t99.5 41Zm-2-140Z"/></svg></button>
<button id="exclamation" onclick="window.location.href='https://docs.google.com/document/d/1NM1T9zkJbtmVScAuqD2ns_DG-BGzUHTEeHTn1QtUOCQ/edit?usp=sharing';">!</button>
<div id="playername" class="stat">Salutations!</div>
<div id="status" class="stat">Start a game or join one</div>
<div id="choice">
<div class="gc">
<div class="mc">
<button class="t-btn" id="new" onclick="newG()">New Game</button>
</div>
<div class="mc">
<button class="t-btn" id="join" onclick="joinGame()">Join a Game</button>
</div>
<div class="mc"><button class="t-btn" id="continue" onclick="cgl()">Continue Game</button> </div>
</div>
<div class="mc" style="display: block;"><button class="t-btn" id="local">Local Game</button></div>
</div>
<div id="game-arena" class="hidden">
<div style="text-align: center; margin-bottom: 10px;">
<button class="t-btn" onclick="goBack()" style="max-width: 150px; padding: 6px 12px; font-size: 14px;">Main Menu</button>
</div>
<div id="game-id-display" style="text-align: center; font-family: monospace; font-size: 10px; margin-bottom: 10px; color: #6cc0f7;"></div>
<div id="HC" class="HC-container"></div>
</div>
<dialog id="YouclickedSettings">
<div class="settings-header">
 <h3>Settings</h3>
 <button id="closingTime" aria-label="Close Settings"> <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#1f1f1f">   <path d="m256-200-56-56 224-224-224-224 56-56 224 224 224-224 56 56-224 224 224 224-56 56-224-224-224 224Z"/> </svg>
 </button>
</div>
<div class="settings-row">
 <label for="cpb">Background Color</label>
 <input type="color" id="cpb" name="cpb" class="colors" value="#f0f4f8">
</div>
<div class="settings-row">
 <label for="cpu">Button Color</label>
 <input type="color" id="cpu" name="cpu" class="colors" value="#6cc0f7">
</div>
<div class="settings-row">
 <label for="bcu">Board Color</label>
 <input type="color" id="bcu" name="bcu" class="colors" value="#0d86a8ff">
</div>
<div class="settings-row">
 <label for="bxc">Cell Color</label>
 <input type="color" id="bxc" name="bxc" class="colors" value="#e2e8f0">
</div>
<div class="settings-row">
<p id="symp" style="font-weight: 500;">Symbol style</p>
<button id="san" onclick="Norm()">X O</button>
<button id="ser" onclick="seriffed()">X O</button>
</div>
<div class="settings-row">
  <button id="oFirst" class="firstbtn" onclick="of()">Place O First</button>
  <button id="xFirst" class="firstbtn" onclick="xf()">Place X First</button>
</div>
<div class="settings-row" style="justify-content:center;">
 <button id="cc" onclick="ppc()">Set Player Colors</button>
</div>
<div id = "hide">
<div class="settings-row">
 <label for="pc">Your Color</label>
 <input type="color" id="pc" name="pc" class="colors" value="#ff0000ff">
 </div>
<div class="settings-row">
 <label for="oc">Opponent's Color</label>
 <input type="color" id="oc" name="pc" class="colors" value="#001affff">
</div>
</div>
</dialog>
<!--More evil playcode here, I imagine (womp womp)-->
<script>
let superC = window.supabase.createClient('https://pqbamyrdxqzbubrchfbk.supabase.co', 'sb_publishable_MecuJPIkwtdJg4RlfQns5w_oCzSgoVT');
const nG = document.getElementById('choice');
const sPan = document.getElementById('status');
const papermarker = document.getElementById('game-arena');
const hyperContain = document.getElementById('HC');
let thisGame = null;
const settings0 = document.getElementById('settings1');
const dialogue = document.getElementById('YouclickedSettings');
const closing = document.getElementById('closingTime');
let symbolism = null;
let liveChannel = null;
let macroSize = 3;
let microSize = 3;
let playing = 'X';
let letterP = 'X';
let gAct = true;
let GState = [];
let totalCells = 0;
let movesMade = 0;
let scores = { X: 0, O: 0 };
let wonScore = new Set();
let wS = 5;
let userName = "Player 1";
let wSync = false;
let p1Name = "Waiting...";
let p2Name = "Waiting...";
let isMove = false;
let fonType = 1;
function ubc(color) {
const buttons = document.querySelectorAll('.t-btn, .join-btn');
buttons.forEach(btn => btn.style.backgroundColor = color);
} window.onload = function() {
  if (localStorage.getItem('playcode-badge-hidden') !== '1'){
    localStorage.setItem('playcode-badge-hidden', '1');
  }
    if (localStorage.getItem('playcode-free-banner-hidden') !== '1'){
    localStorage.setItem('playcode-free-banner-hidden', '1');
  }
let MyColor = localStorage.getItem('myColor');
   if (MyColor) document.getElementById('pc').value = MyColor;
  
   let OppColor = localStorage.getItem('oppColor');
   if (OppColor) document.getElementById('oc').value = OppColor;
   if (localStorage.getItem('useCustomColors') === 'true') {
       document.getElementById('hide').style.display = "flex";
       document.getElementById('cc').innerHTML = "Use Default Colors";
   } aPC();
 let blockStyle = localStorage.getItem('xAndO');
const storedName = localStorage.getItem('tesseractPlayerName');
const profileMenu = document.getElementById('pm');
if (storedName) {
 userName = storedName;profileMenu.classList.add('hidden');
} else {
 profileMenu.classList.remove('hidden');
}
document.getElementById('playername').innerText = `Salutations, ${userName}!`;
document.getElementById('username-input').value = userName;
let bigColor = localStorage.getItem('bgColor');
if (bigColor){
 document.body.style.backgroundColor = bigColor;
 document.getElementById('settings1').style.backgroundColor = bigColor;
 document.getElementById('cpb').value = bigColor;
}
let clickColor = localStorage.getItem('buColor');
if (clickColor){
 ubc(clickColor);
 document.getElementById('cpu').value = clickColor;
}
let boardColor = localStorage.getItem('boColor');
if (boardColor) {
   document.getElementById('bcu').value = boardColor;
   document.documentElement.style.setProperty('--b-color', boardColor);
}
let cellColor = localStorage.getItem('ceColor');
if (cellColor) {
   document.getElementById('bxc').value = cellColor;
   document.documentElement.style.setProperty('--cell-color', cellColor);
}
let savedFont = localStorage.getItem('sF');
 if (savedFont) {
   hyperContain.style.fontFamily = savedFont;
   if (savedFont === 'serif'){
     fonType = 2;
     document.getElementById('san').style.boxShadow = 'none';
     document.getElementById('ser').style.boxShadow = '0 0 10px #2ecc71';
   }
 }
 let orx = localStorage.getItem('oOrX');
 if (orx) {
  if (orx === 'O'){
    document.getElementById('oFirst').style.boxShadow = '0 0 10px #2ecc71';
    document.getElementById('xFirst').style.boxShadow = 'none';
  } else {
    document.getElementById('xFirst').style.boxShadow = '0 0 10px #2ecc71';
    document.getElementById('oFirst').style.boxShadow = 'none';
  }
 }
console.log("I see you clicked inspect.");
};
function of(){
  localStorage.setItem('oOrX', 'O');
  document.getElementById('oFirst').style.boxShadow = '0 0 10px #2ecc71';
 document.getElementById('xFirst').style.boxShadow = 'none';
 orx = 'O';
}
function xf(){
    localStorage.setItem('oOrX', 'X');
  document.getElementById('xFirst').style.boxShadow = '0 0 10px #2ecc71';
 document.getElementById('oFirst').style.boxShadow = 'none';
 orx = 'X';
}
function Norm(){
 hyperContain.style.fontFamily = 'sans-serif';
 localStorage.setItem('sF', 'sans-serif');
 document.getElementById('san').style.boxShadow = '0 0 10px #2ecc71';
     document.getElementById('ser').style.boxShadow = 'none';

}
function seriffed(){
 hyperContain.style.fontFamily = 'serif';
 localStorage.setItem('sF', 'serif');
 document.getElementById('ser').style.boxShadow = '0 0 10px #2ecc71';
     document.getElementById('san').style.boxShadow = 'none';
}
function ppc(){
 let open = document.getElementById('hide');
 if (open.style.display === "none" || open.style.display === "") {
     open.style.display = "flex";
     document.getElementById('cc').innerHTML = "Use Default Colors";
     localStorage.setItem('useCustomColors', 'true');
 } else {
     open.style.display = "none";
     document.getElementById('cc').innerHTML = "Set Player Colors";
     localStorage.setItem('useCustomColors', 'false');
 }
 aPC();
}
function newnames() {
const inputVal = document.getElementById('username-input').value.trim();
if(inputVal !== "") {userName = inputVal;localStorage.setItem('tesseractPlayerName', inputVal);document.getElementById('greeting').innerText = `Hello, ${userName}`;document.getElementById('pm').classList.add('hidden');document.getElementById('playername').innerText = `Salutations, ${userName}!`;
}}settings0.addEventListener('click', () => { dialogue.showModal(); });
closing.addEventListener('click', () => { dialogue.close(); });function newG(){
nG.innerHTML = `
 <div class="gc"> <div class="mc">   <button class="t-btn" onclick="sS(3, 3)">Tesseract Style</button>   <div>3x3 grid of boards</div> </div> <div class="mc">   <button class="t-btn" onclick="sS(4, 3)">Original Style</button>   <div>The one shaped like a 4-pointed star</div> </div>
 </div>`;
const bCol = localStorage.getItem('buColor');
if (bCol) ubc(bCol);
}function sS(macro, micro) {
nG.innerHTML = `
 <div style="text-align: center; font-size: 18px; margin-bottom: -10px;">Select Winning Score:</div>
 <div class="gc"> <div class="mc"><button class="t-btn" onclick="alright(${macro}, ${micro}, 5)">5 Points</button></div> <div class="mc"><button class="t-btn" onclick="alright(${macro}, ${micro}, 10)">10 Points</button></div> <div class="mc"><button class="t-btn" onclick="alright(${macro}, ${micro}, 'custom')">Custom</button></div>
 </div>`;
const bCol = localStorage.getItem('buColor');
if (bCol) ubc(bCol);
}async function alright(macro, micro, score) {
if (score === 'custom') {
 let customVal = prompt("Choose the score required to win:", "7");
 let parsed = parseInt(customVal);
 wS = (!isNaN(parsed) && parsed > 0) ? parsed : 5;
} else {
 wS = score;
}
const IMS = (macro === 4) ? 7 : macro;
const blankState = Array(IMS).fill(null).map(() => Array(IMS).fill(null).map(() => Array(micro).fill(null).map(() => Array(micro).fill(null)) ));
if (macro !== 4) {
 blankState[1][1][1][1] = { player: 'blocked', symbol: '' };
}
const { data, error } = await superC.from('games').insert([{
 p1_name: userName,
 p2_name: 'Waiting for player...',
 winning_score: wS,
 game_state: blankState,
 scores: { X: 0, O: 0 },
 current_player: 'X',
 current_sub_turn: 'X',
 game_active: true,
 moves_made: 0
}]).select().single();
if (error) return alert("Big error. Email me if you get this message: " + error.message);
thisGame = data.id;
symbolism = 'X';
aPC();
setupG(macro, micro, data);
stgs(thisGame);
}async function joinGame() {
sPan.innerText = "Finding games...";
const { data: openGames, error } = await superC.from('games').select('id, p1_name, winning_score, game_state').eq('game_active', true).eq('p2_name', 'Waiting for player...').order('created_at', { ascending: false }).limit(10);
if (error) { sPan.innerText = "Error loading games."; return console.error(error); }
if (!openGames || openGames.length === 0) {
 nG.innerHTML = ` <div style="text-align: center; margin-top: 20px;">   <h3>No open games found!</h3>   <p>Why not start a new one?</p>   <button class="t-btn" onclick="goBack()" style="margin-top:10px;">Back</button>  </div>`;
 sPan.innerText = "Nothing to see...";
const bCol = localStorage.getItem('buColor');
 if (bCol) ubc(bCol);
 return;
}
let htmlList = `<div style="text-align: center;"><p>Available Games</p></div><div class="hi-list">`;
openGames.forEach(game => {
 const isOriginal = game.game_state && game.game_state.length === 7;
 const gameColor = isOriginal ? "Original Style" : "Tesseract Style";htmlList += `  <div class="hi-item"> <div class="hi-item-info"><span class="hi-item-title">${game.p1_name}'s Game</span><span style="font-size: 0.85rem; color: #aaa;">  Target Score: ${game.winning_score} | Mode: <span style="color: #6cc0f7;">${gameColor}</span></span> </div> <button class="join-btn" onclick="jsg('${game.id}')">Join</button>  </div>`;
});
htmlList += `</div> <div style="text-align: center; margin-top: 15px;"><button class="t-btn" onclick="goBack()">Back to Menu</button> </div>`;
nG.innerHTML = htmlList;
sPan.innerText = "Select a game";
const bCol = localStorage.getItem('buColor');
if (bCol) ubc(bCol);
}async function jsg(gameId) {
sPan.innerText = "Connecting...";
thisGame = gameId;
symbolism = 'O';
aPC();
const { data, error } = await superC.from('games').update({ p2_name: userName }).eq('id', thisGame).select().single();
if (error || !data) { return alert("Can't join game session. The game is probably full. :("); }
p1Name = data.p1_name;
p2Name = data.p2_name;
wS = data.winning_score;
const inferredMacro = data.game_state.length === 7 ? 4 : data.game_state.length;
setupG(inferredMacro, 3, data);
stgs(thisGame);
}function stgs(id) {
if (!superC) return;
if(liveChannel) superC.removeChannel(liveChannel);
liveChannel = superC.channel(`room-${id}`) .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'games', filter: `id=eq.${id}` }, payload => { syncGameFields(payload.new); }) .subscribe();
}function syncGameFields(data) {
playing = data.current_player;
letterP = data.current_sub_turn;
gAct = data.game_active;
movesMade = data.moves_made;
scores = data.scores;
p1Name = data.p1_name;
p2Name = data.p2_name;
if (data.last_move) {
 const { w, z, y, x, symbol, p } = data.last_move;
 if (GState[w] && GState[w][z]) { GState[w][z][y][x] = { player: p, symbol: symbol }; const cell = document.querySelector(`[data-w="${w}"][data-z="${z}"][data-y="${y}"][data-x="${x}"]`); if(cell && !cell.innerText) {cell.innerText = symbol;cell.classList.add('player-' + p);YougetPointsfunc(w, z, y, x, false); }
 }}
updateStatusDisplay();
wSync = false;
}function goBack() {
if(liveChannel && superC) superC.removeChannel(liveChannel);
papermarker.classList.add('hidden');
nG.classList.remove('hidden');
thisGame = null;
symbolism = null;
p1Name = "Waiting...";
p2Name = "Waiting...";
nG.innerHTML = ` <div class="gc">  <div class="mc"><button class="t-btn" id="new" onclick="newG()">New Game</button>  </div>  <div class="mc"><button class="t-btn" id="join" onclick="joinGame()">Join a Game</button>  </div>  <div class="mc"><button class="t-btn" id="continue" onclick="cgl()">Continue Game</button>  </div> </div>`;
sPan.innerText = "Start a game or join one";
const bCol = localStorage.getItem('buColor');
if (bCol) ubc(bCol);
}function updateStatusDisplay() {
const p1DisplayName = p1Name;
const p2DisplayName = p2Name;
let turnString = "";
if (!gAct) {
 turnString = "Game Over!";
} else if (symbolism) {
 const isMyTurn = (playing === symbolism);
 turnString = isMyTurn ? `<span style="color: #2ecc71; font-weight:bold;">Your Turn! Place ${letterP}</span>` : `<span style="color: #aaa;">Waiting for opponent...</span>`;
} else {
 turnString = `Turn: ${playing === 'X' ? p1DisplayName : p2DisplayName} (${letterP})`;
}
sPan.innerHTML = `
 <div style="margin-bottom: 5px;"> <span style="color: var(--p-x-color); font-weight: bold;">${p1DisplayName}: ${scores.X} pts</span> &nbsp;|&nbsp; <span style="color: var(--p-o-color); font-weight: bold;">${p2DisplayName}: ${scores.O} pts</span> <div style="font-size: 10px; color: #aaa;">Get ${wS} points to win</div>
 </div>
 <div>${turnString}</div>`;
}function setupG(macro, micro, makeARow) {
const isOriginalStyle = (macro === 4);
macroSize = isOriginalStyle ? 7 : macro;
microSize = micro;
totalCells = 0;
wonScore.clear();
nG.classList.add('hidden');
papermarker.classList.remove('hidden');
hyperContain.className = 'HC-container ' + (isOriginalStyle ? 'grid-original' : 'grid-3d-3');
hyperContain.innerHTML = '';
const originalLayout = [
 [null, null, null, 'b01', null, null, null],
 [null, null, null, 'b02', null, null, null],
 [null, null, 'b03', 'b04', 'b05', null, null],
 ['b06', 'b07', 'b08', null, 'b09', 'b10', 'b11'],
 [null, null, 'b12', 'b13', 'b14', null, null],
 [null, null, null, 'b15', null, null, null],
 [null, null, null, 'b16', null, null, null]
];
GState = JSON.parse(JSON.stringify(makeARow.game_state));
for(let w = 0; w < macroSize; w++) {
 for(let z = 0; z < macroSize; z++) { if (isOriginalStyle) {   const areaName = originalLayout[w][z];   if (!areaName) continue;   const boardWrapper = document.createElement('div');   boardWrapper.className = 'boardmat';   boardWrapper.style.gridArea = areaName;   const board = document.createElement('div');   board.className = `tic-tac-toe-board board-size-3`;   for(let y = 0; y < microSize; y++) { for(let x = 0; x < microSize; x++) {   totalCells++;   const cell = document.createElement('div');   cell.className = 'cell';   cell.dataset.w = w;   cell.dataset.z = z;   cell.dataset.y = y;   cell.dataset.x = x;   const itemState = GState[w][z][y][x];   if(itemState) {cell.innerText = itemState.symbol;cell.classList.add('player-' + itemState.player);   }   cell.addEventListener('click', hcc);   board.appendChild(cell); }   }   boardWrapper.appendChild(board);   hyperContain.appendChild(boardWrapper); } else {   const boardWrapper = document.createElement('div');   boardWrapper.className = 'boardmat';   const board = document.createElement('div');   board.className = `tic-tac-toe-board board-size-3`;   for(let y = 0; y < microSize; y++) { for(let x = 0; x < microSize; x++) {   const cell = document.createElement('div');   cell.dataset.w = w;   cell.dataset.z = z;   cell.dataset.y = y;   cell.dataset.x = x;   const itemState = GState[w][z][y][x];   if (itemState && itemState.player === 'blocked') {cell.className = 'cell blocked';   } else {cell.className = 'cell';totalCells++;if(itemState) {  cell.innerText = itemState.symbol;  cell.classList.add('player-' + itemState.player);}cell.addEventListener('click', hcc);   }   board.appendChild(cell); }   }   boardWrapper.appendChild(board);   hyperContain.appendChild(boardWrapper); }
 }}
syncGameFields(makeARow);
const bCol = localStorage.getItem('buColor');
if (bCol) ubc(bCol);
}async function hcc(e) {
if (!superC) return alert("Error with da server");
if (isMove || wSync) return;
const cell = e.target;
const w = parseInt(cell.dataset.w);
const z = parseInt(cell.dataset.z);
const y = parseInt(cell.dataset.y);
const x = parseInt(cell.dataset.x);
if (!gAct || GState[w][z][y][x] !== null) return;
if (symbolism && playing !== symbolism) return;
isMove = true;
wSync = true;
try {
 const moreMovez = movesMade + 1;
 const nextSubTurn = (letterP === 'X') ? 'O' : 'X';
 let nextPlayer = playing;
 if (letterP === 'O') { nextPlayer = (playing === 'X' ? 'O' : 'X'); }
 GState[w][z][y][x] = { player: playing, symbol: letterP };
 cell.innerText = letterP;
 cell.classList.add('player-' + playing);
 YougetPointsfunc(w, z, y, x);
 let isFinished = (scores[playing] >= wS || moreMovez === totalCells);
 const { error } = await superC.from('games').update({  game_state: GState,  current_player: nextPlayer,  current_sub_turn: nextSubTurn,  moves_made: moreMovez,  scores: scores,  game_active: !isFinished,  last_move: { w, z, y, x, symbol: letterP, p: playing }
 }).eq('id', thisGame);
 if (error) { alert(error.message); }} catch (err) {
 console.error("There was an error:", err);
} finally {
 isMove = false;
}}function YougetPointsfunc(w, z, y, x, suss = true) {
const dirs = [];
for (let dw = -1; dw <= 1; dw++) {
 for (let dz = -1; dz <= 1; dz++) { for (let dy = -1; dy <= 1; dy++) {   for (let dx = -1; dx <= 1; dx++) { if (dw === 0 && dz === 0 && dy === 0 && dx === 0) continue; if (dw < 0 || (dw === 0 && dz < 0) || (dw === 0 && dz === 0 && dy < 0) || (dw === 0 && dz === 0 && dy === 0 && dx < 0)) {   continue; } dirs.push([dw, dz, dy, dx]);}}}}
for (let i = 0; i < dirs.length; i++) {
 const [dw, dz, dy, dx] = dirs[i];
 for (let oSt = -2; oSt <= 0; oSt++) { let linec = []; let valid = true; let targetP = null; let targetSymbol = null; for (let step = 0; step < 3; step++) {   let cw = w + (oSt + step) * dw;   let cz = z + (oSt + step) * dz;   let cy = y + (oSt + step) * dy;   let cx = x + (oSt + step) * dx;   if (!inBounds(cw, macroSize) || !inBounds(cz, macroSize) || !inBounds(cy, microSize) || !inBounds(cx, microSize)) { valid = false; break;   }   let BoardAge = GState[cw][cz];   if (!BoardAge) { valid = false; break;   }   let cellState = BoardAge[cy][cx];   if (!cellState) { valid = false; break;   }   if (step === 0) { targetP = cellState.player; targetSymbol = cellState.symbol;   } else { if (cellState.player !== targetP || cellState.symbol !== targetSymbol) {   valid = false;   break; }   }   linec.push({w: cw, z: cz, y: cy, x: cx});} if (valid) {   let sortedCells = [...linec].sort((a, b) => { if (a.w !== b.w) return a.w - b.w; if (a.z !== b.z) return a.z - b.z; if (a.y !== b.y) return a.y - b.y; return a.x - b.x;   });   let key = sortedCells.map(c => `${c.w},${c.z},${c.y},${c.x}`).join('|');   if (!wonScore.has(key)) { wonScore.add(key); if (suss){   scores[targetP]++; } showin(linec);   }}}}}
function inBounds(val, max) { return val >= 0 && val < max; }
function showin(cells) {
cells.forEach(c => {
 const el = document.querySelector(`[data-w="${c.w}"][data-z="${c.z}"][data-y="${c.y}"][data-x="${c.x}"]`);
 if (el) el.classList.add('winning-cell');
});
}async function cgl() {
sPan.innerText = "Finding games... ";
const { data: myGames, error } = await superC.from('games').select('id, p1_name, p2_name, winning_score, game_state').eq('game_active', true).or(`p1_name.eq."${userName}",p2_name.eq."${userName}"`).order('created_at', { ascending: false }).limit(10);
if (!myGames || myGames.length === 0) {
 nG.innerHTML = `  <div style="text-align: center; margin-top: 20px;"> <h3>"${userName}" hasn't started any games</h3> <p>Make sure your name matches the one used when starting the game.</p> <button class="t-btn" onclick="goBack()" style="margin-top:10px;">Back</button>  </div>`;
 sPan.innerText = "No Games Found";
 return;
}
let htmlList = `<div style="text-align: center;"><h3>Your Active Games</h3></div><div class="hi-list">`;
myGames.forEach(game => {
 const isOriginal = game.game_state && game.game_state.length === 7;
 const gameColor = isOriginal ? "Original Style" : "Tesseract Style";const opponent = game.p1_name === userName ? game.p2_name : game.p1_name;htmlList += `  <div class="hi-item"> <div class="hi-item-info"><span class="hi-item-title">vs. ${opponent}</span><span style="font-size: 0.85rem; color: #aaa;">  Target Score: ${game.winning_score} | Mode: <span style="color: #6cc0f7;">${gameColor}</span></span> </div> <button class="join-btn" style="background-color: #6cc0f7;" onclick="keepgaming('${game.id}')">Resume</button>  </div>`;
});
htmlList += `</div> <div style="text-align: center; margin-top: 15px;"><button class="t-btn" onclick="goBack()">Back to Menu</button> </div>`;
nG.innerHTML = htmlList;
sPan.innerText = "Select a game to resume";
const bCol = localStorage.getItem('buColor');
if (bCol) ubc(bCol);
}
async function keepgaming(gameId) {
sPan.innerText = "Reconnecting? Idk.";
thisGame = gameId;
const { data, error } = await superC.from('games').select('*').eq('id', thisGame).single();
if (error || !data) { return alert("It broke."); }
if (data.p1_name === userName) { symbolism = 'X'; } else { symbolism = 'O'; }
aPC();
p1Name = data.p1_name;
p2Name = data.p2_name;
wS = data.winning_score;
const inferredMacro = data.game_state.length === 7 ? 4 : data.game_state.length;
setupG(inferredMacro, 3, data);
stgs(thisGame);
}
const colorP = document.getElementById('cpb');
const colorB = document.getElementById('cpu');
const colorM = document.getElementById('bcu');
const colorL = document.getElementById('bxc');


colorP.addEventListener('input', (event) => {
const ColorC = event.target.value;
document.body.style.backgroundColor = ColorC;
const stg = document.getElementById('settings1');
stg.style.backgroundColor = ColorC;
localStorage.setItem('bgColor', ColorC); });


colorM.addEventListener('input', (event) => {
   const Ad = event.target.value;
   document.documentElement.style.setProperty('--b-color', Ad);
   localStorage.setItem('boColor', Ad);
});


colorL.addEventListener('input', (event) => {
   const D = event.target.value;
   document.documentElement.style.setProperty('--cell-color', D);
   localStorage.setItem('ceColor', D);
});


colorB.addEventListener('input', (event) => {
const ColorD = event.target.value;
ubc(ColorD);
localStorage.setItem('buColor', ColorD);
});
const colorPX = document.getElementById('pc');
const colorPO = document.getElementById('oc');


function aPC() {
   const customActive = localStorage.getItem('useCustomColors') === 'true';
  
   if (!customActive) {
       document.documentElement.style.setProperty('--p-x-color', '#ff0000');
       document.documentElement.style.setProperty('--p-o-color', '#001aff');
   } else {
       const myCol = colorPX.value;
       const oppCol = colorPO.value;
      
       if (symbolism === 'O') {
           document.documentElement.style.setProperty('--p-o-color', myCol);
           document.documentElement.style.setProperty('--p-x-color', oppCol);
       } else {
           document.documentElement.style.setProperty('--p-x-color', myCol);
           document.documentElement.style.setProperty('--p-o-color', oppCol);
       }
   }
}
colorPX.addEventListener('input', (event) => {
   localStorage.setItem('myColor', event.target.value);
   aPC();
});
colorPO.addEventListener('input', (event) => {
   localStorage.setItem('oppColor', event.target.value);
   aPC();
});
</script>
</body>
</html>

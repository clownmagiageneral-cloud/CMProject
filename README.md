<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">

<title>Drop Catch</title>

<style>

body{
margin:0;
background:#2c2c2c;
font-family:monospace;
text-align:center;
color:white;
}

#console{
width:380px;
margin:auto;
padding:15px;
background:#c8d6b9;
border:12px solid #333;
border-radius:20px;
color:black;
}

#lcd{
width:340px;
height:420px;
background:#9bbc0f;
border:6px solid #222;
margin:auto;
position:relative;
overflow:hidden;
}

#hud{
display:flex;
justify-content:space-between;
font-size:14px;
padding:4px;
}

#player{
position:absolute;
bottom:12px;
width:60px;
height:12px;
background:#0f380f;
}

.item{
position:absolute;
font-size:22px;
font-weight:bold;
color:#0f380f;
}

.blink{
animation:blink 0.4s infinite;
}

@keyframes blink{
0%{opacity:1}
50%{opacity:0}
100%{opacity:1}
}

.effect{
position:absolute;
font-size:26px;
color:#0f380f;
animation:flash .3s;
}

@keyframes flash{
0%{opacity:1}
100%{opacity:0}
}

.centerMsg{
position:absolute;
top:45%;
left:50%;
transform:translate(-50%,-50%);
font-size:26px;
font-weight:bold;
}

#overlay{
position:absolute;
width:100%;
height:100%;
background:rgba(155,188,15,0.95);
display:flex;
flex-direction:column;
justify-content:center;
align-items:center;
}

button{
padding:6px 14px;
margin:4px;
font-size:16px;
}

#controls{
margin-top:10px;
}

.ctrl{
font-size:24px;
padding:10px 30px;
}

#info{
max-width:380px;
margin:20px auto;
background:#eee;
color:black;
padding:15px;
border-radius:10px;
text-align:left;
font-size:14px;
line-height:1.6;
}

#links{
text-align:center;
margin-top:10px;
}

a{
color:#0077cc;
text-decoration:none;
}

</style>
</head>

<body>

<h2>DROP CATCH</h2>

<div id="console">

<div id="hud">
<div id="score">SCORE 0</div>
<div id="hiscore">HI 0</div>
<div id="level">LV 1</div>
</div>

<div id="lcd">

<div id="player"></div>

<div id="overlay">
<h3>PRESS START</h3>
<button onclick="start()">START</button>
</div>

</div>

<div id="controls">
<button class="ctrl" onclick="move(-1)">◀</button>
<button class="ctrl" onclick="move(1)">▶</button>
</div>

</div>

<div id="info">

<b>■ルール</b><br>
○ をキャッチするとスコア +1<br>
○ を取り逃すとゲームオーバー<br>
✖ を取るとゲームオーバー<br>
レベルが上がると落下頻度が増え、点滅する○や✖が登場します。<br><br>

<b>■操作</b><br>
PC : ← → キー<br>
スマホ : ◀ ▶ ボタン<br>

<div id="links">

<br>

<b>■LINK</b><br>

<a href="https://www.youtube.com/@ClownMagia" target="_blank">
YouTube @ClownMagia
</a>

<br>

<a href="https://x.com/Rinka_A_262PD_P" target="_blank">
X @Rinka_A_262PD_P
</a>

</div>

</div>

<script>

const lanes=[20,90,160,230,300]
const steps=[0,60,120,180,240,300,360]

let laneIndex=2

const lcd=document.getElementById("lcd")
const player=document.getElementById("player")

const scoreUI=document.getElementById("score")
const levelUI=document.getElementById("level")
const hiUI=document.getElementById("hiscore")

const overlay=document.getElementById("overlay")

let score=0
let level=1

let items=[]
let running=false

let spawnRate=2000
let fallInterval=550

let hi=localStorage.getItem("dropcatch_hi")||0
hiUI.innerText="HI "+hi

player.style.left=lanes[laneIndex]+"px"

function sound(freq,time){

const ctx=new (window.AudioContext||window.webkitAudioContext)()
const osc=ctx.createOscillator()
const gain=ctx.createGain()

osc.connect(gain)
gain.connect(ctx.destination)

osc.frequency.value=freq
osc.start()

gain.gain.exponentialRampToValueAtTime(0.0001,ctx.currentTime+time)

osc.stop(ctx.currentTime+time)

}

function start(){

overlay.style.display="none"

running=true

spawn()

setInterval(stepFall,fallInterval)

difficulty()

}

function move(dir){

laneIndex+=dir

if(laneIndex<0)laneIndex=0
if(laneIndex>lanes.length-1)laneIndex=lanes.length-1

player.style.left=lanes[laneIndex]+"px"

}

function spawn(){

if(!running)return

let type="normal"
let symbol="○"

if(level>=5 && Math.random()<0.2){
type="danger"
symbol="✖"

warning()
}

const item=document.createElement("div")

item.className="item"

if(level>=3 && Math.random()<0.3){
item.classList.add("blink")
}

item.innerText=symbol

let lane=Math.floor(Math.random()*lanes.length)

item.dataset.lane=lane
item.dataset.step=0
item.dataset.type=type

item.style.left=lanes[lane]+"px"
item.style.top=steps[0]+"px"

lcd.appendChild(item)

items.push(item)

setTimeout(spawn,spawnRate)

}

function stepFall(){

if(!running)return

items.forEach((it,i)=>{

let step=parseInt(it.dataset.step)+1

if(step>=steps.length){

let lane=parseInt(it.dataset.lane)

if(lane===laneIndex){

catchItem(it.dataset.type,it.style.left)

}else{

if(it.dataset.type!="danger") end()

}

it.remove()
items.splice(i,1)

}else{

it.dataset.step=step
it.style.top=steps[step]+"px"

}

})

}

function catchItem(type,left){

if(type=="danger"){
sound(120,0.4)
end()
return
}

score++
sound(700,0.1)

scoreUI.innerText="SCORE "+score

effect(left)

}

function effect(left){

let e=document.createElement("div")

e.className="effect"
e.innerText="★"

e.style.left=left
e.style.top="350px"

lcd.appendChild(e)

setTimeout(()=>e.remove(),300)

}

function warning(){

let w=document.createElement("div")

w.className="centerMsg"
w.innerText="!"

lcd.appendChild(w)

setTimeout(()=>w.remove(),400)

}

function difficulty(){

if(!running)return

level++

sound(1200,0.2)

spawnRate=Math.max(400,spawnRate-150)

levelUI.innerText="LV "+level

levelMsg()

setTimeout(difficulty,15000)

}

function levelMsg(){

let m=document.createElement("div")

m.className="centerMsg"
m.innerText="LEVEL UP!"

lcd.appendChild(m)

setTimeout(()=>m.remove(),800)

}

function comment(){

if(score<20) return "Nice Try!"
if(score<50) return "Good Catch!"
if(score<100) return "Great Skills!"
if(score<200) return "Amazing Reflexes!"
return "Drop Catch Master!"
}

function end(){

running=false

if(score>hi){
hi=score
localStorage.setItem("dropcatch_hi",hi)
}

overlay.style.display="flex"

overlay.innerHTML=`

<h3>GAME OVER</h3>

Score ${score}<br>
Lv ${level}<br>

<button onclick="location.reload()">RETRY</button>

<button onclick="tweet()">POST X</button>

`

}

function tweet(){

let text=
`Drop Catch Lv${level} Score${score}

${comment()}

みんなもプレイしてみよう！

Game Production by Rengoku Rinka

#ClownMagia`

window.open("https://twitter.com/intent/tweet?text="+encodeURIComponent(text))

}

document.addEventListener("keydown",e=>{

if(e.key==="ArrowLeft") move(-1)
if(e.key==="ArrowRight") move(1)

})

</script>

</body>
</html>

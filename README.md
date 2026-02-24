<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mobile Arena Game</title>
<style>
body { margin:0; overflow:hidden; background:#111; touch-action:none; }
canvas { display:block; margin:auto; background:#1f2a38; }

#ui {
position:fixed;
bottom:20px;
width:100%;
display:flex;
justify-content:space-between;
padding:0 20px;
box-sizing:border-box;
}

.btn {
width:70px;
height:70px;
border-radius:50%;
background:rgba(255,255,255,0.2);
color:white;
font-size:18px;
display:flex;
align-items:center;
justify-content:center;
}

#joystick {
position:fixed;
left:40px;
bottom:40px;
width:120px;
height:120px;
border-radius:50%;
background:rgba(255,255,255,0.1);
}
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="joystick"></div>

<div id="ui">
<div class="btn" id="shootBtn">🔥</div>
<div class="btn" id="superBtn">💥</div>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let player, bullets, enemies, particles;
let score=0, gameOver=false;
let joyX=0, joyY=0;

function init(type="balanced") {
player = {
x:canvas.width/2,
y:canvas.height/2,
size:25,
speed:type==="fast"?4:3,
hp:type==="tank"?150:100,
color:type==="fast"?"lime":type==="tank"?"orange":"cyan"
};
bullets=[];
enemies=[];
particles=[];
score=0;
gameOver=false;
}
init("balanced");

function spawnEnemy() {
enemies.push({
x:Math.random()*canvas.width,
y:-30,
size:20,
hp:30,
speed:1.5
});
}
setInterval(spawnEnemy,1500);

function spawnBoss() {
enemies.push({
x:canvas.width/2,
y:100,
size:80,
hp:500,
speed:0.8,
boss:true
});
}

function shoot() {
bullets.push({x:player.x,y:player.y,size:6,speed:6});
}

function superAttack() {
for(let i=0;i<16;i++){
bullets.push({
x:player.x,
y:player.y,
size:5,
speed:5,
angle:(Math.PI*2/16)*i
});
}
}

document.getElementById("shootBtn").ontouchstart=shoot;
document.getElementById("superBtn").ontouchstart=superAttack;

document.getElementById("joystick").ontouchmove=e=>{
let rect=e.target.getBoundingClientRect();
let touch=e.touches[0];
joyX=(touch.clientX-(rect.left+60))/60;
joyY=(touch.clientY-(rect.top+60))/60;
};

document.getElementById("joystick").ontouchend=()=>{
joyX=0; joyY=0;
};

function update(){
if(gameOver)return;

player.x+=joyX*player.speed;
player.y+=joyY*player.speed;

bullets.forEach(b=>{
if(b.angle!==undefined){
b.x+=Math.cos(b.angle)*b.speed;
b.y+=Math.sin(b.angle)*b.speed;
}else{
b.y-=b.speed;
}
});

enemies.forEach((e,ei)=>{
let dx=player.x-e.x;
let dy=player.y-e.y;
let dist=Math.sqrt(dx*dx+dy*dy);
e.x+=dx/dist*e.speed;
e.y+=dy/dist*e.speed;

bullets.forEach((b,bi)=>{
if(Math.abs(e.x-b.x)<e.size){
e.hp-=10;
bullets.splice(bi,1);
particles.push({x:e.x,y:e.y,life:20});
if(e.hp<=0){
if(e.boss) score+=500;
else score+=10;
enemies.splice(ei,1);
}
}
});

if(Math.abs(e.x-player.x)<e.size){
player.hp-=0.5;
if(player.hp<=0) gameOver=true;
}
});

if(score>300 && !enemies.some(e=>e.boss)){
spawnBoss();
}

particles.forEach((p,pi)=>{
p.life--;
if(p.life<=0) particles.splice(pi,1);
});
}

function draw(){
ctx.clearRect(0,0,canvas.width,canvas.height);

ctx.fillStyle=player.color;
ctx.beginPath();
ctx.arc(player.x,player.y,player.size,0,Math.PI*2);
ctx.fill();

ctx.fillStyle="yellow";
bullets.forEach(b=>{
ctx.fillRect(b.x,b.y,b.size,b.size);
});

enemies.forEach(e=>{
ctx.fillStyle=e.boss?"purple":"red";
ctx.beginPath();
ctx.arc(e.x,e.y,e.size,0,Math.PI*2);
ctx.fill();
});

particles.forEach(p=>{
ctx.fillStyle="orange";
ctx.fillRect(p.x,p.y,6,6);
});

ctx.fillStyle="white";
ctx.font="20px Arial";
ctx.fillText("HP: "+Math.floor(player.hp),20,30);
ctx.fillText("Score: "+score,20,60);

if(gameOver){
ctx.font="40px Arial";
ctx.fillText("GAME OVER",canvas.width/2-120,canvas.height/2);
}
}

function loop(){
update();
draw();
requestAnimationFrame(loop);
}
loop();
</script>
</body>
</html>

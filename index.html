<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Brasileirão - Mini Futebol</title>

<style>
body{
    margin:0;
    display:flex;
    justify-content:center;
    align-items:center;
    height:100vh;
    background:#1a1a1a;
    font-family:Arial, Helvetica, sans-serif;
}

canvas{
    background:#2e8b57;
    border:5px solid #fff;
}
</style>
</head>
<body>

<canvas id="game" width="900" height="500"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// Escolha dos times
const timeCasa = "Flamengo";
const timeVisitante = "Palmeiras";

// Cores dos uniformes
const casaCor = "#d40000";
const visitanteCor = "#0b7d35";

const jogador = {
    x:120,
    y:250,
    r:22,
    color:casaCor,
    speed:4
};

const cpu = {
    x:780,
    y:250,
    r:22,
    color:visitanteCor,
    speed:2.4
};

const bola = {
    x:450,
    y:250,
    r:10,
    vx:0,
    vy:0
};

let placarCasa = 0;
let placarVisitante = 0;

const teclas = {};

document.addEventListener("keydown",e=>teclas[e.key]=true);
document.addEventListener("keyup",e=>teclas[e.key]=false);

function moverJogador(){

    if(teclas["ArrowUp"]) jogador.y -= jogador.speed;
    if(teclas["ArrowDown"]) jogador.y += jogador.speed;
    if(teclas["ArrowLeft"]) jogador.x -= jogador.speed;
    if(teclas["ArrowRight"]) jogador.x += jogador.speed;

    jogador.x=Math.max(jogador.r,Math.min(canvas.width-jogador.r,jogador.x));
    jogador.y=Math.max(jogador.r,Math.min(canvas.height-jogador.r,jogador.y));
}

function moverCPU(){

    if(bola.x<cpu.x) cpu.x-=cpu.speed;
    if(bola.x>cpu.x) cpu.x+=cpu.speed;
    if(bola.y<cpu.y) cpu.y-=cpu.speed;
    if(bola.y>cpu.y) cpu.y+=cpu.speed;
}

function chute(personagem){

    const dx=bola.x-personagem.x;
    const dy=bola.y-personagem.y;

    const dist=Math.sqrt(dx*dx+dy*dy);

    if(dist<personagem.r+bola.r){
        bola.vx=dx*0.45;
        bola.vy=dy*0.45;
    }
}

function atualizarBola(){

    bola.x+=bola.vx;
    bola.y+=bola.vy;

    bola.vx*=0.985;
    bola.vy*=0.985;

    if(bola.y<bola.r || bola.y>canvas.height-bola.r){
        bola.vy*=-1;
    }

    if(bola.x<0){
        placarVisitante++;
        reiniciar();
    }

    if(bola.x>canvas.width){
        placarCasa++;
        reiniciar();
    }
}

function reiniciar(){

    jogador.x=120;
    jogador.y=250;

    cpu.x=780;
    cpu.y=250;

    bola.x=450;
    bola.y=250;
    bola.vx=0;
    bola.vy=0;
}

function desenharCampo(){

    ctx.fillStyle="#2e8b57";
    ctx.fillRect(0,0,canvas.width,canvas.height);

    ctx.strokeStyle="white";
    ctx.lineWidth=4;

    ctx.strokeRect(0,0,canvas.width,canvas.height);

    ctx.beginPath();
    ctx.moveTo(canvas.width/2,0);
    ctx.lineTo(canvas.width/2,canvas.height);
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(canvas.width/2,canvas.height/2,60,0,Math.PI*2);
    ctx.stroke();

    // Área esquerda
    ctx.strokeRect(0,160,90,180);

    // Área direita
    ctx.strokeRect(810,160,90,180);
}

function desenharJogador(p){

    ctx.beginPath();
    ctx.fillStyle=p.color;
    ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
    ctx.fill();

    ctx.fillStyle="white";
    ctx.font="bold 12px Arial";
    ctx.textAlign="center";
    ctx.fillText("10",p.x,p.y+4);
}

function desenharBola(){

    ctx.beginPath();
    ctx.fillStyle="white";
    ctx.arc(bola.x,bola.y,bola.r,0,Math.PI*2);
    ctx.fill();
}

function desenharPlacar(){

    ctx.fillStyle="white";
    ctx.font="22px Arial";

    ctx.textAlign="center";

    ctx.fillText(
        `${timeCasa} ${placarCasa} x ${placarVisitante} ${timeVisitante}`,
        canvas.width/2,
        35
    );
}

function loop(){

    moverJogador();
    moverCPU();

    chute(jogador);
    chute(cpu);

    atualizarBola();

    desenharCampo();
    desenharJogador(jogador);
    desenharJogador(cpu);
    desenharBola();
    desenharPlacar();

    requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>

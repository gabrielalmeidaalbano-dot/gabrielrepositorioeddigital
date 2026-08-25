<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Jogo de Futebol JS</title>
<style>
    * { box-sizing: border-box; }
    body {
        margin: 0;
        background: #111;
        color: white;
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
    }
    #hud {
        display: flex;
        gap: 30px;
        align-items: center;
        margin-bottom: 10px;
        background: #222;
        padding: 10px 25px;
        border-radius: 8px;
        box-shadow: 0 4px 10px rgba(0,0,0,0.5);
    }
    #placar { font-size: 28px; font-weight: bold; }
    #tempo { font-size: 20px; color: #ffca28; }
    canvas {
        background: #159447;
        border: 4px solid #fff;
        border-radius: 4px;
        box-shadow: 0 10px 20px rgba(0,0,0,0.6);
        max-width: 95vw;
    }
    p { color: #aaa; margin-top: 10px; }
</style>
</head>
<body>

<div id="hud">
    <div id="placar">Você 0 - 0 CPU</div>
    <div id="tempo">02:00</div>
</div>

<canvas id="campo" width="1000" height="600"></canvas>

<p>🔵 Mover: WASD / Setas | ⚽ Chutar: Espaço</p>

<script>
const canvas = document.getElementById("campo");
const ctx = canvas.getContext("2d");
const placarEl = document.getElementById("placar");
const tempoEl = document.getElementById("tempo");

const CONFIG = {
    atrito: 0.98,
    larguraGol: 160,
    topGol: 220,
    bottomGol: 380
};

const jogador = { x: 250, y: 300, vx: 0, vy: 0, raio: 22, vel: 5.5, cor: "#1683ff", nome: "Você" };
const cpu = { x: 750, y: 300, vx: 0, vy: 0, raio: 22, vel: 3.8, cor: "#ff3333", nome: "CPU" };
const bola = { x: 500, y: 300, vx: 0, vy: 0, raio: 12, massa: 0.5 };

let golsJogador = 0;
let golsCPU = 0;
let tempoRestante = 120;
let jogoAtivo = true;
let congelado = false;
const teclas = {};

document.addEventListener("keydown", e => {
    teclas[e.key.toLowerCase()] = true;
    if (e.code === "Space" && jogoAtivo && !congelado) {
        chutar(jogador);
        e.preventDefault();
    }
});

document.addEventListener("keyup", e => teclas[e.key.toLowerCase()] = false);

// Loop do Cronômetro
const timerInterval = setInterval(() => {
    if (!congelado && jogoAtivo) {
        tempoRestante--;
        const min = String(Math.floor(tempoRestante / 60)).padStart(2, '0');
        const seg = String(tempoRestante % 60).padStart(2, '0');
        tempoEl.textContent = `${min}:${seg}`;

        if (tempoRestante <= 0) {
            jogoAtivo = false;
            clearInterval(timerInterval);
        }
    }
}, 1000);

function distancia(a, b) {
    return Math.hypot(a.x - b.x, a.y - b.y);
}

function chutar(p) {
    if (distancia(p, bola) < p.raio + bola.raio + 15) {
        let dx = bola.x - p.x;
        let dy = bola.y - p.y;
        let dist = Math.hypot(dx, dy) || 1;
        
        bola.vx = (dx / dist) * 14;
        bola.vy = (dy / dist) * 14;
    }
}

function resolverColisaoEntidades(a, b) {
    let dx = b.x - a.x;
    let dy = b.y - a.y;
    let dist = Math.hypot(dx, dy);
    let minDist = a.raio + b.raio;

    if (dist < minDist) {
        let angulo = Math.atan2(dy, dx);
        let sobreposicao = minDist - dist;

        // Afastar entidades
        let taxaA = a === bola ? 1 : 0.5;
        let taxaB = b === bola ? 1 : 0.5;

        a.x -= Math.cos(angulo) * sobreposicao * taxaA;
        a.y -= Math.sin(angulo) * sobreposicao * taxaA;
        b.x += Math.cos(angulo) * sobreposicao * taxaB;
        b.y += Math.sin(angulo) * sobreposicao * taxaB;

        // Transferência de impulso simples para a bola
        if (b === bola) {
            b.vx = Math.cos(angulo) * (Math.hypot(a.vx, a.vy) + 2);
            b.vy = Math.sin(angulo) * (Math.hypot(a.vx, a.vy) + 2);
        }
    }
}

function moverJogador() {
    let dx = 0, dy = 0;
    if (teclas["w"] || teclas["arrowup"]) dy--;
    if (teclas["s"] || teclas["arrowdown"]) dy++;
    if (teclas["a"] || teclas["arrowleft"]) dx--;
    if (teclas["d"] || teclas["arrowright"]) dx++;

    if (dx !== 0 || dy !== 0) {
        let len = Math.hypot(dx, dy);
        jogador.vx = (dx / len) * jogador.vel;
        jogador.vy = (dy / len) * jogador.vel;
    } else {
        jogador.vx = 0;
        jogador.vy = 0;
    }

    jogador.x += jogador.vx;
    jogador.y += jogador.vy;
    limitarPosicao(jogador);
}

function moverCPU() {
    let alvoX = bola.x;
    let alvoY = bola.y;

    // Se a bola estiver do lado do jogador, a CPU recua para defender
    if (bola.x < canvas.width / 2) {
        alvoX = 750;
        alvoY = Math.max(CONFIG.topGol, Math.min(CONFIG.bottomGol, bola.y));
    }

    let dx = alvoX - cpu.x;
    let dy = alvoY - cpu.y;
    let dist = Math.hypot(dx, dy);

    if (dist > 5) {
        cpu.vx = (dx / dist) * cpu.vel;
        cpu.vy = (dy / dist) * cpu.vel;
        cpu.x += cpu.vx;
        cpu.y += cpu.vy;
    }

    limitarPosicao(cpu);

    // CPU Chuta
    if (distancia(cpu, bola) < cpu.raio + bola.raio + 10) {
        chutar(cpu);
    }
}

function limitarPosicao(p) {
    p.x = Math.max(p.raio, Math.min(canvas.width - p.raio, p.x));
    p.y = Math.max(p.raio, Math.min(canvas.height - p.raio, p.y));
}

function moverBola() {
    bola.x += bola.vx;
    bola.y += bola.vy;

    bola.vx *= CONFIG.atrito;
    bola.vy *= CONFIG.atrito;

    // Colisão com as traves superiores e inferiores do campo
    if (bola.y - bola.raio < 10 || bola.y + bola.raio > canvas.height - 10) {
        bola.vy *= -1;
        bola.y = bola.y - bola.raio < 10 ? 10 + bola.raio : canvas.height - 10 - bola.raio;
    }

    // Checar Gols
    if (bola.y > CONFIG.topGol && bola.y < CONFIG.bottomGol) {
        if (bola.x < 10) registrarGol("CPU");
        if (bola.x > canvas.width - 10) registrarGol("Jogador");
    } else {
        // Rebater nas paredes laterais (fora da área do gol)
        if (bola.x - bola.raio < 10) {
            bola.vx *= -1;
            bola.x = 10 + bola.raio;
        }
        if (bola.x + bola.raio > canvas.width - 10) {
            bola.vx *= -1;
            bola.x = canvas.width - 10 - bola.raio;
        }
    }
}

function registrarGol(autor) {
    if (congelado) return;
    congelado = true;

    if (autor === "Jogador") golsJogador++;
    else golsCPU++;

    placarEl.textContent = `Você ${golsJogador} - ${golsCPU} CPU`;

    setTimeout(() => {
        resetarPosicoes();
        congelado = false;
    }, 1500);
}

function resetarPosicoes() {
    jogador.x = 250; jogador.y = 300;
    cpu.x = 750; cpu.y = 300;
    bola.x = 500; bola.y = 300;
    bola.vx = 0; bola.vy = 0;
}

function desenharCampo() {
    ctx.fillStyle = "#159447";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.strokeStyle = "rgba(255, 255, 255, 0.7)";
    ctx.lineWidth = 4;

    // Linhas externas
    ctx.strokeRect(10, 10, canvas.width - 20, canvas.height - 20);

    // Linha de meio campo e círculo central
    ctx.beginPath();
    ctx.moveTo(canvas.width / 2, 10);
    ctx.lineTo(canvas.width / 2, canvas.height - 10);
    ctx.arc(canvas.width / 2, canvas.height / 2, 70, 0, Math.PI * 2);
    ctx.stroke();

    // Áreas
    ctx.strokeRect(10, 150, 120, 300);
    ctx.strokeRect(canvas.width - 130, 150, 120, 300);

    // Gols
    ctx.fillStyle = "rgba(255,255,255,0.2)";
    ctx.fillRect(0, CONFIG.topGol, 10, CONFIG.larguraGol);
    ctx.fillRect(canvas.width - 10, CONFIG.topGol, 10, CONFIG.larguraGol);
}

function desenharEntidade(e) {
    ctx.beginPath();
    ctx.arc(e.x, e.y, e.raio, 0, Math.PI * 2);
    ctx.fillStyle = e.cor;
    ctx.fill();
    ctx.strokeStyle = "#fff";
    ctx.lineWidth = 2;
    ctx.stroke();
    ctx.closePath();
}

function desenharFimDeJogo() {
    ctx.fillStyle = "rgba(0, 0, 0, 0.75)";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = "#fff";
    ctx.font = "48px Arial";
    ctx.textAlign = "center";

    let texto = "Empate!";
    if (golsJogador > golsCPU) texto = "Você Venceu! 🎉";
    if (golsCPU > golsJogador) texto = "CPU Venceu! 🤖";

    ctx.fillText("FIM DE JOGO", canvas.width / 2, 260);
    ctx.fillText(texto, canvas.width / 2, 340);
}

function loop() {
    desenharCampo();

    if (jogoAtivo) {
        if (!congelado) {
            moverJogador();
            moverCPU();
            moverBola();

            resolverColisaoEntidades(jogador, bola);
            resolverColisaoEntidades(cpu, bola);
            resolverColisaoEntidades(jogador, cpu);
        }

        desenharEntidade(jogador);
        desenharEntidade(cpu);
        desenharEntidade(bola);
    } else {
        desenharFimDeJogo();
    }

    requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Futebol JS - Edição Energia</title>
<style>
    * { box-sizing: border-box; user-select: none; }
    body {
        margin: 0;
        background: #0d1117;
        color: white;
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
        overflow: hidden;
    }
    #hud {
        display: flex;
        gap: 20px;
        align-items: center;
        margin-bottom: 10px;
        background: #161b22;
        padding: 10px 25px;
        border-radius: 12px;
        border: 1px solid #30363d;
        box-shadow: 0 4px 20px rgba(0,0,0,0.6);
    }
    #placar { font-size: 26px; font-weight: 800; }
    #tempo { font-size: 20px; color: #ffca28; font-weight: bold; }
    #bar-container {
        width: 120px;
        height: 14px;
        background: #333;
        border-radius: 7px;
        overflow: hidden;
        border: 1px solid #555;
    }
    #bar-energia {
        width: 100%;
        height: 100%;
        background: linear-gradient(90deg, #00c6ff, #0072ff);
        transition: width 0.1s linear;
    }
    canvas {
        background: #159447;
        border: 4px solid #fff;
        border-radius: 8px;
        box-shadow: 0 12px 30px rgba(0,0,0,0.7);
        max-width: 95vw;
        max-height: 70vh;
    }
    p { color: #8b949e; margin-top: 10px; font-size: 14px; }
</style>
</head>
<body>

<div id="hud">
    <div id="placar">Você 0 - 0 CPU</div>
    <div id="tempo">02:00</div>
    <div>
        <small style="display:block; font-size:10px; color:#aaa;">ENERGIA</small>
        <div id="bar-container"><div id="bar-energia"></div></div>
    </div>
</div>

<canvas id="campo" width="1000" height="600"></canvas>

<p>🔵 Mover: WASD / Setas | ⚽ Chutar: Espaço | 🔄 Reiniciar: R</p>

<script>
const canvas = document.getElementById("campo");
const ctx = canvas.getContext("2d");
const placarEl = document.getElementById("placar");
const tempoEl = document.getElementById("tempo");
const barEnergiaEl = document.getElementById("bar-energia");

const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
function tocarSom(freq, tipo, duracao) {
    if (audioCtx.state === 'suspended') audioCtx.resume();
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = tipo;
    osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
    gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duracao);
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + duracao);
}

const CONFIG = {
    atrito: 0.982,
    topGol: 220,
    bottomGol: 380,
    profundidadeGol: 30
};

let estadoJogo = "JOGANDO";
let textoGolAnim = "";

const jogador = { 
    x: 250, y: 300, vx: 0, vy: 0, raio: 22, vel: 5.5, cor: "#1683ff",
    energia: 100, maxEnergia: 100 
};
const cpu = { x: 750, y: 300, vx: 0, vy: 0, raio: 22, vel: 3.8, cor: "#ff3333" };
const bola = { x: 500, y: 300, vx: 0, vy: 0, raio: 12 };

let golsJogador = 0;
let golsCPU = 0;
let tempoRestante = 120;
let timerInterval = null;
const teclas = {};

document.addEventListener("keydown", e => {
    teclas[e.key.toLowerCase()] = true;
    if (e.code === "Space" && estadoJogo === "JOGANDO") {
        chutar(jogador);
        e.preventDefault();
    }
    if (e.key.toLowerCase() === "r") resetarPartidaCompleta();
});

document.addEventListener("keyup", e => teclas[e.key.toLowerCase()] = false);

function iniciarTimer() {
    if (timerInterval) clearInterval(timerInterval);
    timerInterval = setInterval(() => {
        if (estadoJogo === "JOGANDO") {
            tempoRestante--;
            const min = String(Math.floor(tempoRestante / 60)).padStart(2, '0');
            const seg = String(tempoRestante % 60).padStart(2, '0');
            tempoEl.textContent = `${min}:${seg}`;
            if (tempoRestante <= 0) estadoJogo = "FIM";
        }
    }, 1000);
}
iniciarTimer();

function resetarPartidaCompleta() {
    golsJogador = 0; golsCPU = 0;
    tempoRestante = 120;
    jogador.energia = 100;
    estadoJogo = "JOGANDO";
    atualizarPlacar();
    resetarPosicoes();
    iniciarTimer();
}

function distancia(a, b) { return Math.hypot(a.x - b.x, a.y - b.y); }

function chutar(p) {
    // Requer pelo menos 15 de energia para um chute forte
    if (p.energia >= 15 && distancia(p, bola) < p.raio + bola.raio + 15) {
        let dx = bola.x - p.x;
        let dy = bola.y - p.y;
        let dist = Math.hypot(dx, dy) || 1;
        bola.vx = (dx / dist) * 16;
        bola.vy = (dy / dist) * 16;
        p.energia -= 15;
        tocarSom(300, 'square', 0.15);
    }
}

function resolverColisoes() {
    [jogador, cpu].forEach(p => {
        let dx = bola.x - p.x;
        let dy = bola.y - p.y;
        let dist = Math.hypot(dx, dy);
        let minDist = p.raio + bola.raio;

        if (dist < minDist) {
            let angulo = Math.atan2(dy, dx);
            bola.x = p.x + Math.cos(angulo) * minDist;
            bola.y = p.y + Math.sin(angulo) * minDist;
            bola.vx = Math.cos(angulo) * (Math.hypot(p.vx, p.vy) + 3);
            bola.vy = Math.sin(angulo) * (Math.hypot(p.vx, p.vy) + 3);
            tocarSom(150, 'triangle', 0.08);
        }
    });
}

function moverJogador() {
    let dx = 0, dy = 0;
    if (teclas["w"] || teclas["arrowup"]) dy--;
    if (teclas["s"] || teclas["arrowdown"]) dy++;
    if (teclas["a"] || teclas["arrowleft"]) dx--;
    if (teclas["d"] || teclas["arrowright"]) dx++;

    // Regeneração constante de energia
    if (jogador.energia < jogador.maxEnergia) {
        jogador.energia = Math.min(jogador.maxEnergia, jogador.energia + 0.15);
    }

    if ((dx !== 0 || dy !== 0) && jogador.energia > 2) {
        let len = Math.hypot(dx, dy);
        jogador.vx = (dx / len) * jogador.vel;
        jogador.vy = (dy / len) * jogador.vel;
        jogador.energia -= 0.1; // Consumo de energia ao mover
    } else { 
        jogador.vx = 0; 
        jogador.vy = 0; 
    }

    jogador.x += jogador.vx;
    jogador.y += jogador.vy;
    limitarPosicao(jogador);
    
    barEnergiaEl.style.width = `${jogador.energia}%`;
}

function moverCPU() {
    let alvoX = bola.x, alvoY = bola.y;
    if (bola.x < canvas.width / 2) {
        alvoX = 780;
        alvoY = Math.max(CONFIG.topGol, Math.min(CONFIG.bottomGol, bola.y));
    }

    let dx = alvoX - cpu.x, dy = alvoY - cpu.y;
    let dist = Math.hypot(dx, dy);

    if (dist > 5) {
        cpu.vx = (dx / dist) * cpu.vel;
        cpu.vy = (dy / dist) * cpu.vel;
        cpu.x += cpu.vx; cpu.y += cpu.vy;
    }
    limitarPosicao(cpu);

    if (distancia(cpu, bola) < cpu.raio + bola.raio + 8) {
        let dxB = bola.x - cpu.x;
        let dyB = bola.y - cpu.y;
        let distB = Math.hypot(dxB, dyB) || 1;
        bola.vx = (dxB / distB) * 12;
        bola.vy = (dyB / distB) * 12;
    }
}

function limitarPosicao(p) {
    p.x = Math.max(p.raio, Math.min(canvas.width - p.raio, p.x));
    p.y = Math.max(p.raio, Math.min(canvas.height - p.raio, p.y));
}

function moverBola() {
    bola.x += bola.vx; bola.y += bola.vy;
    bola.vx *= CONFIG.atrito; bola.vy *= CONFIG.atrito;

    // GOL APENAS QUANDO A BOLA ENCOSTA NA PAREDE DO FUNDO DO GOL
    let dentroDoGolY = bola.y > CONFIG.topGol && bola.y < CONFIG.bottomGol;

    // Parede do fundo do gol da esquerda (Gol da CPU)
    if (dentroDoGolY && bola.x - bola.raio <= -CONFIG.profundidadeGol) {
        registrarGol("CPU");
        return;
    }
    
    // Parede do fundo do gol da direita (Gol do Jogador)
    if (dentroDoGolY && bola.x + bola.raio >= canvas.width + CONFIG.profundidadeGol) {
        registrarGol("Jogador");
        return;
    }

    // Colisão com as traves (paredes superiores/inferiores da rede)
    if ((bola.x < 10 || bola.x > canvas.width - 10) && dentroDoGolY) {
        if (bola.y - bola.raio <= CONFIG.topGol || bola.y + bola.raio >= CONFIG.bottomGol) {
            bola.vy *= -1;
        }
    } else {
        // Colisão com teto e chão do campo
        if (bola.y - bola.raio < 10 || bola.y + bola.raio > canvas.height - 10) {
            bola.vy *= -1;
            tocarSom(200, 'sine', 0.05);
        }
        // Paredes laterais fora da área do gol
        if (bola.x - bola.raio < 10 || bola.x + bola.raio > canvas.width - 10) {
            bola.vx *= -1;
            tocarSom(200, 'sine', 0.05);
        }
    }
}

function registrarGol(autor) {
    if (estadoJogo !== "JOGANDO") return;
    estadoJogo = "GOL";
    tocarSom(600, 'sawtooth', 0.4);

    if (autor === "Jogador") { golsJogador++; textoGolAnim = "GOL SEU! ⚽"; }
    else { golsCPU++; textoGolAnim = "GOL DA CPU! 🤖"; }

    atualizarPlacar();
    setTimeout(() => {
        resetarPosicoes();
        if (tempoRestante > 0) estadoJogo = "JOGANDO";
    }, 2000);
}

function atualizarPlacar() {
    placarEl.textContent = `Você ${golsJogador} - ${golsCPU} CPU`;
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
    ctx.strokeRect(10, 10, canvas.width - 20, canvas.height - 20);

    // Linha de meio campo e círculo
    ctx.beginPath();
    ctx.moveTo(canvas.width / 2, 10); ctx.lineTo(canvas.width / 2, canvas.height - 10);
    ctx.arc(canvas.width / 2, canvas.height / 2, 70, 0, Math.PI * 2);
    ctx.stroke();

    // Áreas
    ctx.strokeRect(10, 150, 120, 300);
    ctx.strokeRect(canvas.width - 130, 150, 120, 300);

    // Gols Recuados (Rede)
    ctx.fillStyle = "rgba(0,0,0,0.2)";
    ctx.fillRect(-CONFIG.profundidadeGol, CONFIG.topGol, CONFIG.profundidadeGol, 160);
    ctx.fillRect(canvas.width, CONFIG.topGol, CONFIG.profundidadeGol, 160);
    
    ctx.strokeRect(-CONFIG.profundidadeGol, CONFIG.topGol, CONFIG.profundidadeGol, 160);
    ctx.strokeRect(canvas.width, CONFIG.topGol, CONFIG.profundidadeGol, 160);
}

function desenharEntidades() {
    // Bola
    ctx.beginPath();
    ctx.arc(bola.x, bola.y, bola.raio, 0, Math.PI * 2);
    ctx.fillStyle = "#fff"; ctx.fill();
    ctx.strokeStyle = "#000"; ctx.lineWidth = 2; ctx.stroke();

    // Jogadores
    [jogador, cpu].forEach(p => {
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.raio, 0, Math.PI * 2);
        ctx.fillStyle = p.cor; ctx.fill();
        ctx.strokeStyle = "#fff"; ctx.lineWidth = 3; ctx.stroke();
    });
}

function desenharOverlays() {
    ctx.textAlign = "center";
    if (estadoJogo === "GOL") {
        ctx.fillStyle = "rgba(0,0,0,0.4)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "#ffca28"; ctx.font = "bold 60px Arial";
        ctx.fillText(textoGolAnim, canvas.width / 2, 310);
    } else if (estadoJogo === "FIM") {
        ctx.fillStyle = "rgba(0,0,0,0.8)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "#fff"; ctx.font = "bold 48px Arial";
        let res = golsJogador > golsCPU ? "Você Venceu! 🎉" : (golsCPU > golsJogador ? "CPU Venceu! 🤖" : "Empate!");
        ctx.fillText("FIM DE JOGO", canvas.width / 2, 250);
        ctx.fillText(res, canvas.width / 2, 320);
        ctx.font = "18px Arial"; ctx.fillStyle = "#aaa";
        ctx.fillText("Pressione R para jogar novamente", canvas.width / 2, 380);
    }
}

function loop() {
    desenharCampo();

    if (estadoJogo === "JOGANDO") {
        moverJogador();
        moverCPU();
        moverBola();
        resolverColisoes();
    }

    desenharEntidades();
    desenharOverlays();
    requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>
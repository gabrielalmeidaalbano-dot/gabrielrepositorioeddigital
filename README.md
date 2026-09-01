<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Futebol JS - 2 Jogadores Locais</title>
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
    #placar { font-size: 24px; font-weight: 800; }
    #tempo { font-size: 20px; color: #ffca28; font-weight: bold; }
    #modo-hud { font-size: 13px; color: #8b949e; text-transform: uppercase; font-weight: bold; }
    .bar-wrapper { display: flex; flex-direction: column; align-items: center; }
    .bar-container {
        width: 100px;
        height: 12px;
        background: #333;
        border-radius: 6px;
        overflow: hidden;
        border: 1px solid #555;
    }
    .bar-energia { width: 100%; height: 100%; }
    #bar-p1 { background: linear-gradient(90deg, #00c6ff, #0072ff); }
    #bar-p2 { background: linear-gradient(90deg, #ff4e50, #f9d423); }
    #canvas-container { position: relative; }
    canvas {
        background: #159447;
        border: 4px solid #fff;
        border-radius: 8px;
        box-shadow: 0 12px 30px rgba(0,0,0,0.7);
        max-width: 95vw;
        max-height: 70vh;
        display: block;
    }
    #menu-principal {
        position: absolute;
        top: 0; left: 0; width: 100%; height: 100%;
        background: rgba(13, 17, 23, 0.95);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 15px;
        border-radius: 8px;
        z-index: 10;
    }
    #menu-principal h2 { font-size: 32px; margin: 0 0 10px 0; color: #fff; }
    .btn-opcao {
        width: 240px;
        padding: 12px;
        font-size: 18px;
        font-weight: bold;
        color: white;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        transition: transform 0.1s, background 0.2s;
    }
    .btn-opcao:hover { transform: scale(1.05); }
    .btn-cpu { background: #2ea44f; }
    .btn-pvp { background: #0969da; }
    .btn-penaltis { background: #d97706; }
    p { color: #8b949e; margin-top: 10px; font-size: 13px; text-align: center; }
</style>
</head>
<body>

<div id="hud">
    <div class="bar-wrapper">
        <small style="font-size:10px; color:#1683ff;">P1 ENERGIA</small>
        <div class="bar-container"><div id="bar-p1" class="bar-energia"></div></div>
    </div>
    <div id="placar">P1 0 - 0 P2</div>
    <div id="tempo">02:00</div>
    <div id="modo-hud">VS CPU</div>
    <div class="bar-wrapper">
        <small style="font-size:10px; color:#ff3333;">P2 ENERGIA</small>
        <div class="bar-container"><div id="bar-p2" class="bar-energia"></div></div>
    </div>
</div>

<div id="canvas-container">
    <canvas id="campo" width="1000" height="600"></canvas>
    
    <div id="menu-principal">
        <h2>Selecione o Modo</h2>
        <button class="btn-opcao btn-cpu" onclick="iniciarJogo('CPU')">1 Jogador (vs CPU)</button>
        <button class="btn-opcao btn-pvp" onclick="iniciarJogo('PVP')">2 Jogadores (Local)</button>
        <button class="btn-opcao btn-penaltis" onclick="iniciarJogo('PENALTIS')">Disputa de Pênaltis</button>
    </div>
</div>

<p>🔵 <b>P1:</b> WASD + Espaço (Pênaltis: 1-Esq, 2-Meio, 3-Dir) | 🔴 <b>P2:</b> Setas + Enter/Shift | 🔄 <b>R:</b> Menu</p>

<script>
const canvas = document.getElementById("campo");
const ctx = canvas.getContext("2d");
const placarEl = document.getElementById("placar");
const tempoEl = document.getElementById("tempo");
const barP1El = document.getElementById("bar-p1");
const barP2El = document.getElementById("bar-p2");
const modoHudEl = document.getElementById("modo-hud");
const menuEl = document.getElementById("menu-principal");

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
    atritoBola: 0.985,
    atritoJogador: 0.88,
    aceleracao: 0.8,
    topGol: 210,
    bottomGol: 390,
    larguraGol: 180,
    profundidadeGol: 40
};

let estadoJogo = "MENU";
let modoJogo = "CPU";
let textoGolAnim = "";

// Sistema de Partículas
let particulasGol = [];

function CriarParticula(x, y, cor) {
    return {
        x: x, y: y,
        vx: (Math.random() - 0.5) * 8,
        vy: (Math.random() - 0.5) * 8,
        raio: Math.random() * 4 + 2,
        cor: cor,
        vida: 1.0,
        decaiVida: Math.random() * 0.02 + 0.01
    };
}

function DispararExplosaoGol(autor) {
    let x, cor;
    let yVal = bola.y;

    if (autor === "P1") {
        x = canvas.width - 10;
        cor = p1.cor;
    } else {
        x = 10;
        cor = p2.cor;
    }

    for (let i = 0; i < 50; i++) {
        particulasGol.push(CriarParticula(x, yVal, cor));
    }
}

function atualizarParticulas() {
    for (let i = particulasGol.length - 1; i >= 0; i--) {
        let p = particulasGol[i];
        p.x += p.vx;
        p.y += p.vy;
        p.vida -= p.decaiVida;
        if (p.vida <= 0) particulasGol.splice(i, 1);
    }
}

function desenharParticulas() {
    particulasGol.forEach(p => {
        ctx.globalAlpha = p.vida;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.raio, 0, Math.PI * 2);
        ctx.fillStyle = p.cor;
        ctx.fill();
        ctx.globalAlpha = 1.0;
    });
}

const p1 = { 
    x: 250, y: 300, vx: 0, vy: 0, raio: 22, velMax: 5.5, cor: "#1683ff",
    energia: 100, maxEnergia: 100, nome: "P1" 
};
const p2 = { 
    x: 750, y: 300, vx: 0, vy: 0, raio: 22, velMax: 5.5, cor: "#ff3333",
    energia: 100, maxEnergia: 100, nome: "P2" 
};

const bola = { x: 500, y: 300, vx: 0, vy: 0, raio: 12 };

let golsP1 = 0;
let golsP2 = 0;
let tempoRestante = 120;
let timerInterval = null;
const teclas = {};

// Variáveis da Disputa de Pênaltis
let penaltisRodada = 1;
let penaltisTurno = "P1_CHUTA"; // "P1_CHUTA" ou "CPU_CHUTA"
let mensagemPenalti = "";

document.addEventListener("keydown", e => {
    teclas[e.key.toLowerCase()] = true;
    teclas[e.code] = true;
    
    if (estadoJogo === "JOGANDO") {
        if (modoJogo !== "PENALTIS") {
            if (e.code === "Space") { chutar(p1); e.preventDefault(); }
            if (e.code === "Enter" || e.code === "ShiftRight") { chutar(p2); e.preventDefault(); }
        } else {
            if (["1", "2", "3"].includes(e.key)) {
                processarPenalti(parseInt(e.key));
            }
        }
    }
    
    if (e.key.toLowerCase() === "r") voltarAoMenu();
});

document.addEventListener("keyup", e => {
    teclas[e.key.toLowerCase()] = false;
    teclas[e.code] = false;
});

function iniciarJogo(modo) {
    modoJogo = modo;
    modoHudEl.textContent = modo === "CPU" ? "VS CPU" : (modo === "PVP" ? "2P LOCAL" : "PÊNALTIS");
    menuEl.style.display = "none";
    
    if (modo === "PENALTIS") {
        iniciarPenaltis();
    } else {
        iniciarPartida();
    }
}

function voltarAoMenu() {
    estadoJogo = "MENU";
    menuEl.style.display = "flex";
    if (timerInterval) clearInterval(timerInterval);
    tempoEl.textContent = "02:00";
    golsP1 = 0; golsP2 = 0;
    particulasGol = [];
    atualizarPlacar();
    resetarPosicoes();
}

function iniciarPartida() {
    golsP1 = 0; golsP2 = 0;
    tempoRestante = 120;
    p1.energia = 100; p2.energia = 100;
    p2.velMax = modoJogo === "CPU" ? 3.8 : 5.5;
    estadoJogo = "JOGANDO";
    atualizarPlacar();
    resetarPosicoes();
    
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

function iniciarPenaltis() {
    golsP1 = 0; golsP2 = 0;
    penaltisRodada = 1;
    penaltisTurno = "P1_CHUTA";
    estadoJogo = "JOGANDO";
    tempoEl.textContent = "R1 / 5";
    mensagemPenalti = "P1: Escolha a direção do chute (1-Esq, 2-Meio, 3-Dir)";
    atualizarPlacar();
    configurarPosicaoPenalti();
}

function configurarPosicaoPenalti() {
    if (penaltisTurno === "P1_CHUTA") {
        p1.x = 750; p1.y = 300;
        p2.x = 960; p2.y = 300; // Goleiro na linha
        bola.x = 800; bola.y = 300;
    } else {
        p2.x = 250; p2.y = 300;
        p1.x = 40; p1.y = 300; // Goleiro na linha
        bola.x = 200; bola.y = 300;
    }
    bola.vx = 0; bola.vy = 0;
}

function processarPenalti(escolhaJogador) {
    const direcoesY = { 1: 230, 2: 300, 3: 370 }; // Esq, Meio, Dir
    const escolhaCPU = Math.floor(Math.random() * 3) + 1;

    if (penaltisTurno === "P1_CHUTA") {
        let alvoY = direcoesY[escolhaJogador];
        let defesaY = direcoesY[escolhaCPU];

        p2.y = defesaY;
        bola.vx = 14;
        bola.vy = (alvoY - bola.y) * 0.2;

        if (escolhaJogador !== escolhaCPU) {
            registrarGol("P1");
        } else {
            tocarSom(150, 'triangle', 0.2);
            mensagemPenalti = "DEFENDEU O GOLEIRO!";
            avancarTurnoPenalti();
        }
    } else {
        let alvoY = direcoesY[escolhaCPU];
        let defesaY = direcoesY[escolhaJogador];

        p1.y = defesaY;
        bola.vx = -14;
        bola.vy = (alvoY - bola.y) * 0.2;

        if (escolhaJogador !== escolhaCPU) {
            registrarGol("P2");
        } else {
            tocarSom(150, 'triangle', 0.2);
            mensagemPenalti = "DEFESA! Você salvou!";
            avancarTurnoPenalti();
        }
    }
}

function avancarTurnoPenalti() {
    setTimeout(() => {
        if (penaltisTurno === "P1_CHUTA") {
            penaltisTurno = "CPU_CHUTA";
            mensagemPenalti = "Sua vez de defender! Escolha para onde pular (1, 2 ou 3)";
        } else {
            penaltisTurno = "P1_CHUTA";
            penaltisRodada++;
            if (penaltisRodada > 5) {
                estadoJogo = "FIM";
                return;
            }
            mensagemPenalti = "P1: Escolha a direção do chute (1, 2 ou 3)";
        }
        tempoEl.textContent = `R${penaltisRodada} / 5`;
        configurarPosicaoPenalti();
    }, 1500);
}

function distancia(a, b) { return Math.hypot(a.x - b.x, a.y - b.y); }

function chutar(p) {
    if (p.energia >= 10 && distancia(p, bola) < p.raio + bola.raio + 12) {
        let dx = bola.x - p.x;
        let dy = bola.y - p.y;
        let dist = Math.hypot(dx, dy) || 1;
        bola.vx = (dx / dist) * 16;
        bola.vy = (dy / dist) * 16;
        p.energia -= 10;
        tocarSom(300, 'square', 0.15);
    }
}

function resolverColisoes() {
    let dxP = p2.x - p1.x;
    let dyP = p2.y - p1.y;
    let distP = Math.hypot(dxP, dyP);
    if (distP < p1.raio + p2.raio) {
        let ang = Math.atan2(dyP, dxP);
        let sobreposicao = (p1.raio + p2.raio) - distP;
        p1.x -= Math.cos(ang) * sobreposicao * 0.5;
        p1.y -= Math.sin(ang) * sobreposicao * 0.5;
        p2.x += Math.cos(ang) * sobreposicao * 0.5;
        p2.y += Math.sin(ang) * sobreposicao * 0.5;
    }

    [p1, p2].forEach(p => {
        let dx = bola.x - p.x;
        let dy = bola.y - p.y;
        let dist = Math.hypot(dx, dy);
        let minDist = p.raio + bola.raio;

        if (dist < minDist) {
            let angulo = Math.atan2(dy, dx);
            bola.x = p.x + Math.cos(angulo) * minDist;
            bola.y = p.y + Math.sin(angulo) * minDist;
            
            let velImpacto = Math.hypot(p.vx, p.vy);
            bola.vx = Math.cos(angulo) * (velImpacto + 3);
            bola.vy = Math.sin(angulo) * (velImpacto + 3);
            tocarSom(150, 'triangle', 0.08);
        }
    });
}

function moverP1() {
    let inputX = 0, inputY = 0;
    if (teclas["w"]) inputY--;
    if (teclas["s"]) inputY++;
    if (teclas["a"]) inputX--;
    if (teclas["d"]) inputX++;

    if (inputX !== 0 && inputY !== 0) { inputX *= 0.7071; inputY *= 0.7071; }

    if ((inputX !== 0 || inputY !== 0) && p1.energia > 2) {
        p1.vx += inputX * CONFIG.aceleracao;
        p1.vy += inputY * CONFIG.aceleracao;
        p1.energia = Math.max(0, p1.energia - 0.08);
    } else {
        p1.energia = Math.min(p1.maxEnergia, p1.energia + 0.2);
    }

    let velAtual = Math.hypot(p1.vx, p1.vy);
    if (velAtual > p1.velMax) {
        p1.vx = (p1.vx / velAtual) * p1.velMax;
        p1.vy = (p1.vy / velAtual) * p1.velMax;
    }
    p1.vx *= CONFIG.atritoJogador; p1.vy *= CONFIG.atritoJogador;
    p1.x += p1.vx; p1.y += p1.vy;
    limitarPosicao(p1);
    barP1El.style.width = `${p1.energia}%`;
}

function moverP2Humano() {
    let inputX = 0, inputY = 0;
    if (teclas["ArrowUp"] || teclas["arrowup"]) inputY--;
    if (teclas["ArrowDown"] || teclas["arrowdown"]) inputY++;
    if (teclas["ArrowLeft"] || teclas["arrowleft"]) inputX--;
    if (teclas["ArrowRight"] || teclas["arrowright"]) inputX++;

    if (inputX !== 0 && inputY !== 0) { inputX *= 0.7071; inputY *= 0.7071; }

    if ((inputX !== 0 || inputY !== 0) && p2.energia > 2) {
        p2.vx += inputX * CONFIG.aceleracao;
        p2.vy += inputY * CONFIG.aceleracao;
        p2.energia = Math.max(0, p2.energia - 0.08);
    } else {
        p2.energia = Math.min(p2.maxEnergia, p2.energia + 0.2);
    }

    let velAtual = Math.hypot(p2.vx, p2.vy);
    if (velAtual > p2.velMax) {
        p2.vx = (p2.vx / velAtual) * p2.velMax;
        p2.vy = (p2.vy / velAtual) * p2.velMax;
    }
    p2.vx *= CONFIG.atritoJogador; p2.vy *= CONFIG.atritoJogador;
    p2.x += p2.vx; p2.y += p2.vy;
    limitarPosicao(p2);
    barP2El.style.width = `${p2.energia}%`;
}

function moverCPU() {
    let alvoX = bola.x, alvoY = bola.y;
    if (bola.x < canvas.width / 2) {
        alvoX = 750;
        alvoY = Math.max(CONFIG.topGol, Math.min(CONFIG.bottomGol, bola.y));
    }

    let dx = alvoX - p2.x, dy = alvoY - p2.y;
    let dist = Math.hypot(dx, dy);

    if (dist > 5) {
        p2.vx += (dx / dist) * (CONFIG.aceleracao * 0.7);
        p2.vy += (dy / dist) * (CONFIG.aceleracao * 0.7);
    }

    let velAtual = Math.hypot(p2.vx, p2.vy);
    if (velAtual > p2.velMax) {
        p2.vx = (p2.vx / velAtual) * p2.velMax;
        p2.vy = (p2.vy / velAtual) * p2.velMax;
    }
    p2.vx *= CONFIG.atritoJogador; p2.vy *= CONFIG.atritoJogador;
    p2.x += p2.vx; p2.y += p2.vy;
    limitarPosicao(p2);

    if (distancia(p2, bola) < p2.raio + bola.raio + 8) {
        let dxB = bola.x - p2.x, dyB = bola.y - p2.y;
        let distB = Math.hypot(dxB, dyB) || 1;
        bola.vx = (dxB / distB) * 12;
        bola.vy = (dyB / distB) * 12;
    }
    barP2El.style.width = `100%`;
}

function limitarPosicao(p) {
    p.x = Math.max(p.raio, Math.min(canvas.width - p.raio, p.x));
    p.y = Math.max(p.raio, Math.min(canvas.height - p.raio, p.y));
}

function moverBola() {
    bola.x += bola.vx; bola.y += bola.vy;
    bola.vx *= CONFIG.atritoBola; bola.vy *= CONFIG.atritoBola;

    let dentroDaBocaDoGol = bola.y > CONFIG.topGol && bola.y < CONFIG.bottomGol;

    if (dentroDaBocaDoGol && (bola.x + bola.raio < 10)) {
        registrarGol("P2");
        return;
    }
    
    if (dentroDaBocaDoGol && (bola.x - bola.raio > canvas.width - 10)) {
        registrarGol("P1");
        return;
    }

    const traves = [
        { x: 10, y: CONFIG.topGol }, { x: 10, y: CONFIG.bottomGol },
        { x: canvas.width - 10, y: CONFIG.topGol }, { x: canvas.width - 10, y: CONFIG.bottomGol }
    ];

    traves.forEach(trave => {
        let d = distancia(bola, trave);
        if (d < bola.raio + 6) {
            let ang = Math.atan2(bola.y - trave.y, bola.x - trave.x);
            bola.vx = Math.cos(ang) * 8;
            bola.vy = Math.sin(ang) * 8;
            tocarSom(400, 'square', 0.1);
        }
    });

    if (bola.x < 10 || bola.x > canvas.width - 10) {
        if (bola.y - bola.raio <= CONFIG.topGol || bola.y + bola.raio >= CONFIG.bottomGol) bola.vy *= -1;
        if (bola.x - bola.raio <= -CONFIG.profundidadeGol || bola.x + bola.raio >= canvas.width + CONFIG.profundidadeGol) bola.vx *= -1;
    } else {
        if (bola.y - bola.raio < 10 || bola.y + bola.raio > canvas.height - 10) {
            bola.vy *= -1;
            tocarSom(200, 'sine', 0.05);
        }
        if (!dentroDaBocaDoGol) {
            if (bola.x - bola.raio < 10 || bola.x + bola.raio > canvas.width - 10) {
                bola.vx *= -1;
                tocarSom(200, 'sine', 0.05);
            }
        }
    }
}

function registrarGol(autor) {
    if (estadoJogo !== "JOGANDO") return;
    estadoJogo = "GOL";
    tocarSom(600, 'sawtooth', 0.4);
    DispararExplosaoGol(autor);

    if (autor === "P1") { golsP1++; textoGolAnim = "GOL DO P1! ⚽"; }
    else { golsP2++; textoGolAnim = modoJogo === "CPU" ? "GOL DA CPU! 🤖" : "GOL DO P2! ⚽"; }

    atualizarPlacar();
    
    if (modoJogo === "PENALTIS") {
        avancarTurnoPenalti();
        setTimeout(() => {
            if (tempoRestante > 0 && estadoJogo !== "FIM") estadoJogo = "JOGANDO";
        }, 1500);
    } else {
        setTimeout(() => {
            resetarPosicoes();
            particulasGol = [];
            if (tempoRestante > 0) estadoJogo = "JOGANDO";
        }, 2000);
    }
}

function atualizarPlacar() {
    let nomeP2 = modoJogo === "CPU" ? "CPU" : "P2";
    placarEl.textContent = `P1 ${golsP1} - ${golsP2} ${nomeP2}`;
}

function resetarPosicoes() {
    p1.x = 250; p1.y = 300; p1.vx = 0; p1.vy = 0;
    p2.x = 750; p2.y = 300; p2.vx = 0; p2.vy = 0;
    bola.x = 500; bola.y = 300; bola.vx = 0; bola.vy = 0;
}

function desenharCampo() {
    ctx.fillStyle = "#159447";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.strokeStyle = "rgba(255, 255, 255, 0.7)";
    ctx.lineWidth = 4;
    ctx.strokeRect(10, 10, canvas.width - 20, canvas.height - 20);

    ctx.beginPath();
    ctx.moveTo(canvas.width / 2, 10); ctx.lineTo(canvas.width / 2, canvas.height - 10);
    ctx.arc(canvas.width / 2, canvas.height / 2, 70, 0, Math.PI * 2);
    ctx.stroke();

    ctx.strokeRect(10, 150, 120, 300);
    ctx.strokeRect(canvas.width - 130, 150, 120, 300);

    ctx.fillStyle = "rgba(255,255,255,0.15)";
    ctx.fillRect(-CONFIG.profundidadeGol, CONFIG.topGol, CONFIG.profundidadeGol, CONFIG.larguraGol);
    ctx.strokeRect(-CONFIG.profundidadeGol, CONFIG.topGol, CONFIG.profundidadeGol, CONFIG.larguraGol);

    ctx.fillRect(canvas.width, CONFIG.topGol, CONFIG.profundidadeGol, CONFIG.larguraGol);
    ctx.strokeRect(canvas.width, CONFIG.topGol, CONFIG.profundidadeGol, CONFIG.larguraGol);

    ctx.fillStyle = "#fff";
    [ [10, CONFIG.topGol], [10, CONFIG.bottomGol], [canvas.width - 10, CONFIG.topGol], [canvas.width - 10, CONFIG.bottomGol] ].forEach(t => {
        ctx.beginPath();
        ctx.arc(t[0], t[1], 6, 0, Math.PI * 2);
        ctx.fill();
    });
}

function desenharEntidades() {
    ctx.beginPath();
    ctx.arc(bola.x, bola.y, bola.raio, 0, Math.PI * 2);
    ctx.fillStyle = "#fff"; ctx.fill();
    ctx.strokeStyle = "#000"; ctx.lineWidth = 2; ctx.stroke();

    [p1, p2].forEach(p => {
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.raio, 0, Math.PI * 2);
        ctx.fillStyle = p.cor; ctx.fill();
        ctx.strokeStyle = "#fff"; ctx.lineWidth = 3; ctx.stroke();
    });
}

function desenharOverlays() {
    ctx.textAlign = "center";
    
    if (modoJogo === "PENALTIS" && estadoJogo === "JOGANDO") {
        ctx.fillStyle = "#ffca28"; ctx.font = "bold 20px Arial";
        ctx.fillText(mensagemPenalti, canvas.width / 2, 50);
    }

    if (estadoJogo === "GOL") {
        ctx.fillStyle = "rgba(0,0,0,0.4)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "#ffca28"; ctx.font = "bold 60px Arial";
        ctx.fillText(textoGolAnim, canvas.width / 2, 310);
    } else if (estadoJogo === "FIM") {
        ctx.fillStyle = "rgba(0,0,0,0.8)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "#fff"; ctx.font = "bold 48px Arial";
        
        let nomeP2 = modoJogo === "CPU" ? "CPU" : "P2";
        let res = golsP1 > golsP2 ? "Jogador 1 Venceu! 🎉" : (golsP2 > golsP1 ? `${nomeP2} Venceu! 🎉` : "Empate!");
        
        ctx.fillText("FIM DE JOGO", canvas.width / 2, 250);
        ctx.fillText(res, canvas.width / 2, 320);
        ctx.font = "18px Arial"; ctx.fillStyle = "#aaa";
        ctx.fillText("Pressione R para voltar ao menu", canvas.width / 2, 380);
    }
}

function loop() {
    desenharCampo();
    atualizarParticulas();
    desenharParticulas();

    if (estadoJogo === "JOGANDO" || estadoJogo === "GOL") {
        if (modoJogo !== "PENALTIS") {
            moverP1();
            if (modoJogo === "PVP") moverP2Humano();
            else moverCPU();
            moverBola();
            resolverColisoes();
        } else if (estadoJogo === "GOL") {
            moverBola();
        }
    }

    desenharEntidades();
    desenharOverlays();
    requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html><!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Futebol JS - 2 Jogadores Locais</title>
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
    #placar { font-size: 24px; font-weight: 800; }
    #tempo { font-size: 20px; color: #ffca28; font-weight: bold; }
    #modo-hud { font-size: 13px; color: #8b949e; text-transform: uppercase; font-weight: bold; }
    .bar-wrapper { display: flex; flex-direction: column; align-items: center; }
    .bar-container {
        width: 100px;
        height: 12px;
        background: #333;
        border-radius: 6px;
        overflow: hidden;
        border: 1px solid #555;
    }
    .bar-energia { width: 100%; height: 100%; }
    #bar-p1 { background: linear-gradient(90deg, #00c6ff, #0072ff); }
    #bar-p2 { background: linear-gradient(90deg, #ff4e50, #f9d423); }
    #canvas-container { position: relative; }
    canvas {
        background: #159447;
        border: 4px solid #fff;
        border-radius: 8px;
        box-shadow: 0 12px 30px rgba(0,0,0,0.7);
        max-width: 95vw;
        max-height: 70vh;
        display: block;
    }
    #menu-principal {
        position: absolute;
        top: 0; left: 0; width: 100%; height: 100%;
        background: rgba(13, 17, 23, 0.95);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 15px;
        border-radius: 8px;
        z-index: 10;
    }
    #menu-principal h2 { font-size: 32px; margin: 0 0 10px 0; color: #fff; }
    .btn-opcao {
        width: 240px;
        padding: 12px;
        font-size: 18px;
        font-weight: bold;
        color: white;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        transition: transform 0.1s, background 0.2s;
    }
    .btn-opcao:hover { transform: scale(1.05); }
    .btn-cpu { background: #2ea44f; }
    .btn-pvp { background: #0969da; }
    .btn-penaltis { background: #d97706; }
    p { color: #8b949e; margin-top: 10px; font-size: 13px; text-align: center; }
</style>
</head>
<body>

<div id="hud">
    <div class="bar-wrapper">
        <small style="font-size:10px; color:#1683ff;">P1 ENERGIA</small>
        <div class="bar-container"><div id="bar-p1" class="bar-energia"></div></div>
    </div>
    <div id="placar">P1 0 - 0 P2</div>
    <div id="tempo">02:00</div>
    <div id="modo-hud">VS CPU</div>
    <div class="bar-wrapper">
        <small style="font-size:10px; color:#ff3333;">P2 ENERGIA</small>
        <div class="bar-container"><div id="bar-p2" class="bar-energia"></div></div>
    </div>
</div>

<div id="canvas-container">
    <canvas id="campo" width="1000" height="600"></canvas>
    
    <div id="menu-principal">
        <h2>Selecione o Modo</h2>
        <button class="btn-opcao btn-cpu" onclick="iniciarJogo('CPU')">1 Jogador (vs CPU)</button>
        <button class="btn-opcao btn-pvp" onclick="iniciarJogo('PVP')">2 Jogadores (Local)</button>
        <button class="btn-opcao btn-penaltis" onclick="iniciarJogo('PENALTIS')">Disputa de Pênaltis</button>
    </div>
</div>

<p>🔵 <b>P1:</b> WASD + Espaço (Pênaltis: 1-Esq, 2-Meio, 3-Dir) | 🔴 <b>P2:</b> Setas + Enter/Shift | 🔄 <b>R:</b> Menu</p>

<script>
const canvas = document.getElementById("campo");
const ctx = canvas.getContext("2d");
const placarEl = document.getElementById("placar");
const tempoEl = document.getElementById("tempo");
const barP1El = document.getElementById("bar-p1");
const barP2El = document.getElementById("bar-p2");
const modoHudEl = document.getElementById("modo-hud");
const menuEl = document.getElementById("menu-principal");

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
    atritoBola: 0.985,
    atritoJogador: 0.88,
    aceleracao: 0.8,
    topGol: 210,
    bottomGol: 390,
    larguraGol: 180,
    profundidadeGol: 40
};

let estadoJogo = "MENU";
let modoJogo = "CPU";
let textoGolAnim = "";

// Sistema de Partículas
let particulasGol = [];

function CriarParticula(x, y, cor) {
    return {
        x: x, y: y,
        vx: (Math.random() - 0.5) * 8,
        vy: (Math.random() - 0.5) * 8,
        raio: Math.random() * 4 + 2,
        cor: cor,
        vida: 1.0,
        decaiVida: Math.random() * 0.02 + 0.01
    };
}

function DispararExplosaoGol(autor) {
    let x, cor;
    let yVal = bola.y;

    if (autor === "P1") {
        x = canvas.width - 10;
        cor = p1.cor;
    } else {
        x = 10;
        cor = p2.cor;
    }

    for (let i = 0; i < 50; i++) {
        particulasGol.push(CriarParticula(x, yVal, cor));
    }
}

function atualizarParticulas() {
    for (let i = particulasGol.length - 1; i >= 0; i--) {
        let p = particulasGol[i];
        p.x += p.vx;
        p.y += p.vy;
        p.vida -= p.decaiVida;
        if (p.vida <= 0) particulasGol.splice(i, 1);
    }
}

function desenharParticulas() {
    particulasGol.forEach(p => {
        ctx.globalAlpha = p.vida;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.raio, 0, Math.PI * 2);
        ctx.fillStyle = p.cor;
        ctx.fill();
        ctx.globalAlpha = 1.0;
    });
}

const p1 = { 
    x: 250, y: 300, vx: 0, vy: 0, raio: 22, velMax: 5.5, cor: "#1683ff",
    energia: 100, maxEnergia: 100, nome: "P1" 
};
const p2 = { 
    x: 750, y: 300, vx: 0, vy: 0, raio: 22, velMax: 5.5, cor: "#ff3333",
    energia: 100, maxEnergia: 100, nome: "P2" 
};

const bola = { x: 500, y: 300, vx: 0, vy: 0, raio: 12 };

let golsP1 = 0;
let golsP2 = 0;
let tempoRestante = 120;
let timerInterval = null;
const teclas = {};

// Variáveis da Disputa de Pênaltis
let penaltisRodada = 1;
let penaltisTurno = "P1_CHUTA"; // "P1_CHUTA" ou "CPU_CHUTA"
let mensagemPenalti = "";

document.addEventListener("keydown", e => {
    teclas[e.key.toLowerCase()] = true;
    teclas[e.code] = true;
    
    if (estadoJogo === "JOGANDO") {
        if (modoJogo !== "PENALTIS") {
            if (e.code === "Space") { chutar(p1); e.preventDefault(); }
            if (e.code === "Enter" || e.code === "ShiftRight") { chutar(p2); e.preventDefault(); }
        } else {
            if (["1", "2", "3"].includes(e.key)) {
                processarPenalti(parseInt(e.key));
            }
        }
    }
    
    if (e.key.toLowerCase() === "r") voltarAoMenu();
});

document.addEventListener("keyup", e => {
    teclas[e.key.toLowerCase()] = false;
    teclas[e.code] = false;
});

function iniciarJogo(modo) {
    modoJogo = modo;
    modoHudEl.textContent = modo === "CPU" ? "VS CPU" : (modo === "PVP" ? "2P LOCAL" : "PÊNALTIS");
    menuEl.style.display = "none";
    
    if (modo === "PENALTIS") {
        iniciarPenaltis();
    } else {
        iniciarPartida();
    }
}

function voltarAoMenu() {
    estadoJogo = "MENU";
    menuEl.style.display = "flex";
    if (timerInterval) clearInterval(timerInterval);
    tempoEl.textContent = "02:00";
    golsP1 = 0; golsP2 = 0;
    particulasGol = [];
    atualizarPlacar();
    resetarPosicoes();
}

function iniciarPartida() {
    golsP1 = 0; golsP2 = 0;
    tempoRestante = 120;
    p1.energia = 100; p2.energia = 100;
    p2.velMax = modoJogo === "CPU" ? 3.8 : 5.5;
    estadoJogo = "JOGANDO";
    atualizarPlacar();
    resetarPosicoes();
    
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

function iniciarPenaltis() {
    golsP1 = 0; golsP2 = 0;
    penaltisRodada = 1;
    penaltisTurno = "P1_CHUTA";
    estadoJogo = "JOGANDO";
    tempoEl.textContent = "R1 / 5";
    mensagemPenalti = "P1: Escolha a direção do chute (1-Esq, 2-Meio, 3-Dir)";
    atualizarPlacar();
    configurarPosicaoPenalti();
}

function configurarPosicaoPenalti() {
    if (penaltisTurno === "P1_CHUTA") {
        p1.x = 750; p1.y = 300;
        p2.x = 960; p2.y = 300; // Goleiro na linha
        bola.x = 800; bola.y = 300;
    } else {
        p2.x = 250; p2.y = 300;
        p1.x = 40; p1.y = 300; // Goleiro na linha
        bola.x = 200; bola.y = 300;
    }
    bola.vx = 0; bola.vy = 0;
}

function processarPenalti(escolhaJogador) {
    const direcoesY = { 1: 230, 2: 300, 3: 370 }; // Esq, Meio, Dir
    const escolhaCPU = Math.floor(Math.random() * 3) + 1;

    if (penaltisTurno === "P1_CHUTA") {
        let alvoY = direcoesY[escolhaJogador];
        let defesaY = direcoesY[escolhaCPU];

        p2.y = defesaY;
        bola.vx = 14;
        bola.vy = (alvoY - bola.y) * 0.2;

        if (escolhaJogador !== escolhaCPU) {
       

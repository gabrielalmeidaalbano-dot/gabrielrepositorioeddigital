<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Futebol JS - Seleção de Dificuldade</title>
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
    #dificuldade-hud { font-size: 14px; color: #8b949e; text-transform: uppercase; font-weight: bold; }
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
    }
    #canvas-container {
        position: relative;
    }
    canvas {
        background: #159447;
        border: 4px solid #fff;
        border-radius: 8px;
        box-shadow: 0 12px 30px rgba(0,0,0,0.7);
        max-width: 95vw;
        max-height: 70vh;
        display: block;
    }
    /* Overlay HTML para Menu de Dificuldade */
    #menu-dificuldade {
        position: absolute;
        top: 0; left: 0; width: 100%; height: 100%;
        background: rgba(13, 17, 23, 0.9);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 15px;
        border-radius: 8px;
        z-index: 10;
    }
    #menu-dificuldade h2 { font-size: 32px; margin: 0 0 10px 0; color: #fff; }
    .btn-dif {
        width: 200px;
        padding: 12px;
        font-size: 18px;
        font-weight: bold;
        color: white;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        transition: transform 0.1s, background 0.2s;
    }
    .btn-dif:hover { transform: scale(1.05); }
    .btn-facil { background: #2ea44f; }
    .btn-medio { background: #d97706; }
    .btn-dificil { background: #dc2626; }
    p { color: #8b949e; margin-top: 10px; font-size: 14px; }
</style>
</head>
<body>

<div id="hud">
    <div id="placar">Você 0 - 0 CPU</div>
    <div id="tempo">02:00</div>
    <div id="dificuldade-hud">MÉDIO</div>
    <div>
        <small style="display:block; font-size:10px; color:#aaa;">ENERGIA</small>
        <div id="bar-container"><div id="bar-energia"></div></div>
    </div>
</div>

<div id="canvas-container">
    <canvas id="campo" width="1000" height="600"></canvas>
    
    <div id="menu-dificuldade">
        <h2>Escolha a Dificuldade</h2>
        <button class="btn-dif btn-facil" onclick="selecionarDificuldade('FACIL')">1 - Fácil</button>
        <button class="btn-dif btn-medio" onclick="selecionarDificuldade('MEDIO')">2 - Médio</button>
        <button class="btn-dif btn-dificil" onclick="selecionarDificuldade('DIFICIL')">3 - Difícil</button>
    </div>
</div>

<p>🔵 Mover: WASD / Setas | ⚽ Chutar: Espaço | 🔄 Reiniciar Menu: R</p>

<script>
const canvas = document.getElementById("campo");
const ctx = canvas.getContext("2d");
const placarEl = document.getElementById("placar");
const tempoEl = document.getElementById("tempo");
const barEnergiaEl = document.getElementById("bar-energia");
const difHudEl = document.getElementById("dificuldade-hud");
const menuDifEl = document.getElementById("menu-dificuldade");

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

// Configurações por nível de dificuldade
const NIVEIS_DIFICULDADE = {
    FACIL: { cpuVel: 2.8, cpuChuteForce: 10, barreiraVel: 2.0, barreiraAltura: 60, nome: "Fácil" },
    MEDIO: { cpuVel: 3.8, cpuChuteForce: 12, barreiraVel: 3.2, barreiraAltura: 80, nome: "Médio" },
    DIFICIL: { cpuVel: 4.8, cpuChuteForce: 15, barreiraVel: 4.5, barreiraAltura: 100, nome: "Difícil" }
};

let dificuldadeAtual = NIVEIS_DIFICULDADE.MEDIO;
let estadoJogo = "MENU";
let textoGolAnim = "";

const jogador = { 
    x: 250, y: 300, vx: 0, vy: 0, raio: 22, velMax: 5.5, cor: "#1683ff",
    energia: 100, maxEnergia: 100 
};
const cpu = { x: 750, y: 300, vx: 0, vy: 0, raio: 22, velMax: 3.8, cor: "#ff3333" };

const barreiraCPU = {
    x: 930,
    y: 300,
    largura: 16,
    altura: 80,
    vel: 3.2,
    cor: "#888888"
};

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
    if (estadoJogo === "MENU") {
        if (e.key === "1") selecionarDificuldade('FACIL');
        if (e.key === "2") selecionarDificuldade('MEDIO');
        if (e.key === "3") selecionarDificuldade('DIFICIL');
    }
    if (e.key.toLowerCase() === "r") voltarAoMenu();
});

document.addEventListener("keyup", e => teclas[e.key.toLowerCase()] = false);

function selecionarDificuldade(nivelKey) {
    dificuldadeAtual = NIVEIS_DIFICULDADE[nivelKey];
    
    // Aplicar atributos da CPU e Barreira
    cpu.velMax = dificuldadeAtual.cpuVel;
    barreiraCPU.vel = dificuldadeAtual.barreiraVel;
    barreiraCPU.altura = dificuldadeAtual.barreiraAltura;
    difHudEl.textContent = dificuldadeAtual.nome;
    
    menuDifEl.style.display = "none";
    iniciarPartida();
}

function voltarAoMenu() {
    estadoJogo = "MENU";
    menuDifEl.style.display = "flex";
    if (timerInterval) clearInterval(timerInterval);
    tempoEl.textContent = "02:00";
    golsJogador = 0; golsCPU = 0;
    atualizarPlacar();
    resetarPosicoes();
}

function iniciarPartida() {
    golsJogador = 0; golsCPU = 0;
    tempoRestante = 120;
    jogador.energia = 100;
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

function moverBarreiraCPU() {
    let dy = bola.y - barreiraCPU.y;
    if (Math.abs(dy) > 5) {
        barreiraCPU.y += Math.sign(dy) * barreiraCPU.vel;
    }
    let meioAltura = barreiraCPU.altura / 2;
    barreiraCPU.y = Math.max(CONFIG.topGol + meioAltura, Math.min(CONFIG.bottomGol - meioAltura, barreiraCPU.y));
}

function resolverColisaoBarreira() {
    let bx = barreiraCPU.x - barreiraCPU.largura / 2;
    let by = barreiraCPU.y - barreiraCPU.altura / 2;
    let bw = barreiraCPU.largura;
    let bh = barreiraCPU.altura;

    let closestX = Math.max(bx, Math.min(bola.x, bx + bw));
    let closestY = Math.max(by, Math.min(bola.y, by + bh));

    let dx = bola.x - closestX;
    let dy = bola.y - closestY;
    let dist = Math.hypot(dx, dy);

    if (dist < bola.raio) {
        if (Math.abs(dx) > Math.abs(dy)) {
            bola.vx *= -0.8;
            bola.x = dx > 0 ? closestX + bola.raio : closestX - bola.raio;
        } else {
            bola.vy *= -0.8;
            bola.y = dy > 0 ? closestY + bola.raio : closestY - bola.raio;
        }
        tocarSom(250, 'square', 0.08);
    }
}

function resolverColisoes() {
    let dxP = cpu.x - jogador.x;
    let dyP = cpu.y - jogador.y;
    let distP = Math.hypot(dxP, dyP);
    if (distP < jogador.raio + cpu.raio) {
        let ang = Math.atan2(dyP, dxP);
        let sobreposicao = (jogador.raio + cpu.raio) - distP;
        jogador.x -= Math.cos(ang) * sobreposicao * 0.5;
        jogador.y -= Math.sin(ang) * sobreposicao * 0.5;
        cpu.x += Math.cos(ang) * sobreposicao * 0.5;
        cpu.y += Math.sin(ang) * sobreposicao * 0.5;
    }

    [jogador, cpu].forEach(p => {
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

    resolverColisaoBarreira();
}

function moverJogador() {
    let inputX = 0, inputY = 0;
    if (teclas["w"] || teclas["arrowup"]) inputY--;
    if (teclas["s"] || teclas["arrowdown"]) inputY++;
    if (teclas["a"] || teclas["arrowleft"]) inputX--;
    if (teclas["d"] || teclas["arrowright"]) inputX++;

    if (inputX !== 0 && inputY !== 0) {
        inputX *= 0.7071;
        inputY *= 0.7071;
    }

    if ((inputX !== 0 || inputY !== 0) && jogador.energia > 2) {
        jogador.vx += inputX * CONFIG.aceleracao;
        jogador.vy += inputY * CONFIG.aceleracao;
        jogador.energia = Math.max(0, jogador.energia - 0.08);
    } else {
        jogador.energia = Math.min(jogador.maxEnergia, jogador.energia + 0.2);
    }

    let velAtual = Math.hypot(jogador.vx, jogador.vy);
    if (velAtual > jogador.velMax) {
        jogador.vx = (jogador.vx / velAtual) * jogador.velMax;
        jogador.vy = (jogador.vy / velAtual) * jogador.velMax;
    }
    jogador.vx *= CONFIG.atritoJogador;
    jogador.vy *= CONFIG.atritoJogador;

    jogador.x += jogador.vx;
    jogador.y += jogador.vy;
    limitarPosicao(jogador);
    
    barEnergiaEl.style.width = `${jogador.energia}%`;
}

function moverCPU() {
    let alvoX = bola.x, alvoY = bola.y;
    if (bola.x < canvas.width / 2) {
        alvoX = 750;
        alvoY = Math.max(CONFIG.topGol, Math.min(CONFIG.bottomGol, bola.y));
    }

    let dx = alvoX - cpu.x, dy = alvoY - cpu.y;
    let dist = Math.hypot(dx, dy);

    if (dist > 5) {
        let dirX = (dx / dist);
        let dirY = (dy / dist);
        cpu.vx += dirX * (CONFIG.aceleracao * 0.7);
        cpu.vy += dirY * (CONFIG.aceleracao * 0.7);
    }

    let velAtual = Math.hypot(cpu.vx, cpu.vy);
    if (velAtual > cpu.velMax) {
        cpu.vx = (cpu.vx / velAtual) * cpu.velMax;
        cpu.vy = (cpu.vy / velAtual) * cpu.velMax;
    }
    cpu.vx *= CONFIG.atritoJogador;
    cpu.vy *= CONFIG.atritoJogador;

    cpu.x += cpu.vx; cpu.y += cpu.vy;
    limitarPosicao(cpu);

    if (distancia(cpu, bola) < cpu.raio + bola.raio + 8) {
        let dxB = bola.x - cpu.x;
        let dyB = bola.y - cpu.y;
        let distB = Math.hypot(dxB, dyB) || 1;
        bola.vx = (dxB / distB) * dificuldadeAtual.cpuChuteForce;
        bola.vy = (dyB / distB) * dificuldadeAtual.cpuChuteForce;
    }
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
        registrarGol("CPU");
        return;
    }
    
    if (dentroDaBocaDoGol && (bola.x - bola.raio > canvas.width - 10)) {
        registrarGol("Jogador");
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
        if (bola.y - bola.raio <= CONFIG.topGol || bola.y + bola.raio >= CONFIG.bottomGol) {
            bola.vy *= -1;
        }
        if (bola.x - bola.raio <= -CONFIG.profundidadeGol || bola.x + bola.raio >= canvas.width + CONFIG.profundidadeGol) {
            bola.vx *= -1;
        }
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

    if (autor === "Jogador") { golsJogador++; textoGolAnim = "GOLAAAAÇO! ⚽"; }
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
    jogador.x = 250; jogador.y = 300; jogador.vx = 0; jogador.vy = 0;
    cpu.x = 750; cpu.y = 300; cpu.vx = 0; cpu.vy = 0;
    barreiraCPU.y = 300;
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
    // Barreira da CPU
    ctx.fillStyle = barreiraCPU.cor;
    ctx.fillRect(
        barreiraCPU.x - barreiraCPU.largura / 2,
        barreiraCPU.y - barreiraCPU.altura / 2,
        barreiraCPU.largura,
        barreiraCPU.altura
    );
    ctx.strokeStyle = "#ffffff";
    ctx.lineWidth = 2;
    ctx.strokeRect(
        barreiraCPU.x - barreiraCPU.largura / 2,
        barreiraCPU.y - barreiraCPU.altura / 2,
        barreiraCPU.largura,
        barreiraCPU.altura
    );

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
        ctx.fillText("Pressione R para voltar ao menu", canvas.width / 2, 380);
    }
}

function loop() {
    desenharCampo();

    if (estadoJogo === "JOGANDO" || estadoJogo === "GOL") {
        moverJogador();
        moverCPU();
        moverBarreiraCPU();
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
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Jogo de Futebol</title>

<style>
    * {
        box-sizing: border-box;
    }

    body {
        margin: 0;
        background: #111;
        color: white;
        font-family: Arial, sans-serif;
        text-align: center;
    }

    h1 {
        margin: 15px 0 5px;
    }

    #placar {
        font-size: 24px;
        margin: 10px;
    }

    canvas {
        background: #159447;
        border: 5px solid white;
        max-width: 95vw;
        height: auto;
    }

    p {
        color: #ddd;
    }
</style>
</head>

<body>

<h1>⚽ Futebol</h1>
<div id="placar">Você 0 x 0 CPU</div>

<canvas id="campo" width="1000" height="600"></canvas>

<p>
    🔵 Mover: WASD / Setas &nbsp; | &nbsp;
    ⚽ Chutar: Espaço
</p>

<script>
const canvas = document.getElementById("campo");
const ctx = canvas.getContext("2d");
const placar = document.getElementById("placar");

const jogador = {
    x: 250,
    y: 300,
    raio: 22,
    velocidade: 5,
    cor: "#1683ff"
};

const cpu = {
    x: 750,
    y: 300,
    raio: 22,
    velocidade: 2.5,
    cor: "#ff3333"
};

const bola = {
    x: 500,
    y: 300,
    raio: 12,
    vx: 0,
    vy: 0
};

let golsJogador = 0;
let golsCPU = 0;

const teclas = {};

document.addEventListener("keydown", e => {
    teclas[e.key.toLowerCase()] = true;

    if (e.code === "Space") {
        chutar();
        e.preventDefault();
    }
});

document.addEventListener("keyup", e => {
    teclas[e.key.toLowerCase()] = false;
});

function distancia(a, b) {
    return Math.hypot(a.x - b.x, a.y - b.y);
}

function chutar() {
    const d = distancia(jogador, bola);

    if (d < 55) {
        let dx = bola.x - jogador.x;
        let dy = bola.y - jogador.y;

        const tamanho = Math.hypot(dx, dy) || 1;

        bola.vx = (dx / tamanho) * 11;
        bola.vy = (dy / tamanho) * 11;
    }
}

function moverJogador() {
    let dx = 0;
    let dy = 0;

    if (teclas["w"] || teclas["arrowup"]) dy--;
    if (teclas["s"] || teclas["arrowdown"]) dy++;
    if (teclas["a"] || teclas["arrowleft"]) dx--;
    if (teclas["d"] || teclas["arrowright"]) dx++;

    if (dx !== 0 || dy !== 0) {
        const tamanho = Math.hypot(dx, dy);

        dx /= tamanho;
        dy /= tamanho;

        jogador.x += dx * jogador.velocidade;
        jogador.y += dy * jogador.velocidade;
    }

    limitarJogador(jogador);
}

function limitarJogador(p) {
    p.x = Math.max(25, Math.min(canvas.width - 25, p.x));
    p.y = Math.max(25, Math.min(canvas.height - 25, p.y));
}

function moverCPU() {
    let dx = bola.x - cpu.x;
    let dy = bola.y - cpu.y;

    const tamanho = Math.hypot(dx, dy);

    if (tamanho > 5) {
        cpu.x += (dx / tamanho) * cpu.velocidade;
        cpu.y += (dy / tamanho) * cpu.velocidade;
    }

    limitarJogador(cpu);

    // CPU chuta quando chega perto da bola
    if (distancia(cpu, bola) < 45) {
        let direcaoX = -1;
        let direcaoY = (300 - bola.y) / 300;

        const tamanhoDir = Math.hypot(direcaoX, direcaoY);

        bola.vx = (direcaoX / tamanhoDir) * 8;
        bola.vy = (direcaoY / tamanhoDir) * 8;
    }
}

function moverBola() {
    bola.x += bola.vx;
    bola.y += bola.vy;

    // Atrito
    bola.vx *= 0.985;
    bola.vy *= 0.985;

    // Rebater nas laterais
    if (bola.y < bola.raio || bola.y > canvas.height - bola.raio) {
        bola.vy *= -1;
    }

    // Gol do jogador
    if (bola.x < -10 && bola.y > 220 && bola.y < 380) {
        golsCPU++;
        reiniciar();
        atualizarPlacar();
        return;
    }

    // Gol da CPU
    if (bola.x > canvas.width + 10 && bola.y > 220 && bola.y < 380) {
        golsJogador++;
        reiniciar();
        atualizarPlacar();
        return;
    }

    // Limites sem gol
    if (bola.x < bola.raio) {
        bola.x = bola.raio;
        bola.vx *= -1;
    }

    if (bola.x > canvas.width - bola.raio) {
        bola.x = canvas.width - bola.raio;
        bola.vx *= -1;
    }
}

function colisao(p) {
    const dx = bola.x - p.x;
    const dy = bola.y - p.y;

    const d = Math.hypot(dx, dy);

    if (d < p.raio + bola.raio) {
        const angulo = Math.atan2(dy, dx);

        bola.x = p.x + Math.cos(angulo) * (p.raio + bola.raio);
        bola.y = p.y + Math.sin(angulo) * (p.raio + bola.raio);

        bola.vx = Math.cos(angulo) * 3;
        bola.vy = Math.sin(angulo) * 3;
    }
}

function reiniciar() {
    jogador.x = 250;
    jogador.y = 300;

    cpu.x = 750;
    cpu.y = 300;

    bola.x = 500;
    bola.y = 300;

    bola.vx = 0;
    bola.vy = 0;
}

function atualizarPlacar() {
    placar.textContent =
        `Você ${golsJogador} x ${golsCPU} CPU`;
}

function desenharCampo() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Campo
    ctx.fillStyle = "#159447";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Linhas
    ctx.strokeStyle = "white";
    ctx.lineWidth = 4;

    ctx.strokeRect(10, 10, canvas.width - 20, canvas.height - 20);

    // Linha central
    ctx.beginPath();
    ctx.moveTo(canvas.width / 2, 10);
    ctx.lineTo(canvas.width / 2, canvas.height - 10);
    ctx.stroke();

    // Círculo central
    ctx.beginPath();
    ctx.arc(500, 300, 80, 0, Math.PI * 2);
    ctx.stroke();

    // Ponto central
    ctx.beginPath();
    ctx.arc(500, 300, 5, 0, Math.PI * 2);
    ctx.fillStyle = "white";
    ctx.fill();

    // Áreas
    ctx.strokeRect(10, 180, 130, 240);
    ctx.strokeRect(860, 180, 130, 240);

    // Gols
    ctx.strokeRect(-5, 220, 25, 160);
    ctx.strokeRect(980, 220, 25, 160);
}

function desenharJogador(p) {
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.raio, 0, Math.PI * 2);
    ctx.fillStyle = p.cor;
    ctx.fill();

    ctx.strokeStyle = "white";
    ctx.lineWidth = 3;
    ctx.stroke();
}

function desenharBola() {
    ctx.beginPath();
    ctx.arc(
        bola.x,
        bola.y,
        bola.raio,
        0,
        Math.PI * 2
    );

    ctx.fillStyle = "white";
    ctx.fill();

    ctx.strokeStyle = "black";
    ctx.stroke();
}

function jogo() {
    moverJogador();
    moverCPU();

    colisao(jogador);
    colisao(cpu);

    moverBola();

    desenharCampo();
    desenharJogador(jogador);
    desenharJogador(cpu);
    desenharBola();

    requestAnimationFrame(jogo);
}

atualizarPlacar();
jogo();
</script>

</body>
</html>

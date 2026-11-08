
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");


const musica = document.getElementById("musica");
const somComer = document.getElementById("somComer");
const somGameOver = document.getElementById("somGameOver");
const somClique = document.getElementById("somClique");

function tocarSom(som) {
    som.currentTime = 0;
    som.play();
}


const tamanho = 20;

let cobrinha;
let direcao;
let proximaDirecao;
let comida;
let pontos;
let jogoAtivo = false;
let intervalo;
let cobrinhaAnterior = [];
let inicioAnimacao = 0;
let quadroAnimacao;
const duracaoMovimento = 150;

function iniciarJogo() {
    clearInterval(intervalo);

    cobrinha = [
        { x: 200, y: 200 },
        { x: 180, y: 200 },
        { x: 160, y: 200 }
    ];
    cobrinhaAnterior = cobrinha.map(parte => ({ ...parte }));
inicioAnimacao = performance.now();

    direcao = "DIREITA";
    proximaDirecao = "DIREITA";
    pontos = 0;

    document.getElementById("score").textContent = pontos;

    document.getElementById("telaInicial").classList.add("escondido");
    document.getElementById("telaFinal").classList.add("escondido");
    document.getElementById("areaJogo").style.display = "block";

    criarComida();

    jogoAtivo = true;
animar();

musica.volume = 0.25;
musica.play();
tocarSom(somClique);

    intervalo = setInterval(atualizar, 150);
}

function criarComida() {
    comida = {
        x: Math.floor(Math.random() * 20) * tamanho,
        y: Math.floor(Math.random() * 20) * tamanho
    };
}


function desenhar() {
    ctx.fillStyle = "#222";
    ctx.fillRect(0, 0, 400, 400);

    // Grade do tabuleiro
    ctx.strokeStyle = "#333";
    ctx.lineWidth = 1;

    for (let i = 0; i <= 400; i += 20) {
        ctx.beginPath();
        ctx.moveTo(i, 0);
        ctx.lineTo(i, 400);
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(0, i);
        ctx.lineTo(400, i);
        ctx.stroke();
    }

    // Comida
    ctx.fillStyle = "red";
    ctx.beginPath();
    ctx.arc(comida.x + 10, comida.y + 10, 9, 0, Math.PI * 2);
    ctx.fill();

    // Animação da cobrinha
    const progresso = Math.min(
        (performance.now() - inicioAnimacao) / duracaoMovimento,
        1
    );

    cobrinha.forEach((parte, index) => {
        const anterior = cobrinhaAnterior[index] ||
            cobrinhaAnterior[cobrinhaAnterior.length - 1] ||
            parte;

        const x = anterior.x + (parte.x - anterior.x) * progresso;
        const y = anterior.y + (parte.y - anterior.y) * progresso;

        ctx.fillStyle = index === 0 ? "#55ff77" : "#32cc55";

        ctx.fillRect(
            x,
            y,
            tamanho - 2,
            tamanho - 2
        );
    });

    const cabecaAnterior = cobrinhaAnterior[0] || cobrinha[0];

const xCabeca = cabecaAnterior.x +
    (cobrinha[0].x - cabecaAnterior.x) * progresso;

const yCabeca = cabecaAnterior.y +
    (cobrinha[0].y - cabecaAnterior.y) * progresso;

desenharRostinho(xCabeca, yCabeca);
}

function animar() {
    if (!jogoAtivo) return;

    desenhar();
    quadroAnimacao = requestAnimationFrame(animar);
}

function desenharRostinho(x, y) {

    ctx.fillStyle = "black";

    let olho1;
    let olho2;

    if (direcao === "DIREITA") {
        olho1 = { x: x + 14, y: y + 5 };
        olho2 = { x: x + 14, y: y + 13 };
    } else if (direcao === "ESQUERDA") {
        olho1 = { x: x + 4, y: y + 5 };
        olho2 = { x: x + 4, y: y + 13 };
    } else if (direcao === "CIMA") {
        olho1 = { x: x + 5, y: y + 4 };
        olho2 = { x: x + 13, y: y + 4 };
    } else {
        olho1 = { x: x + 5, y: y + 14 };
        olho2 = { x: x + 13, y: y + 14 };
    }

    ctx.beginPath();
    ctx.arc(olho1.x, olho1.y, 2, 0, Math.PI * 2);
    ctx.arc(olho2.x, olho2.y, 2, 0, Math.PI * 2);
    ctx.fill();
}

function atualizar() {
    if (!jogoAtivo) return;

    direcao = proximaDirecao;

    let cabeca = { ...cobrinha[0] };

    if (direcao === "DIREITA") cabeca.x += tamanho;
    if (direcao === "ESQUERDA") cabeca.x -= tamanho;
    if (direcao === "CIMA") cabeca.y -= tamanho;
    if (direcao === "BAIXO") cabeca.y += tamanho;

    // Paredes
    if (
        cabeca.x < 0 ||
        cabeca.x >= 400 ||
        cabeca.y < 0 ||
        cabeca.y >= 400
    ) {
        finalizarJogo();
        return;
    }

    // Colisão com o próprio corpo
    if (cobrinha.some(parte => parte.x === cabeca.x && parte.y === cabeca.y)) {
        finalizarJogo();
        return;
    }
    cobrinhaAnterior = cobrinha.map(parte => ({ ...parte }));
    cobrinha.unshift(cabeca);

    if (cabeca.x === comida.x && cabeca.y === comida.y) {
        pontos++;
        tocarSom(somComer);
        document.getElementById("score").textContent = pontos;

        criarComida();
    } else {
        cobrinha.pop();
    }
    inicioAnimacao = performance.now();
}


function finalizarJogo() {
    jogoAtivo = false;
    clearInterval(intervalo);
    musica.pause();
musica.currentTime = 0;
tocarSom(somGameOver);
    document.getElementById("pontuacaoFinal").textContent = pontos;
    document.getElementById("telaFinal").classList.remove("escondido");
}

document.addEventListener("keydown", event => {
    if (!jogoAtivo) return;

    if (event.key === "ArrowRight" && direcao !== "ESQUERDA")
        proximaDirecao = "DIREITA";

    if (event.key === "ArrowLeft" && direcao !== "DIREITA")
        proximaDirecao = "ESQUERDA";

    if (event.key === "ArrowUp" && direcao !== "BAIXO")
        proximaDirecao = "CIMA";

    if (event.key === "ArrowDown" && direcao !== "CIMA")
        proximaDirecao = "BAIXO";
});

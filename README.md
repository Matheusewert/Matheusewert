<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Pong Game | Matheus Ewert</title>

  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      min-height: 100vh;
      background: #080d1b;
      color: white;
      font-family: Arial, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    h1 {
      color: #35a7ff;
      margin-bottom: 10px;
      letter-spacing: 4px;
    }

    p {
      color: #aab8d0;
      margin-bottom: 20px;
      text-align: center;
    }

    canvas {
      width: 100%;
      max-width: 800px;
      background: #101b31;
      border: 2px solid #168cff;
      border-radius: 10px;
      box-shadow: 0 0 25px #087cff55;
    }

    .controls {
      margin-top: 20px;
      text-align: center;
      line-height: 1.8;
    }

    button {
      margin-top: 15px;
      padding: 12px 25px;
      border: none;
      border-radius: 6px;
      background: #168cff;
      color: white;
      font-size: 16px;
      cursor: pointer;
    }

    button:hover {
      background: #0066cc;
    }
  </style>
</head>

<body>

  <h1>PONG GAME</h1>
  <p>Um clássico reinventado por Matheus Ewert</p>

  <canvas id="game" width="800" height="450"></canvas>

  <div class="controls">
    <strong>Jogador 1:</strong> W e S
    &nbsp; | &nbsp;
    <strong>Jogador 2:</strong> ↑ e ↓
    <br>
    Primeiro jogador a marcar 5 pontos vence!
    <br>
    <button onclick="restartGame()">Reiniciar jogo</button>
  </div>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    const paddleWidth = 12;
    const paddleHeight = 90;

    let player1, player2, ball;
    let score1 = 0;
    let score2 = 0;
    let gameOver = false;

    const keys = {};

    function resetBall(direction = 1) {
      ball = {
        x: canvas.width / 2,
        y: canvas.height / 2,
        radius: 9,
        speedX: 5 * direction,
        speedY: 3
      };
    }

    function restartGame() {
      score1 = 0;
      score2 = 0;
      gameOver = false;

      player1 = {
        x: 25,
        y: canvas.height / 2 - paddleHeight / 2,
        score: 0
      };

      player2 = {
        x: canvas.width - 25 - paddleWidth,
        y: canvas.height / 2 - paddleHeight / 2,
        score: 0
      };

      resetBall();
    }

    document.addEventListener("keydown", (e) => {
      keys[e.key.toLowerCase()] = true;

      if (["ArrowUp", "ArrowDown", " "].includes(e.key)) {
        e.preventDefault();
      }

      if (gameOver && e.key === "Enter") {
        restartGame();
      }
    });

    document.addEventListener("keyup", (e) => {
      keys[e.key.toLowerCase()] = false;
    });

    function movePaddles() {
      const speed = 7;

      if (keys["w"]) player1.y -= speed;
      if (keys["s"]) player1.y += speed;

      if (keys["arrowup"]) player2.y -= speed;
      if (keys["arrowdown"]) player2.y += speed;

      player1.y = Math.max(0, Math.min(
        canvas.height - paddleHeight, player1.y
      ));

      player2.y = Math.max(0, Math.min(
        canvas.height - paddleHeight, player2.y
      ));
    }

    function update() {
      if (gameOver) return;

      movePaddles();

      ball.x += ball.speedX;
      ball.y += ball.speedY;

      if (
        ball.y - ball.radius <= 0 ||
        ball.y + ball.radius >= canvas.height
      ) {
        ball.speedY *= -1;
      }

      // Colisão com a raquete do jogador 1
      if (
        ball.speedX < 0 &&
        ball.x - ball.radius <= player1.x + paddleWidth &&
        ball.x + ball.radius >= player1.x &&
        ball.y >= player1.y &&
        ball.y <= player1.y + paddleHeight
      ) {
        ball.speedX = Math.abs(ball.speedX) * 1.05;
        ball.speedY += (ball.y - (player1.y + paddleHeight / 2)) * 0.08;
      }

      // Colisão com a raquete do jogador 2
      if (
        ball.speedX > 0 &&
        ball.x + ball.radius >= player2.x &&
        ball.x - ball.radius <= player2.x + paddleWidth &&
        ball.y >= player2.y &&
        ball.y <= player2.y + paddleHeight
      ) {
        ball.speedX = -Math.abs(ball.speedX) * 1.05;
        ball.speedY += (ball.y - (player2.y + paddleHeight / 2)) * 0.08;
      }

      // Pontuação
      if (ball.x < 0) {
        score2++;
        resetBall(-1);
      }

      if (ball.x > canvas.width) {
        score1++;
        resetBall(1);
      }

      if (score1 >= 5 || score2 >= 5) {
        gameOver = true;
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Linha central
      ctx.setLineDash([10, 12]);
      ctx.strokeStyle = "#52627e";
      ctx.beginPath();
      ctx.moveTo(canvas.width / 2, 0);
      ctx.lineTo(canvas.width / 2, canvas.height);
      ctx.stroke();
      ctx.setLineDash([]);

      // Placar
      ctx.fillStyle = "#ffffff";
      ctx.font = "bold 50px Arial";
      ctx.textAlign = "center";
      ctx.fillText(score1, canvas.width / 4, 65);
      ctx.fillText(score2, canvas.width * 3 / 4, 65);

      // Raquetes
      ctx.fillStyle = "#35a7ff";
      ctx.fillRect(player1.x, player1.y, paddleWidth, paddleHeight);

      ctx.fillStyle = "#ff4d8d";
      ctx.fillRect(player2.x, player2.y, paddleWidth, paddleHeight);

      // Bola
      ctx.beginPath();
      ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
      ctx.fillStyle = "#ffffff";
      ctx.shadowBlur = 15;
      ctx.shadowColor = "#ffffff";
      ctx.fill();
      ctx.shadowBlur = 0;

      // Fim de jogo
      if (gameOver) {
        ctx.fillStyle = "rgba(0, 0, 0, 0.75)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#35a7ff";
        ctx.font = "bold 36px Arial";
        ctx.fillText(
          score1 >= 5 ? "JOGADOR 1 VENCEU!" : "JOGADOR 2 VENCEU!",
          canvas.width / 2,
          canvas.height / 2 - 10
        );

        ctx.fillStyle = "#ffffff";
        ctx.font = "20px Arial";
        ctx.fillText(
          "Pressione ENTER ou clique em Reiniciar",
          canvas.width / 2,
          canvas.height / 2 + 35
        );
      }
    }

    function gameLoop() {
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }

    restartGame();
    gameLoop();
  </script>

</body>
</html>



# 👋 Olá! Eu sou Matheus Ewert

<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=600&size=22&pause=1000&color=2F81F7&center=true&vCenter=true&width=650&lines=Mercado+Financeiro+%7C+Tecnologia;Produtos+Digitais+%7C+Fintechs;Aprendendo+e+construindo+solucoes!" alt="Apresentação animada" />

</div>

<div align="center">

### 💼 Administração | Mercado Financeiro | Tecnologia

Buscando conectar negócios, tecnologia e inovação para desenvolver soluções digitais.

</div>

---

## 👨‍💻 Sobre mim

Sou bacharel em Administração e atuo no mercado financeiro. Tenho experiência comercial, visão de negócios e relacionamento com clientes.

Atualmente, estou ampliando meus conhecimentos em tecnologia, desenvolvimento de produtos digitais e programação, com foco em soluções para o setor financeiro.

- 💼 Experiência no mercado financeiro
- 🎓 Bacharel em Administração
- 📊 Interesse em investimentos e educação financeira
- 🚀 Interesse em Product Management e Fintechs
- 💻 Aprendendo programação e desenvolvimento de soluções digitais

## 🛠️ Tecnologias e ferramentas

<div align="center">

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

</div>

## 🎯 Objetivos profissionais

- Desenvolver projetos digitais próprios.
- Aprimorar conhecimentos em tecnologia e programação.
- Criar soluções voltadas ao mercado financeiro.
- Aprender sobre gestão e desenvolvimento de produtos digitais.
- Unir conhecimento de negócios, finanças e tecnologia.

## 🚀 Projetos

Atualmente, estou desenvolvendo projetos para aplicar meus conhecimentos e construir meu portfólio.

Em breve, novos projetos estarão disponíveis por aqui!

## 📫 Onde me encontrar

- **LinkedIn:** [Acesse meu perfil](https://www.linkedin.com/in/em/matheusewert)
- **GitHub:** [Confira meus projetos](https://github.com/Matheusewert)

---

<div align="center">

**Obrigado pela visita! 🚀**

![Visitas](https://komarev.com/ghpvc/?username=Matheusewert&color=blue&style=for-the-badge&label=VISITAS+NO+PERFIL)

</div>




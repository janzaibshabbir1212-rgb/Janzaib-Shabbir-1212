
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Pong Game</title>
  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      background: #111827;
      color: white;
      font-family: Arial, sans-serif;
    }

    .game-wrap {
      text-align: center;
    }

    #scoreboard {
      font-size: 28px;
      font-weight: bold;
      margin-bottom: 12px;
      letter-spacing: 2px;
    }

    canvas {
      background: #0b1220;
      border: 4px solid #e5e7eb;
      border-radius: 8px;
      box-shadow: 0 0 30px rgba(255, 255, 255, 0.1);
      display: block;
    }
  </style>
</head>
<body>
  <div class="game-wrap">
    <div id="scoreboard">0 : 0</div>
    <canvas id="gameCanvas" width="900" height="500"></canvas>
  </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreboard = document.getElementById("scoreboard");

    const paddleWidth = 14;
    const paddleHeight = 110;
    const paddleSpeed = 7;
    const ballSize = 12;

    let leftScore = 0;
    let rightScore = 0;

    const leftPaddle = {
      x: 20,
      y: canvas.height / 2 - paddleHeight / 2,
      width: paddleWidth,
      height: paddleHeight,
      speed: paddleSpeed
    };

    const rightPaddle = {
      x: canvas.width - 20 - paddleWidth,
      y: canvas.height / 2 - paddleHeight / 2,
      width: paddleWidth,
      height: paddleHeight,
      speed: 5
    };

    const ball = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      size: ballSize,
      vx: 5,
      vy: 3
    };

    const keys = {
      ArrowUp: false,
      ArrowDown: false
    };

    let mouseY = canvas.height / 2;

    window.addEventListener("keydown", (event) => {
      if (event.key === "ArrowUp" || event.key === "ArrowDown") {
        keys[event.key] = true;
      }
    });

    window.addEventListener("keyup", (event) => {
      if (event.key === "ArrowUp" || event.key === "ArrowDown") {
        keys[event.key] = false;
      }
    });

    canvas.addEventListener("mousemove", (event) => {
      const rect = canvas.getBoundingClientRect();
      mouseY = ((event.clientY - rect.top) / (rect.height)) * canvas.height;
    });

    function resetBall(direction) {
      ball.x = canvas.width / 2;
      ball.y = canvas.height / 2;
      const baseSpeed = 5;
      const randomAngle = (Math.random() * 1.2) - 0.6;

      ball.vx = direction * (baseSpeed + Math.random() * 1.5);
      ball.vy = randomAngle * 5;
    }

    function updateLeftPaddle() {
      // Mouse control
      const targetY = mouseY - leftPaddle.height / 2;
      const diff = targetY - leftPaddle.y;
      leftPaddle.y += diff * 0.18;

      // Keyboard control (works alongside mouse)
      if (keys.ArrowUp) leftPaddle.y -= leftPaddle.speed;
      if (keys.ArrowDown) leftPaddle.y += leftPaddle.speed;

      leftPaddle.y = Math.max(0, Math.min(canvas.height - leftPaddle.height, leftPaddle.y));
    }

    function updateRightPaddle() {
      const paddleCenter = rightPaddle.y + rightPaddle.height / 2;
      const ballCenter = ball.y + ball.size / 2;

      if (ballCenter < paddleCenter) {
        rightPaddle.y -= rightPaddle.speed;
      } else if (ballCenter > paddleCenter) {
        rightPaddle.y += rightPaddle.speed;
      }

      rightPaddle.y = Math.max(0, Math.min(canvas.height - rightPaddle.height, rightPaddle.y));
    }

    function updateBall() {
      ball.x += ball.vx;
      ball.y += ball.vy;

      // Top and bottom wall collision
      if (ball.y <= 0 || ball.y + ball.size >= canvas.height) {
        ball.vy *= -1;
        ball.y = Math.max(0, Math.min(canvas.height - ball.size, ball.y));
      }

      // Left paddle collision
      if (
        ball.x <= leftPaddle.x + leftPaddle.width &&
        ball.x + ball.size >= leftPaddle.x &&
        ball.y + ball.size >= leftPaddle.y &&
        ball.y <= leftPaddle.y + leftPaddle.height &&
        ball.vx < 0
      ) {
        ball.x = leftPaddle.x + leftPaddle.width;
        ball.vx = Math.abs(ball.vx) + 0.2;
        const impact = (ball.y + ball.size / 2 - (leftPaddle.y + leftPaddle.height / 2)) / (leftPaddle.height / 2);
        ball.vy = impact * 6;
      }

      // Right paddle collision
      if (
        ball.x + ball.size >= rightPaddle.x &&
        ball.x <= rightPaddle.x + rightPaddle.width &&
        ball.y + ball.size >= rightPaddle.y &&
        ball.y <= rightPaddle.y + rightPaddle.height &&
        ball.vx > 0
      ) {
        ball.x = rightPaddle.x - ball.size;
        ball.vx = -Math.abs(ball.vx) - 0.2;
        const impact = (ball.y + ball.size / 2 - (rightPaddle.y + rightPaddle.height / 2)) / (rightPaddle.height / 2);
        ball.vy = impact * 6;
      }

      // Scoring
      if (ball.x + ball.size < 0) {
        rightScore++;
        updateScoreboard();
        resetBall(1);
      }

      if (ball.x > canvas.width) {
        leftScore++;
        updateScoreboard();
        resetBall(-1);
      }
    }

    function updateScoreboard() {
      scoreboard.textContent = `${leftScore} : ${rightScore}`;
    }

    function drawNet() {
      ctx.strokeStyle = "rgba(255,255,255,0.5)";
      ctx.lineWidth = 3;
      ctx.setLineDash([12, 12]);
      ctx.beginPath();
      ctx.moveTo(canvas.width / 2, 0);
      ctx.lineTo(canvas.width / 2, canvas.height);
      ctx.stroke();
      ctx.setLineDash([]);
    }

    function drawPaddle(x, y, width, height) {
      ctx.fillStyle = "#f8fafc";
      ctx.fillRect(x, y, width, height);
    }

    function drawBall(x, y, size) {
      ctx.fillStyle = "#fbbf24";
      ctx.beginPath();
      ctx.arc(x + size / 2, y + size / 2, size / 2, 0, Math.PI * 2);
      ctx.fill();
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      drawNet();
      drawPaddle(leftPaddle.x, leftPaddle.y, leftPaddle.width, leftPaddle.height);
      drawPaddle(rightPaddle.x, rightPaddle.y, rightPaddle.width, rightPaddle.height);
      drawBall(ball.x, ball.y, ball.size);

      ctx.fillStyle = "rgba(255,255,255,0.2)";
      ctx.fillRect(0, 0, canvas.width, 3);
      ctx.fillRect(0, canvas.height - 3, canvas.width, 3);
    }

    function gameLoop() {
      updateLeftPaddle();
      updateRightPaddle();
      updateBall();
      draw();
      requestAnimationFrame(gameLoop);
    }

    updateScoreboard();
    resetBall(1);
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const scoreEl = document.getElementById('score');
const livesEl = document.getElementById('lives');
const timeEl = document.getElementById('time');
const messageEl = document.getElementById('message');
const startBtn = document.getElementById('startBtn');

const GAME_WIDTH = canvas.width;
const GAME_HEIGHT = canvas.height;
const PLAYER_WIDTH = 120;
const PLAYER_HEIGHT = 16;
const PLAYER_Y = GAME_HEIGHT - 30;
const PLAYER_SPEED = 7;

let score = 0;
let lives = 3;
let timeLeft = 60;
let gameRunning = false;
let animationId = null;
let lastSpawn = 0;
let lastTime = 0;
let player = {
  x: GAME_WIDTH / 2 - PLAYER_WIDTH / 2,
  y: PLAYER_Y,
  width: PLAYER_WIDTH,
  height: PLAYER_HEIGHT,
}

const keys = {
  left: false,
  right: false,
};

let fallingItems = [];

function resetGame() {
  score = 0;
  lives = 3;
  timeLeft = 60;
  fallingItems = [];
  lastSpawn = 0;
  player.x = GAME_WIDTH / 2 - PLAYER_WIDTH / 2;
  scoreEl.textContent = String(score);
  livesEl.textContent = String(lives);
  timeEl.textContent = String(timeLeft);
  messageEl.textContent = 'Catch the glowing stars and avoid the dark meteors.';
}

function startGame() {
  resetGame();
  gameRunning = true;
  lastTime = performance.now();
  startBtn.textContent = 'Restart Game';
  messageEl.textContent = 'Game on!';
  cancelAnimationFrame(animationId);
  animationId = requestAnimationFrame(gameLoop);
}

function endGame() {
  gameRunning = false;
  cancelAnimationFrame(animationId);
  const finalMessage = score >= 20 ? 'Amazing! You are a sky catcher!' : 'Game over! Try again!';
  messageEl.textContent = `${finalMessage} Final score: ${score}`;
}

function spawnItem() {
  const isBomb = Math.random() < 0.26;
  const radius = isBomb ? 18 : 14;
  const x = Math.random() * (GAME_WIDTH - radius * 2) + radius;
  const speed = isBomb ? 2.2 + Math.random() * 1.4 : 2 + Math.random() * 1.2;

  fallingItems.push({
    x,
    y: -radius,
    radius,
    speed,
    type: isBomb ? 'bomb' : 'star',
  });
}

function updatePlayer() {
  if (keys.left) player.x -= PLAYER_SPEED;
  if (keys.right) player.x += PLAYER_SPEED;
  player.x = Math.max(0, Math.min(GAME_WIDTH - player.width, player.x));
}

function updateItems(delta) {
  for (let i = fallingItems.length - 1; i >= 0; i--) {
    const item = fallingItems[i];
    item.y += item.speed * delta * 0.06;

    const isColliding =
      item.x + item.radius > player.x &&
      item.x - item.radius < player.x + player.width &&
      item.y + item.radius > player.y &&
      item.y - item.radius < player.y + player.height;

    if (isColliding) {
      if (item.type === 'star') {
        score += 10;
        messageEl.textContent = 'Nice catch!';
      } else {
        lives -= 1;
        messageEl.textContent = 'Oops! You hit a meteor.';
      }

      scoreEl.textContent = String(score);
      livesEl.textContent = String(lives);
      fallingItems.splice(i, 1);

      if (lives <= 0) {
        endGame();
        return;
      }
      continue;
    }

    if (item.y - item.radius > GAME_HEIGHT) {
      if (item.type === 'star') {
        lives -= 1;
        livesEl.textContent = String(lives);
        messageEl.textContent = 'A star slipped away!';
      }
      fallingItems.splice(i, 1);
      if (lives <= 0) {
        endGame();
        return;
      }
    }
  }
}

function updateTimer(delta) {
  timeLeft -= delta * 0.001;
  timeEl.textContent = String(Math.max(0, Math.ceil(timeLeft)));

  if (timeLeft <= 0) {
    endGame();
    messageEl.textContent = `Time's up! Final score: ${score}`;
  }
}

function drawBackground() {
  ctx.clearRect(0, 0, GAME_WIDTH, GAME_HEIGHT);

  const sky = ctx.createLinearGradient(0, 0, 0, GAME_HEIGHT);
  sky.addColorStop(0, '#112d4e');
  sky.addColorStop(1, '#081722');
  ctx.fillStyle = sky;
  ctx.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);

  for (let i = 0; i < 65; i += 1) {
    const x = (i * 79) % GAME_WIDTH;
    const y = (i * 53) % (GAME_HEIGHT - 80);
    ctx.fillStyle = 'rgba(255,255,255,0.7)';
    ctx.fillRect(x, y, 2, 2);
  }
}

function drawPlayer() {
  ctx.fillStyle = '#66d9ef';
  ctx.fillRect(player.x, player.y, player.width, player.height);

  ctx.fillStyle = 'rgba(255,255,255,0.35)';
  ctx.fillRect(player.x + 8, player.y + 4, player.width - 16, 3);
}

function drawItems() {
  for (const item of fallingItems) {
    if (item.type === 'star') {
      ctx.beginPath();
      ctx.moveTo(item.x, item.y - item.radius);
      for (let i = 1; i <= 10; i += 1) {
        const angle = (i * Math.PI) / 5;
        const outerRadius = item.radius;
        const innerRadius = item.radius / 2.5;
        const x = item.x + Math.cos(angle - Math.PI / 2) * (i % 2 === 0 ? innerRadius : outerRadius);
        const y = item.y + Math.sin(angle - Math.PI / 2) * (i % 2 === 0 ? innerRadius : outerRadius);
        if (i === 1) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();
      ctx.fillStyle = '#ffca3a';
      ctx.shadowColor = '#ffd76d';
      ctx.shadowBlur = 18;
      ctx.fill();
      ctx.shadowBlur = 0;
    } else {
      ctx.beginPath();
      ctx.arc(item.x, item.y, item.radius, 0, Math.PI * 2);
      ctx.fillStyle = '#ff5d73';
      ctx.shadowColor = '#ff7d8e';
      ctx.shadowBlur = 18;
      ctx.fill();
      ctx.shadowBlur = 0;

      ctx.beginPath();
      ctx.moveTo(item.x - 7, item.y);
      ctx.lineTo(item.x + 7, item.y);
      ctx.moveTo(item.x, item.y - 7);
      ctx.lineTo(item.x, item.y + 7);
      ctx.strokeStyle = '#ffd4da';
      ctx.lineWidth = 2;
      ctx.stroke();
    }
  }
}

function draw() {
  drawBackground();
  drawPlayer();
  drawItems();
}

function gameLoop(timestamp) {
  if (!gameRunning) return;

  const delta = timestamp - lastTime;
  lastTime = timestamp;

  if (timestamp - lastSpawn > 800) {
    spawnItem();
    lastSpawn = timestamp;
  }

  updatePlayer();
  updateItems(delta);
  updateTimer(delta);
  draw();

  if (gameRunning) {
    animationId = requestAnimationFrame(gameLoop);
  }
}

window.addEventListener('keydown', (event) => {
  if (event.key === 'ArrowLeft' || event.key.toLowerCase() === 'a') keys.left = true;
  if (event.key === 'ArrowRight' || event.key.toLowerCase() === 'd') keys.right = true;
});

window.addEventListener('keyup', (event) => {
  if (event.key === 'ArrowLeft' || event.key.toLowerCase() === 'a') keys.left = false;
  if (event.key === 'ArrowRight' || event.key.toLowerCase() === 'd') keys.right = false;
});

canvas.addEventListener('mousemove', (event) => {
  const rect = canvas.getBoundingClientRect();
  const mouseX = ((event.clientX - rect.left) / rect.width) * GAME_WIDTH;
  player.x = mouseX - player.width / 2;
});

startBtn.addEventListener('click', startGame);

resetGame();
draw();

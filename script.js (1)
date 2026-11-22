
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let W = 0;
let H = 0;
let running = false;
let gameOver = false;
let score = 0;
let coins = 0;
let lives = 3;
let distance = 0;
let speed = 5;
let spawnTimer = 0;
let coinTimer = 0;
let obstacles = [];
let coinItems = [];
let particles = [];
let playerLane = 1;
let playerY = 0;
let lastTime = 0;
let animationId = 0;
let stunned = false;
let slowMotion = false;
const laneCount = 3;
const multipliers = [
  { value: 2, cost: 250 },
  { value: 5, cost: 500 },
  { value: 10, cost: 1000 }
];

let wallet = Number(localStorage.getItem("sr_wallet")) || 0;
let bestScore = Number(localStorage.getItem("sr_record")) || 0;
let multiplier = Number(localStorage.getItem("sr_multiplier")) || 1;
let purchased = JSON.parse(
  localStorage.getItem("sr_purchased") || "[]"
);
let reviveChance = Number(localStorage.getItem("sr_reviveChance")) || 2;
let revivePending = false;
if (!Array.isArray(purchased)) purchased = [];

function saveData() {
  localStorage.setItem("sr_wallet", wallet);
  localStorage.setItem("sr_record", bestScore);
  localStorage.setItem("sr_multiplier", multiplier);
  localStorage.setItem("sr_purchased", JSON.stringify(purchased));
}
function saveData() {
  localStorage.setItem("sr_wallet", wallet);
  localStorage.setItem("sr_record", bestScore);
  localStorage.setItem("sr_multiplier", multiplier);
  localStorage.setItem("sr_purchased", JSON.stringify(purchased));
  localStorage.setItem("sr_reviveChance", reviveChance);
}
function resizeCanvas() {
  W = window.innerWidth;
  H = window.innerHeight;

  const dpr = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width = Math.round(W * dpr);
  canvas.height = Math.round(H * dpr);
  canvas.style.width = W + "px";
  canvas.style.height = H + "px";

  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  playerY = H * 0.78;
}

window.addEventListener("resize", resizeCanvas);
resizeCanvas();

function laneX(lane) {
  const roadWidth = Math.min(W * 0.88, 480);
  const roadLeft = (W - roadWidth) / 2;
  return roadLeft + roadWidth * (lane + 0.5) / laneCount;
}

function roadBounds() {
  const roadWidth = Math.min(W * 0.88, 480);
  return {
    left: (W - roadWidth) / 2,
    right: (W + roadWidth) / 2,
    width: roadWidth
  };
}

function createInterface() {
  const hud = document.createElement("div");
  hud.id = "hud";
  hud.innerHTML = `
    <div>
      <div class="hud-box">⭐ Pontos: <span id="score">0</span></div>
      <div class="hud-box" style="margin-top:5px">
        🪙 Moedas: <span id="wallet">0</span>
      </div>
    </div>
    <div>
      <div class="hud-box">❤️ Vidas: <span id="lives">3</span></div>
      <div class="hud-box" style="margin-top:5px">
        ⚡ Multiplicador: <span id="multiplier">1</span>x
      </div>
    </div>
  `;
  document.body.appendChild(hud);

  const shopButton = document.createElement("button");
  shopButton.id = "shopButton";
  shopButton.textContent = "🛒 Loja";
  shopButton.onclick = openShop;
  document.body.appendChild(shopButton);
  const pauseButton = document.createElement("button");
pauseButton.id = "pauseButton";
pauseButton.textContent = "⏸ PAUSAR";
pauseButton.onclick = pauseGame;
document.body.appendChild(pauseButton);

  const controls = document.createElement("div");
  controls.id = "mobileControls";
  controls.innerHTML = `
    <button class="control-button" id="leftButton" aria-label="Esquerda">◀</button>
    <button class="control-button" id="rightButton" aria-label="Direita">▶</button>
  `;
  document.body.appendChild(controls);

  document.getElementById("leftButton").addEventListener("pointerdown", e => {
    e.preventDefault();
    movePlayer(-1);
  });

  document.getElementById("rightButton").addEventListener("pointerdown", e => {
    e.preventDefault();
    movePlayer(1);
  });

  const startScreen = document.createElement("div");
  startScreen.id = "startScreen";
  startScreen.className = "overlay";
  startScreen.innerHTML = `
    <div class="panel">
      <h1>🚇 Subway Runner V2</h1>
      <p>Desvie dos obstáculos, pegue moedas e aumente seus pontos!</p>
      <p>🏆 Recorde: <strong id="startRecord">0</strong></p>
      <p>🪙 Carteira: <strong id="startWallet">0</strong> moedas</p>
      <button class="game-button" id="startButton">▶ JOGAR</button>
      <button class="game-button secondary" id="startShopButton">🛒 LOJA</button>
      <p style="font-size:13px">Use as setas ou deslize o dedo para mudar de pista.</p>
    </div>
  `;
  document.body.appendChild(startScreen);

  document.getElementById("startButton").onclick = startGame;
  document.getElementById("startShopButton").onclick = openShop;

  const shop = document.createElement("div");
  shop.id = "shopScreen";
  shop.className = "overlay hidden";
  shop.innerHTML = `
    <div class="panel">
      <h1>🛒 Loja de melhorias</h1>
      <p>Suas moedas: <strong id="shopWallet">0</strong> 🪙</p>
      <p>Multiplicador atual: <strong id="shopMultiplier">1</strong>x</p>
      <div id="shopItems"></div>
      <button class="game-button secondary" id="closeShop">VOLTAR</button>
    </div>
  `;
  document.body.appendChild(shop);
  document.getElementById("closeShop").onclick = closeShop;

  const over = document.createElement("div");
  over.id = "gameOverScreen";
  over.className = "overlay hidden";
  over.innerHTML = `
    <div class="panel">
      <h1>💥 Fim de jogo!</h1>
      <p>Pontuação: <strong id="finalScore">0</strong></p>
      <p>Recorde: <strong id="finalRecord">0</strong></p>
      <p>Moedas coletadas nesta partida: <strong id="finalCoins">0</strong></p>
      <button class="game-button" id="restartButton">🔄 JOGAR NOVAMENTE</button>
      <button class="game-button secondary" id="overShopButton">🛒 LOJA</button>
    </div>
  `;
  document.body.appendChild(over);

const pauseScreen = document.createElement("div");
pauseScreen.id = "pauseScreen";
pauseScreen.className = "overlay hidden";

pauseScreen.innerHTML = `
  <div class="panel">
    <h1>⏸ Jogo pausado</h1>
    <p>Respire um pouco. Sua partida está esperando!</p>
    <button class="game-button" id="resumeButton">
      ▶ CONTINUAR
    </button>
    <button class="game-button secondary" id="pauseShopButton">
      🛒 LOJA
    </button>
    <button class="game-button secondary" id="pauseMenuButton">
      🏠 MENU INICIAL
    </button>
  </div>
`;

document.body.appendChild(pauseScreen);

document.getElementById("resumeButton").onclick = resumeGame;

document.getElementById("pauseShopButton").onclick = () => {
  pauseScreen.classList.add("hidden");
  openShop();
};

document.getElementById("pauseMenuButton").onclick = () => {
  running = false;
  gameOver = false;

  pauseScreen.classList.add("hidden");
  document.getElementById("shopScreen").classList.add("hidden");
  document.getElementById("gameOverScreen").classList.add("hidden");
  document.getElementById("startScreen").classList.remove("hidden");

};
  document.getElementById("restartButton").onclick = startGame;
  document.getElementById("overShopButton").onclick = openShop;

  updateHUD();
  renderShop();
}

function updateHUD() {
  const setText = (id, value) => {
    const el = document.getElementById(id);
    if (el) el.textContent = value;
  };

  setText("score", Math.floor(score));
  setText("wallet", wallet);
  setText("lives", lives);
  setText("multiplier", multiplier);
  setText("startRecord", bestScore);
  setText("startWallet", wallet);
  setText("shopWallet", wallet);
  setText("shopMultiplier", multiplier);
}

function renderShop() {
  const container = document.getElementById("shopItems");
  if (!container) return;

  container.innerHTML = "";

  // Melhorias de multiplicador
  multipliers.forEach(item => {
    const alreadyBought = purchased.includes(item.value);
    const active = multiplier === item.value;

    const row = document.createElement("div");
    row.className = "shop-item";

    const description = document.createElement("div");
    description.innerHTML = `
      <strong>${item.value}x pontos</strong>
      <small>${item.cost} moedas</small>
    `;

    const button = document.createElement("button");

    if (alreadyBought) {
      button.textContent = active ? "Equipado" : "Comprado";
      button.disabled = true;
    } else {
      button.textContent = "Comprar";
      button.disabled = wallet < item.cost;
      button.onclick = () => buyUpgrade(item);
    }

    row.appendChild(description);
    row.appendChild(button);
    container.appendChild(row);
  });

  // Melhorias da roleta
  const reviveUpgrades = [
    { value: 5, cost: 5000 },
    { value: 10, cost: 10000 }
  ];

  const title = document.createElement("h2");
  title.textContent = "🎡 Chance de reviver";
  container.appendChild(title);

  reviveUpgrades.forEach(item => {
    const row = document.createElement("div");
    row.className = "shop-item";

    const description = document.createElement("div");
    description.innerHTML = `
      <strong>${item.value}% de chance</strong>
      <small>${item.cost} moedas</small>
    `;

    const button = document.createElement("button");

    if (reviveChance >= item.value) {
      button.textContent = "Comprado";
      button.disabled = true;
    } else {
      button.textContent = "Comprar";
      button.disabled = wallet < item.cost;

      button.onclick = () => {
        if (wallet < item.cost) return;
        if (reviveChance >= item.value) return;

        wallet -= item.cost;
        reviveChance = item.value;

        saveData();
        updateHUD();
        renderShop();
      };
    }

    row.appendChild(description);
    row.appendChild(button);
    container.appendChild(row);
  });
}

function buyUpgrade(item) {
  if (purchased.includes(item.value)) return;
  if (wallet < item.cost) return;

  wallet -= item.cost;
  purchased.push(item.value);
  multiplier = item.value;

  saveData();
  updateHUD();
  renderShop();
}

function openShop() {
  const wasRunning = running;
  running = false;

  document.getElementById("shopScreen").classList.remove("hidden");
  document.getElementById("startScreen").classList.add("hidden");
  document.getElementById("gameOverScreen").classList.add("hidden");

  document.getElementById("closeShop").textContent =
    wasRunning ? "VOLTAR AO JOGO" : "VOLTAR";

  renderShop();
  updateHUD();
}

function closeShop() {
  document.getElementById("shopScreen").classList.add("hidden");

  if (gameOver) {
    document.getElementById("gameOverScreen").classList.remove("hidden");
  } else if (score === 0 && distance === 0) {
    document.getElementById("startScreen").classList.remove("hidden");
  } else {
    running = true;
    lastTime = performance.now();
  }
}

function pauseGame() {
  if (!running || gameOver) return;

  running = false;

  document.getElementById("pauseScreen").classList.remove("hidden");
}

function resumeGame() {
  if (gameOver) return;

  document.getElementById("pauseScreen").classList.add("hidden");

  running = true;
  lastTime = performance.now();

  cancelAnimationFrame(animationId);
  animationId = requestAnimationFrame(gameLoop);
}
function startGame() {
  score = 0;
  coins = 0;
  lives = 3;
  distance = 0;
  speed = 5;
  spawnTimer = 0;
  coinTimer = 0;
  obstacles = [];
  coinItems = [];
  particles = [];
  playerLane = 1;
  gameOver = false;
  running = true;
  lastTime = performance.now();

  document.getElementById("startScreen").classList.add("hidden");
  document.getElementById("shopScreen").classList.add("hidden");
  document.getElementById("gameOverScreen").classList.add("hidden");
document.getElementById("pauseScreen").classList.add("hidden");
  updateHUD();
  cancelAnimationFrame(animationId);
  animationId = requestAnimationFrame(gameLoop);
}

function movePlayer(direction) {
  if (!running || stunned) return;

  playerLane = Math.max(
    0,
    Math.min(laneCount - 1, playerLane + direction)
  );
}

document.addEventListener("keydown", e => {
  if (e.key === "ArrowLeft" || e.key.toLowerCase() === "a") {
    movePlayer(-1);
  }
  if (e.key === "ArrowRight" || e.key.toLowerCase() === "d") {
    movePlayer(1);
  }
  if (e.key === "Escape" &&
      !document.getElementById("shopScreen").classList.contains("hidden")) {
    closeShop();
  }
});

let touchStartX = 0;
let touchStartY = 0;

canvas.addEventListener("touchstart", e => {
  const touch = e.changedTouches[0];
  touchStartX = touch.clientX;
  touchStartY = touch.clientY;
}, { passive: true });

canvas.addEventListener("touchend", e => {
  if (!running) return;
  const touch = e.changedTouches[0];
  const dx = touch.clientX - touchStartX;
  const dy = touch.clientY - touchStartY;

  if (Math.abs(dx) > 25 && Math.abs(dx) > Math.abs(dy)) {
    movePlayer(dx > 0 ? 1 : -1);
  }
}, { passive: true });

function spawnObstacle() {
  const lane = Math.floor(Math.random() * laneCount);
  const types = ["barrier", "train", "sign"];
  const type = types[Math.floor(Math.random() * types.length)];

  obstacles.push({
    lane,
    y: -90,
    type,
    width: type === "train" ? 58 : 44,
    height: type === "train" ? 75 : 48,
    hit: false
  });
}

function spawnCoins() {
  const lane = Math.floor(Math.random() * laneCount);
  const count = 3 + Math.floor(Math.random() * 3);

  for (let i = 0; i < count; i++) {
    coinItems.push({
      lane,
      y: -30 - i * 45,
      taken: false
    });
  }
}

function addPoints(amount) {
  score += amount * multiplier;
  if (score > bestScore) {
    bestScore = Math.floor(score);
    localStorage.setItem("sr_record", bestScore);
  }
}

function collectCoin(coin) {
  if (coin.taken) return;

  coin.taken = true;
  coins++;
  wallet++;
  saveData();

  addPoints(10);

  // Cada 30 moedas coletadas na partida dá uma vida extra.
  if (coins % 30 === 0) {
    lives++;
  }

  for (let i = 0; i < 7; i++) {
    particles.push({
      x: laneX(coin.lane),
      y: coin.y,
      vx: (Math.random() - 0.5) * 4,
      vy: -Math.random() * 3,
      life: 25
    });
  }

  updateHUD();
}

function hitObstacle(obstacle) {
  if (obstacle.hit || revivePending || stunned) return;

  obstacle.hit = true;
  lives--;

  // Atordoa o personagem por 900 milissegundos.
  stunned = true;
slowMotion = true;

setTimeout(() => {
  slowMotion = false;
}, 900);
  setTimeout(() => {
    stunned = false;
  }, 500);

  particles.push({
    x: laneX(obstacle.lane),
    y: playerY,
    vx: 0,
    vy: -2,
    life: 25,
    text: "💥"
  });

  updateHUD();

  if (lives <= 0) showReviveRoulette();
}

function endGame() {
  running = false;
  gameOver = true;

  if (score > bestScore) bestScore = Math.floor(score);

  saveData();

  document.getElementById("finalScore").textContent = Math.floor(score);
  document.getElementById("finalRecord").textContent = bestScore;
  document.getElementById("finalCoins").textContent = coins;
  document.getElementById("gameOverScreen").classList.remove("hidden");

  updateHUD();
}

function drawBackground() {
  const gradient = ctx.createLinearGradient(0, 0, 0, H);
  gradient.addColorStop(0, "#182c45");
  gradient.addColorStop(1, "#42617b");

  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, W, H);

  // Prédios nas laterais.
  const buildingWidth = Math.max(25, (W - roadBounds().width) / 4);

  for (let i = 0; i < 8; i++) {
    const side = i % 2 === 0 ? 0 : 1;
    const x = side === 0
      ? i * buildingWidth * 0.5
      : W - (i + 1) * buildingWidth * 0.5;

    const bh = H * (0.15 + (i % 4) * 0.06);

    ctx.fillStyle = i % 2 ? "#26394e" : "#30485f";
    ctx.fillRect(x, H * 0.25 - bh * 0.2, buildingWidth, bh + H * 0.2);

    ctx.fillStyle = "#f9d976";

    for (let wy = H * 0.28; wy < H * 0.45; wy += 24) {
      for (let wx = x + 6; wx < x + buildingWidth - 4; wx += 16) {
        ctx.fillRect(wx, wy, 6, 9);
      }
    }
  }

  const road = roadBounds();

  ctx.fillStyle = "#343a46";
  ctx.fillRect(road.left, 0, road.width, H);

  ctx.fillStyle = "#515a68";
  ctx.fillRect(road.left, 0, 5, H);
  ctx.fillRect(road.right - 5, 0, 5, H);

  // Linhas divisórias da pista.
  ctx.strokeStyle = "#d6dbe2";
  ctx.lineWidth = 3;
  ctx.setLineDash([28, 25]);
  ctx.lineDashOffset = -distance % 53;

  for (let i = 1; i < laneCount; i++) {
    const x = road.left + road.width * i / laneCount;
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, H);
    ctx.stroke();
  }

  ctx.setLineDash([]);
}
const playerImage = new Image();
playerImage.src = "personagem.png";

function drawPlayer() {
  const x = laneX(playerLane);
  const y = playerY;

  const playerWidth = Math.min(W * 0.19, 90);
  const playerHeight = playerWidth * 1.5;

  // Pisca enquanto estiver atordoado.
  if (stunned && Math.floor(Date.now() / 100) % 2 === 0) {
    ctx.globalAlpha = 0.25;
  }

  if (playerImage.complete && playerImage.naturalWidth > 0) {
    ctx.drawImage(
      playerImage,
      x - playerWidth / 2,
      y - playerHeight / 2,
      playerWidth,
      playerHeight
    );
  } else {
    ctx.fillStyle = stunned ? "#ff3030" : "#ff3030";
    ctx.fillRect(x - 15, y - 25, 30, 50);
  }

  // Evita que a transparência afete outros desenhos.
  ctx.globalAlpha = 1;
}



function drawObstacles() {
  obstacles.forEach(obstacle => {
    const x = laneX(obstacle.lane);
    const y = obstacle.y;

    if (obstacle.type === "barrier") {
      ctx.fillStyle = "#ff4b42";
      ctx.fillRect(x - 23, y - 18, 46, 36);

      ctx.fillStyle = "#ffffff";
      ctx.fillRect(x - 20, y - 9, 40, 7);
      ctx.fillRect(x - 20, y + 5, 40, 7);
    } else if (obstacle.type === "train") {
      ctx.fillStyle = "#357bd1";
      ctx.fillRect(x - 27, y - 34, 54, 68);

      ctx.fillStyle = "#9be7ff";
      ctx.fillRect(x - 18, y - 23, 36, 18);

      ctx.fillStyle = "#17243a";
      ctx.fillRect(x - 20, y + 14, 12, 12);
      ctx.fillRect(x + 8, y + 14, 12, 12);
    } else {
      ctx.fillStyle = "#ffe66d";
      ctx.fillRect(x - 22, y - 20, 44, 40);

      ctx.fillStyle = "#303849";
      ctx.font = "bold 22px Arial";
      ctx.textAlign = "center";
      ctx.fillText("!", x, y + 8);
    }
  });
}

function drawCoins() {
  coinItems.forEach(coin => {
    if (coin.taken) return;

    const x = laneX(coin.lane);

    ctx.fillStyle = "#ffcc33";
    ctx.beginPath();
    ctx.arc(x, coin.y, 11, 0, Math.PI * 2);
    ctx.fill();

    ctx.strokeStyle = "#fff1a8";
    ctx.lineWidth = 3;
    ctx.stroke();

    ctx.fillStyle = "#9a6100";
    ctx.font = "bold 13px Arial";
    ctx.textAlign = "center";
    ctx.fillText("$", x, coin.y + 5);
  });
}

function drawParticles() {
  particles.forEach(p => {
    if (p.text) {
      ctx.font = "24px Arial";
      ctx.textAlign = "center";
      ctx.fillText(p.text, p.x, p.y);
    } else {
      ctx.fillStyle = "#ffe66d";
      ctx.beginPath();
      ctx.arc(p.x, p.y, 3, 0, Math.PI * 2);
      ctx.fill();
    }
  });
}

function update(delta) {
  if (!running) return;

  // Durante o stun, o jogo roda a 35% da velocidade normal.
  const timeScale = slowMotion ? 0.35 : 1;
  const factor = Math.min(delta / 16.67, 2.5) * timeScale;

  speed = Math.min(12, 5 + score / 500);
  distance += speed * factor;
  score += 0.12 * multiplier * factor;

  spawnTimer += factor;
  coinTimer += factor;

  if (spawnTimer > Math.max(22, 55 - speed * 2)) {
    spawnObstacle();
    spawnTimer = 0;
  }

  if (coinTimer > 40) {
    spawnCoins();
    coinTimer = 0;
  }

  obstacles.forEach(obstacle => {
    obstacle.y += speed * factor;

    const closeY = Math.abs(obstacle.y - playerY) < 34;
    if (closeY && obstacle.lane === playerLane && !obstacle.hit) {
      hitObstacle(obstacle);
    }
  });

  coinItems.forEach(coin => {
    coin.y += speed * factor;

    const closeY = Math.abs(coin.y - playerY) < 28;
    if (closeY && coin.lane === playerLane && !coin.taken) {
      collectCoin(coin);
    }
  });

  obstacles = obstacles.filter(o => o.y < H + 100);
  coinItems = coinItems.filter(c => c.y < H + 50 && !c.taken);

  particles.forEach(p => {
    p.x += (p.vx || 0) * factor;
    p.y += (p.vy || 0) * factor;
    p.life -= factor;
  });

  particles = particles.filter(p => p.life > 0);

  updateHUD();
}

function draw() {
  drawBackground();
  drawCoins();
  drawObstacles();
  drawPlayer();
  drawParticles();

  // Fase atual.
  const phase = Math.floor(score / 250) + 1;
  ctx.fillStyle = "rgba(0,0,0,0.4)";
  ctx.fillRect(W / 2 - 55, 12, 110, 30);

  ctx.fillStyle = "#ffffff";
  ctx.font = "bold 15px Arial";
  ctx.textAlign = "center";
  ctx.fillText("FASE " + phase, W / 2, 32);
}

function gameLoop(timestamp) {
  const delta = lastTime ? timestamp - lastTime : 16.67;
  lastTime = timestamp;

  update(delta);
  draw();

  if (running) {
    animationId = requestAnimationFrame(gameLoop);
  }
}
function showReviveRoulette() {
  if (revivePending || gameOver) return;

  revivePending = true;
  running = false;
  cancelAnimationFrame(animationId);

  const roulette = document.createElement("div");

  roulette.style.cssText = `
    position:fixed;
    inset:0;
    background:rgba(0,0,0,0.92);
    display:flex;
    flex-direction:column;
    justify-content:center;
    align-items:center;
    z-index:9999;
    color:white;
    font-family:Arial,sans-serif;
    text-align:center;
    padding:20px;
    box-sizing:border-box;
  `;

  roulette.innerHTML = `
    <h1>🎰 CAÇA-NÍQUEL DA VIDA</h1>
    <p>Chance de reviver: ${reviveChance}%</p>

    <div style="
      display:flex;
      gap:10px;
      margin:20px 0;
    ">
      <div class="slot-reel">🍀</div>
      <div class="slot-reel">⭐</div>
      <div class="slot-reel">❤️</div>
    </div>

    <h2 id="rouletteResult">Girando...</h2>
  `;

  const style = document.createElement("style");
  style.textContent = `
    .slot-reel {
      width:75px;
      height:90px;
      display:flex;
      justify-content:center;
      align-items:center;
      background:linear-gradient(#fff,#ddd);
      border:5px solid gold;
      border-radius:14px;
      box-shadow:0 0 18px gold;
      font-size:48px;
      color:black;
    }

    @keyframes slotFlash {
      0%, 100% { transform:scale(1); }
      50% { transform:scale(1.08); }
    }

    .slot-reel.winner {
      animation:slotFlash .5s ease-in-out 3;
      box-shadow:0 0 25px #22c55e;
    }
  `;

  document.head.appendChild(style);
  document.body.appendChild(roulette);

  const reels = [...roulette.querySelectorAll(".slot-reel")];
  const symbols = ["🍀", "⭐", "❤️", "💀", "🪙"];
  const result = roulette.querySelector("#rouletteResult");

  // O resultado é decidido pela chance comprada.
  const won = Math.random() * 100 < reviveChance;

  const spin = setInterval(() => {
    reels.forEach(reel => {
      reel.textContent =
        symbols[Math.floor(Math.random() * symbols.length)];
    });
  }, 80);

  // Para os símbolos depois de 3 segundos.
  setTimeout(() => {
    clearInterval(spin);

    if (won) {
      reels.forEach(reel => reel.textContent = "❤️");
      reels.forEach(reel => reel.classList.add("winner"));

      result.textContent = "🎉 VOCÊ REVIVEU!";
      lives = 1;
      updateHUD();

      setTimeout(() => {
        roulette.remove();
        style.remove();

        revivePending = false;
        running = true;
        lastTime = performance.now();

        cancelAnimationFrame(animationId);
        animationId = requestAnimationFrame(gameLoop);
      }, 1500);

    } else {
      reels.forEach(reel => reel.textContent = "💀");
      result.textContent = "💀 NÃO FOI DESSA VEZ!";

      setTimeout(() => {
        roulette.remove();
        style.remove();

        revivePending = false;
        endGame();
      }, 1500);
    }
  }, 3000);
}
createInterface();
draw();

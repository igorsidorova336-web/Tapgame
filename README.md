<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Тапалка</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; user-select: none; }
    body {
      background: #0f0f1a;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      font-family: Arial, sans-serif;
      color: white;
    }
    .app {
      background: #1a1a2e;
      padding: 30px;
      border-radius: 30px;
      width: 100%;
      max-width: 350px;
      text-align: center;
    }
    .coin {
      font-size: 100px;
      cursor: pointer;
      display: inline-block;
      transition: 0.1s;
      margin: 10px 0;
    }
    .coin:active {
      transform: scale(0.8);
    }
    .score {
      font-size: 40px;
      font-weight: bold;
      color: #f5c842;
    }
    .energy {
      font-size: 16px;
      color: #aaa;
      margin: 5px 0;
    }
    button {
      background: #f5c842;
      border: none;
      padding: 14px;
      border-radius: 50px;
      font-weight: bold;
      font-size: 16px;
      width: 100%;
      margin-top: 10px;
      cursor: pointer;
      transition: 0.15s;
      color: #0f0f1a;
    }
    button:active {
      transform: scale(0.95);
    }
    .stats {
      display: flex;
      justify-content: space-between;
      background: #12121f;
      padding: 10px;
      border-radius: 16px;
      margin: 10px 0;
      font-size: 14px;
    }
    .stats span { color: #f5c842; font-weight: bold; }
  </style>
</head>
<body>
<div class="app">
  <h1>💰 КЛИКЕР</h1>
  <div class="score" id="score">0</div>
  <div class="coin" id="coin">🪙</div>
  <div class="energy">⚡ Энергия: <span id="energy">100</span>%</div>
  <div class="stats">
    <div>👆 Сила: <span id="tapPower">1</span></div>
    <div>📈 Уровень: <span id="level">1</span></div>
  </div>
  <button id="upgradeBtn">🔧 Улучшить (стоит 50)</button>
  <div style="font-size:12px; opacity:0.3; margin-top:15px;">Просто нажми на монетку!</div>
</div>

<script>
  // ========== ИГРА ==========
  // Загружаем сохранённые данные
let savedData = localStorage.getItem('tapGameData');
if (savedData) {
  try {
    savedData = JSON.parse(savedData);
    score = savedData.score || 0;
    energy = savedData.energy || 100;
    tapPower = savedData.tapPower || 1;
    level = savedData.level || 1;
  } catch(e) {
    score = 0;
    energy = 100;
    tapPower = 1;
    level = 1;
  }
} else {
  score = 0;
  energy = 100;
  tapPower = 1;
  level = 1;
}
  let level = 1;
  const MAX_ENERGY = 100;

  const scoreEl = document.getElementById('score');
  const energyEl = document.getElementById('energy');
  const tapPowerEl = document.getElementById('tapPower');
  const levelEl = document.getElementById('level');
  const coin = document.getElementById('coin');
  const upgradeBtn = document.getElementById('upgradeBtn');

  function updateUI() {
    scoreEl.textContent = Math.floor(score);
    energyEl.textContent = Math.floor(energy);
    tapPowerEl.textContent = tapPower;
    levelEl.textContent = level;
    upgradeBtn.textContent = '🔧 Улучшить (стоит ' + (level * 50) + ')';
  }

  // Клик по монетке
  function handleTap(e) {
    e.preventDefault();
    if (energy < 1) {
      alert('⛔ Нет энергии! Подожди 3 секунды.');
      return;
    }
    energy -= 1;
    score += tapPower;
    coin.style.transform = 'scale(0.7)';
    setTimeout(() => { coin.style.transform = 'scale(1)'; }, 100);
    updateUI();
  }

  // Восстановление энергии
  setInterval(() => {
    if (energy < MAX_ENERGY) {
      energy = Math.min(energy + 2, MAX_ENERGY);
      updateUI();
    }
  }, 3000);

  // Улучшение
  function buyUpgrade() {
    const cost = level * 50;
    if (score < cost) {
      alert('❌ Нужно ' + cost + ' монет!');
      return;
    }
    score -= cost;
    tapPower += 1;
    level += 1;
    updateUI();
  }

  // ========== СОБЫТИЯ ==========
  coin.addEventListener('click', handleTap);
  coin.addEventListener('touchstart', function(e) {
    e.preventDefault();
    handleTap(e);
  }, { passive: false });

  upgradeBtn.addEventListener('click', buyUpgrade);

  // Запуск
  updateUI();
  // ========== СОХРАНЕНИЕ ==========
function saveGame() {
  const data = {
    score: score,
    energy: energy,
    tapPower: tapPower,
    level: level
  };
  localStorage.setItem('tapGameData', JSON.stringify(data));
}

// Сохраняем при каждом изменении
function updateUI() {
  scoreEl.textContent = Math.floor(score);
  energyEl.textContent = Math.floor(energy);
  tapPowerEl.textContent = tapPower;
  levelEl.textContent = level;
  upgradeBtn.textContent = '🔧 Улучшить (стоит ' + (level * 50) + ')';
  saveGame(); // 👈 добавляем автосохранение
}

// Сохраняем при закрытии
window.addEventListener('beforeunload', saveGame);
</script>
</body>
</html>

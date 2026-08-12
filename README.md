 html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Tap Game</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #0f0f1a;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      font-family: system-ui, -apple-system, sans-serif;
      color: white;
      touch-action: manipulation;
    }
    .app {
      background: #1a1a2e;
      padding: 30px 20px 40px;
      border-radius: 32px;
      width: 100%;
      max-width: 360px;
      text-align: center;
      box-shadow: 0 20px 40px rgba(0,0,0,0.6);
    }
    h1 {
      font-size: 22px;
      opacity: 0.7;
      margin-bottom: 5px;
    }
    .score {
      font-size: 52px;
      font-weight: 800;
      background: linear-gradient(135deg, #f5af19, #f12711);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      margin: 5px 0;
    }
    .coin {
      font-size: 100px;
      margin: 15px 0;
      cursor: pointer;
      transition: transform 0.08s ease;
      user-select: none;
      display: inline-block;
    }
    .coin:active {
      transform: scale(0.75);
    }
    .energy {
      background: #2a2a40;
      padding: 8px 16px;
      border-radius: 40px;
      display: inline-block;
      margin: 5px 0 15px;
      font-size: 16px;
      font-weight: 600;
    }
    .energy span { color: #4fc3f7; }
    .stats {
      display: flex;
      justify-content: space-between;
      background: #12121f;
      padding: 12px 18px;
      border-radius: 16px;
      margin: 15px 0;
      font-size: 14px;
    }
    .stats div span { color: #f5c842; font-weight: bold; }
    button {
      background: #f5c842;
      border: none;
      padding: 14px 20px;
      border-radius: 50px;
      font-weight: 700;
      font-size: 16px;
      width: 100%;
      cursor: pointer;
      transition: 0.15s;
      color: #0f0f1a;
    }
    button:active { transform: scale(0.95); }
    .info {
      font-size: 12px;
      opacity: 0.4;
      margin-top: 20px;
    }
  </style>
</head>
<body>
<div class="app">
  <h1>💰 КЛИКЕР</h1>
  <div class="score" id="score">0</div>
  
  <div class="coin" id="coin">🪙</div>
  
  <div class="energy">⚡ Энергия: <span id="energy">100</span>%</div>
  
  <div class="stats">
    <div style="margin: 10px 0;">
  <button id="bonusBtn" style="background: linear-gradient(135deg, #f7971e, #ffd200); padding: 12px 20px; border-radius: 50px; border: none; font-weight: 700; font-size: 15px; width: 100%; color: #0f0f1a;">
    🎁 Забрать бонус
  </button>
  <div style="font-size: 13px; opacity: 0.6; margin-top: 5px;">
    Следующий бонус через: <span id="bonusTimer">--:--:--</span>
  </div>
</div>
    <div>👆 Сила: <span id="tapPower">1</span></div>
    <div>📈 Уровень: <span id="level">1</span></div>
  </div>
  
  <button id="upgradeBtn">🔧 Улучшить тап (стоит 50)</button>
  <div class="info">⬆️ Автосохранение каждые 5 секунд</div>
</div>

<script>
  // ---------- ИГРОВАЯ ЛОГИКА ----------
  let score = 0;
  let energy = 100;
  let tapPower = 1;
  let level = 1;
  const MAX_ENERGY = 100;

  const scoreEl = document.getElementById('score');
  const energyEl = document.getElementById('energy');
  const tapPowerEl = document.getElementById('tapPower');
  const levelEl = document.getElementById('level');
  const coin = document.getElementById('coin');
  const upgradeBtn = document.getElementById('upgradeBtn');

  // Загружаем сохранённый прогресс
  function loadGame() {
    try {
      const saved = localStorage.getItem('tapGameData');
      if (saved) {
        const data = JSON.parse(saved);
        score = data.score || 0;
        energy = data.energy !== undefined ? data.energy : 100;
        tapPower = data.tapPower || 1;
        level = data.level || 1;
      }
    } catch(e) {}
    updateUI();
  }

  // Сохраняем прогресс
  function saveGame() {
    const data = { score, energy, tapPower, level };
    localStorage.setItem('tapGameData', JSON.stringify(data));
  }

  // Обновляем экран
  function updateUI() {
    scoreEl.textContent = Math.floor(score);
    energyEl.textContent = Math.floor(energy);
    tapPowerEl.textContent = tapPower;
    levelEl.textContent = level;
    upgradeBtn.textContent = `🔧 Улучшить тап (стоит ${level * 50})`;
  }

  // Клик по монетке
  function handleTap(e) {
    if (energy < 1) {
      // виброотклик, если есть
      if (navigator.vibrate) navigator.vibrate(30);
      alert('⛔ Нет энергии! Подожди 3 секунды.');
      return;
    }

    energy -= 1;
    score += tapPower;

    // Анимация нажатия
    coin.style.transform = 'scale(0.7)';
    setTimeout(() => coin.style.transform = 'scale(1)', 90);

    // Всплывающее "+N"
    const popup = document.createElement('div');
    popup.textContent = `+${tapPower}`;
    popup.style.cssText = `
      position: fixed;
      color: #f5c842;
      font-size: 32px;
      font-weight: 900;
      pointer-events: none;
      left: ${e.clientX - 20}px;
      top: ${e.clientY - 30}px;
      transition: all 0.5s ease-out;
      opacity: 1;
      text-shadow: 0 0 10px rgba(245,200,66,0.5);
    `;
    document.body.appendChild(popup);
    // Анимация улетания вверх
    requestAnimationFrame(() => {
      popup.style.transform = 'translateY(-70px)';
      popup.style.opacity = '0';
    });
    setTimeout(() => popup.remove(), 500);

    // Виброотклик (если поддерживается)
    if (navigator.vibrate) navigator.vibrate(10);

    updateUI();
  }

  // Восстановление энергии
  setInterval(() => {
    if (energy < MAX_ENERGY) {
      energy = Math.min(energy + 2, MAX_ENERGY);
      updateUI();
    }
  }, 3000);

  // Покупка улучшения
  function buyUpgrade() {
    const cost = level * 50;
    if (score < cost) {
      alert(`❌ Нужно ${cost} монет! У тебя ${Math.floor(score)}.`);
      return;
    }
    score -= cost;
    tapPower += 1;
    level += 1;
    updateUI();
    // Визуальная обратная связь
    upgradeBtn.style.transform = 'scale(0.9)';
    setTimeout(() => upgradeBtn.style.transform = 'scale(1)', 100);
  }

  // ---- ПРИВЯЗКА СОБЫТИЙ ----
  coin.addEventListener('click', handleTap);
  upgradeBtn.addEventListener('click', buyUpgrade);

  // ---- АВТОСОХРАНЕНИЕ ----
  setInterval(saveGame, 5000);

  // ---- ЗАГРУЗКА ПРИ СТАРТЕ ----
  loadGame();

  // Сохраняем при закрытии страницы
  window.addEventListener('beforeunload', saveGame);
  // ---------- ЕЖЕДНЕВНЫЙ БОНУС ----------
const bonusBtn = document.getElementById('bonusBtn');
const bonusTimer = document.getElementById('bonusTimer');

// Загружаем данные о бонусе
let lastBonusDate = localStorage.getItem('lastBonusDate') || null;
let bonusStreak = parseInt(localStorage.getItem('bonusStreak')) || 0;

// Проверяем, можно ли забрать бонус
function canClaimBonus() {
    if (!lastBonusDate) return true;
    const last = new Date(parseInt(lastBonusDate));
    const now = new Date();
    const diff = now - last;
    const hoursPassed = diff / (1000 * 60 * 60);
    return hoursPassed >= 24;
}

// Считаем бонусные монеты
function getBonusAmount() {
    return 50 + bonusStreak * 30; // 50, 80, 110, 140...
}

// Обновляем таймер до следующего бонуса
function updateBonusTimer() {
    if (!lastBonusDate) {
        bonusTimer.textContent = 'Готово!';
        return;
    }
    const last = new Date(parseInt(lastBonusDate));
    const next = new Date(last.getTime() + 24 * 60 * 60 * 1000);
    const now = new Date();
    const diff = next - now;
    
    if (diff <= 0) {
        bonusTimer.textContent = '🎯 Готово!';
        return;
    }
    
    const hours = Math.floor(diff / (1000 * 60 * 60));
    const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
    const seconds = Math.floor((diff % (1000 * 60)) / 1000);
    bonusTimer.textContent = `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
}

// Забрать бонус
function claimBonus() {
    if (!canClaimBonus()) {
        alert('⏳ Бонус ещё не готов! Подожди 24 часа.');
        return;
    }
    
    const bonus = getBonusAmount();
    const streak = bonusStreak + 1;
    
    score += bonus;
    bonusStreak = streak;
    lastBonusDate = Date.now().toString();
    
    // Сохраняем
    localStorage.setItem('lastBonusDate', lastBonusDate);
    localStorage.setItem('bonusStreak', bonusStreak.toString());
    
    // Красивое уведомление
    alert(`🎉 День ${streak} подряд! Ты получил ${bonus} монет!`);
    
    // Анимация кнопки
    bonusBtn.style.transform = 'scale(0.9)';
    setTimeout(() => bonusBtn.style.transform = 'scale(1)', 150);
    
    updateUI();
    updateBonusUI();
}

// Обновляем интерфейс бонуса
function updateBonusUI() {
    if (canClaimBonus()) {
        bonusBtn.textContent = '🎁 Забрать бонус';
        bonusBtn.style.background = 'linear-gradient(135deg, #f7971e, #ffd200)';
        bonusBtn.disabled = false;
    } else {
        bonusBtn.textContent = '⏳ Бонус недоступен';
        bonusBtn.style.background = '#2a2a40';
        bonusBtn.style.color = '#666';
        bonusBtn.disabled = true;
    }
    updateBonusTimer();
}

// Запускаем обновление таймера каждую секунду
setInterval(updateBonusTimer, 1000);

// Привязываем кнопку
bonusBtn.addEventListener('click', claimBonus);

// Загружаем состояние при старте
updateBonusUI();
</script>
</body>
</html>

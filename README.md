<!DOCTYPE html>
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
    h1 { font-size: 22px; opacity: 0.7; margin-bottom: 5px; }
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
      user-select: none;
      display: inline-block;
      transition: transform 0.08s ease;
    }
    .coin:active { transform: scale(0.75); }
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

    /* Таблица лидеров */
    .leaderboard-entry {
      display: flex;
      justify-content: space-between;
      padding: 4px 0;
      border-bottom: 1px solid #1a1a2e;
      font-size: 13px;
    }
    .leaderboard-entry .name { text-align: left; }
    .leaderboard-entry .score { color: #f5c842; font-weight: bold; }
    input {
      flex: 1;
      padding: 8px 12px;
      border-radius: 30px;
      border: none;
      background: #2a2a40;
      color: white;
      font-size: 14px;
      outline: none;
    }
    input::placeholder { color: #666; }
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
  <button id="upgradeBtn">🔧 Улучшить тап (стоит 50)</button>

  <!-- ТАБЛИЦА ЛИДЕРОВ -->
  <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
    <div style="font-size:14px;font-weight:bold;margin-bottom:10px;">🏆 Таблица лидеров</div>
    <div style="display:flex;gap:6px;margin-bottom:8px;">
      <input id="playerNameInput" type="text" placeholder="Твоё имя" value="Игрок">
      <button id="saveScoreBtn" style="width:auto;padding:8px 16px;font-size:13px;background:#4fc3f7;margin-top:0;">Сохранить</button>
    </div>
    <div id="leaderboardList" style="text-align:left;"></div>
  </div>

  <div class="info">⬆️ Автосохранение каждые 5 секунд</div>
</div>

<script>
  // ==================== ПЕРЕМЕННЫЕ ====================
  // ==================== ЕЖЕДНЕВНЫЙ БОНУС ====================
let lastBonusDate = localStorage.getItem('lastBonusDate') || null;
let bonusStreak = parseInt(localStorage.getItem('bonusStreak')) || 0;

const bonusBtn = document.getElementById('bonusBtn');
const bonusTimer = document.getElementById('bonusTimer');

function getBonusAmount() {
  return 50 + bonusStreak * 30; // 50, 80, 110, 140...
}

function canClaimBonus() {
  if (!lastBonusDate) return true;
  var last = new Date(parseInt(lastBonusDate));
  var now = new Date();
  var diff = now - last;
  var hoursPassed = diff / (1000 * 60 * 60);
  return hoursPassed >= 24;
}

function updateBonusTimer() {
  if (!lastBonusDate) {
    bonusTimer.textContent = 'Готово!';
    return;
  }
  var last = new Date(parseInt(lastBonusDate));
  var next = new Date(last.getTime() + 24 * 60 * 60 * 1000);
  var now = new Date();
  var diff = next - now;

  if (diff <= 0) {
    bonusTimer.textContent = '🎯 Готово!';
    return;
  }

  var hours = Math.floor(diff / (1000 * 60 * 60));
  var minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  var seconds = Math.floor((diff % (1000 * 60)) / 1000);
  bonusTimer.textContent = 
    String(hours).padStart(2, '0') + ':' + 
    String(minutes).padStart(2, '0') + ':' + 
    String(seconds).padStart(2, '0');
}

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

function claimBonus() {
  if (!canClaimBonus()) {
    alert('⏳ Бонус ещё не готов! Подожди 24 часа.');
    return;
  }

  var bonus = getBonusAmount();
  var streak = bonusStreak + 1;

  score += bonus;
  bonusStreak = streak;
  lastBonusDate = Date.now().toString();

  localStorage.setItem('lastBonusDate', lastBonusDate);
  localStorage.setItem('bonusStreak', bonusStreak.toString());

  alert('🎉 День ' + streak + ' подряд! Ты получил ' + formatNumber(bonus) + ' монет!');

  bonusBtn.style.transform = 'scale(0.9)';
  setTimeout(function() { bonusBtn.style.transform = 'scale(1)'; }, 150);

  updateUI();
  updateBonusUI();
  saveGame();
}

// Кнопка бонуса
if (bonusBtn) {
  bonusBtn.addEventListener('click', claimBonus);
}

// Обновление таймера каждую секунду
setInterval(updateBonusTimer, 1000);
  let score = 0;
  let energy = 100;
  let tapPower = 1;
  let level = 1;
  const MAX_ENERGY = 100;

  let leaderboard = [];
  let playerName = 'Игрок';

  const scoreEl = document.getElementById('score');
  const energyEl = document.getElementById('energy');
  const tapPowerEl = document.getElementById('tapPower');
  const levelEl = document.getElementById('level');
  const coin = document.getElementById('coin');
  const upgradeBtn = document.getElementById('upgradeBtn');
<!-- ЕЖЕДНЕВНЫЙ БОНУС -->
<div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
  <div style="font-size:14px;font-weight:bold;margin-bottom:8px;">🎁 Ежедневный бонус</div>
  <button id="bonusBtn" style="background:linear-gradient(135deg, #f7971e, #ffd200); padding:12px 16px; border-radius:50px; border:none; font-weight:700; font-size:15px; width:100%; color:#0f0f1a; margin-top:0;">
    🎁 Забрать бонус
  </button>
  <div style="font-size:12px; opacity:0.5; margin-top:5px;">
    Следующий бонус через: <span id="bonusTimer">--:--:--</span>
  </div>
</div>
  // ==================== ТАБЛИЦА ЛИДЕРОВ ====================
  function formatNumber(num) {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
  }

  function renderLeaderboard() {
    const list = document.getElementById('leaderboardList');
    if (!list) return;
    if (leaderboard.length === 0) {
      list.innerHTML = '<div style="opacity:0.4;text-align:center;padding:10px;font-size:13px;">Нет записей. Стань первым! 🚀</div>';
      return;
    }
    let html = '';
    for (var i = 0; i < leaderboard.length; i++) {
      var entry = leaderboard[i];
      var medal = i === 0 ? '🥇' : i === 1 ? '🥈' : i === 2 ? '🥉' : (i + 1) + '.';
      var isYou = entry.name === playerName ? ' ⭐' : '';
      html += '<div class="leaderboard-entry"><span class="name">' + medal + ' ' + entry.name + isYou + '</span><span class="score">' + formatNumber(entry.score) + ' 🪙</span></div>';
    }
    list.innerHTML = html;
  }

  function saveLeaderboard() {
    localStorage.setItem('leaderboard', JSON.stringify(leaderboard));
  }

  function loadLeaderboard() {
    var saved = localStorage.getItem('leaderboard');
    if (saved) {
      try { leaderboard = JSON.parse(saved); } catch(e) { leaderboard = []; }
    } else { leaderboard = []; }
    renderLeaderboard();
  }

  // ==================== СОХРАНЕНИЕ ====================
 function saveGame() {
    var data = {
      lastBonusDate: lastBonusDate,
bonusStreak: bonusStreak
      score: score,
      energy: energy,
      tapPower: tapPower,
      level: level,
      leaderboard: leaderboard
    };
    localStorage.setItem('tapGameData', JSON.stringify(data)) ;
  }

  function loadGame() {
    try {
      var saved = localStorage.getItem('tapGameData');
      if (saved) {
        var data = JSON.parse(saved);
        score = data.score || 0;
        energy = data.energy !== undefined ? data.energy : 100;
        tapPower = data.tapPower || 1;
        level = data.level || 1;
        if (data.leaderboard) leaderboard = data.leaderboard;
        if (data.lastBonusDate) lastBonusDate = data.lastBonusDate;
if (data.bonusStreak) bonusStreak = data.bonusStreak;
      }
    } catch(e) {}
    updatebonusUI();
    updateUI();
    renderLeaderboard();
  }

  // ==================== UI ====================
  function updateUI() {
    scoreEl.textContent = Math.floor(score);
    energyEl.textContent = Math.floor(energy);
    tapPowerEl.textContent = tapPower;
    levelEl.textContent = level;
    upgradeBtn.textContent = '🔧 Улучшить тап (стоит ' + (level * 50) + ')';
  }

  // ==================== ТАП ====================
  function handleTap(e) {
    e.preventDefault();
    if (energy < 1) {
      if (navigator.vibrate) navigator.vibrate(30);
      alert('⛔ Нет энергии! Подожди 3 секунды.');
      return;
    }
    energy -= 1;
    score += tapPower;
    coin.style.transform = 'scale(0.7)';
    setTimeout(function() { coin.style.transform = 'scale(1)'; }, 90);
    if (navigator.vibrate) navigator.vibrate(10);
    updateUI();
  }

  // ==================== ЭНЕРГИЯ ====================
  setInterval(function() {
    if (energy < MAX_ENERGY) {
      energy = Math.min(energy + 2, MAX_ENERGY);
      updateUI();
    }
  }, 3000);

  // ==================== УЛУЧШЕНИЕ ====================
  function buyUpgrade() {
    var cost = level * 50;
    if (score < cost) {
      alert('❌ Нужно ' + cost + ' монет! У тебя ' + Math.floor(score) + '.');
      return;
    }
    score -= cost;
    tapPower += 1;
    level += 1;
    updateUI();
    saveGame();
  }

  // ==================== СОБЫТИЯ ====================
  coin.addEventListener('click', handleTap);
  coin.addEventListener('touchend', function(e) {
    e.preventDefault();
    handleTap(e);
  }, { passive: false });

  upgradeBtn.addEventListener('click', buyUpgrade);

  // Имя игрока
  var nameInput = document.getElementById('playerNameInput');
  if (nameInput) {
    nameInput.value = playerName;
    nameInput.addEventListener('input', function() {
      playerName = this.value || 'Игрок';
      localStorage.setItem('playerName', playerName);
      renderLeaderboard();
    });
  }

  // Кнопка сохранить в таблицу
  document.getElementById('saveScoreBtn').addEventListener('click', function() {
    var currentScore = Math.floor(score);
    if (currentScore === 0) {
      alert('Сначала набери монеты!');
      return;
    }
    var existing = -1;
    for (var i = 0; i < leaderboard.length; i++) {
      if (leaderboard[i].name === playerName) {
        existing = i;
        break;
      }
    }
    if (existing !== -1) {
      if (currentScore > leaderboard[existing].score) {
        leaderboard[existing].score = currentScore;
      } else {
        alert('У тебя уже есть рекорд: ' + formatNumber(leaderboard[existing].score) + ' монет. Новый счёт ' + formatNumber(currentScore) + ' — не побит.');
        return;
      }
    } else {
      leaderboard.push({ name: playerName, score: currentScore });
    }
    leaderboard.sort(function(a, b) { return b.score - a.score; });
    if (leaderboard.length > 10) leaderboard = leaderboard.slice(0, 10);
    saveLeaderboard();
    saveGame();
    renderLeaderboard();
    alert('✅ Результат сохранён в таблицу лидеров!');
  });

  // ==================== АВТОСОХРАНЕНИЕ ====================
  setInterval(saveGame, 5000);
  window.addEventListener('beforeunload', saveGame);

  // ==================== СТАРТ ====================
  loadGame();
</script>
</body>
</html>

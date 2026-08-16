<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Tap Game</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
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
      padding: 20px;
      border-radius: 32px;
      width: 100%;
      max-width: 380px;
      text-align: center;
      box-shadow: 0 20px 40px rgba(0,0,0,0.6);
      max-height: 98vh;
      overflow-y: auto;
      transition: background 0.3s ease;
      background-size: cover;
      background-position: center;
    }
    h1 { font-size: 22px; opacity: 0.7; margin-bottom: 5px; }
    .score {
      font-size: 48px;
      font-weight: 800;
      background: linear-gradient(135deg, #f5af19, #f12711);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      margin: 5px 0;
    }
    #coin3d {
      width: 150px;
      height: 150px;
      margin: 10px auto;
      cursor: pointer;
    }
    .coin { display: none; }
    .energy {
      background: #2a2a40;
      padding: 6px 16px;
      border-radius: 40px;
      display: inline-block;
      margin: 5px 0 10px;
      font-size: 15px;
      font-weight: 600;
    }
    .energy span { color: #4fc3f7; }
    .stats {
      display: flex;
      justify-content: space-between;
      background: #12121f;
      padding: 10px 16px;
      border-radius: 16px;
      margin: 10px 0;
      font-size: 13px;
    }
    .stats div span { color: #f5c842; font-weight: bold; }
    button {
      background: #f5c842;
      border: none;
      padding: 12px 16px;
      border-radius: 50px;
      font-weight: 700;
      font-size: 15px;
      width: 100%;
      cursor: pointer;
      transition: 0.15s;
      color: #0f0f1a;
      margin-top: 6px;
    }
    button:active { transform: scale(0.95); }
    button:disabled { opacity: 0.4; transform: none; }
    .info { font-size: 11px; opacity: 0.3; margin-top: 15px; }

    .skin-item {
      width: 56px;
      height: 66px;
      background: #1a1a2e;
      border-radius: 12px;
      display: inline-flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      border: 2px solid transparent;
      transition: 0.15s;
      font-size: 26px;
      padding: 4px;
      margin: 3px;
    }
    .skin-item:active { transform: scale(0.9); }
    .skin-item.owned { border-color: #4caf50; }
    .skin-item.active { border-color: #f5c842; box-shadow: 0 0 15px rgba(245,200,66,0.3); }
    .skin-item.locked { opacity: 0.4; }
    .skin-price { font-size: 9px; color: #aaa; margin-top: 2px; }
    .skin-badge { font-size: 7px; background: #ffd700; color: #000; padding: 1px 6px; border-radius: 10px; font-weight: bold; margin-top: 2px; }
    .skin-item.vip { border-color: #ff6b6b; background: #1a0a0a; }
    .skin-item.legendary { border-color: #ffd700; background: #1a1500; box-shadow: 0 0 20px rgba(255,215,0,0.15); }
    .skin-item.legendary .skin-price { color: #ffd700; font-weight: bold; }

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
    #bonusBtn:disabled { opacity: 0.4; transform: none; }
    .economy-box {
      background: #1a1a2e;
      border-radius: 12px;
      padding: 10px;
      margin-bottom: 8px;
    }
    .economy-box .row {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .economy-box .title { font-size: 13px; font-weight: bold; }
    .economy-box .sub { font-size: 11px; opacity: 0.6; }
    .economy-box .mini { font-size: 11px; opacity: 0.4; }
    .btn-small {
      width: auto;
      padding: 6px 14px;
      font-size: 12px;
      margin-top: 0;
    }
    .btn-green { background: #4caf50; color: white; }
    .btn-red { background: #ff6b6b; color: white; }
    .btn-gold { background: #ffd700; color: #000; }
    .btn-gold:disabled { background: #2a2a40; color: #666; }
    .tab-btn {
      flex: 1;
      padding: 8px;
      font-size: 13px;
      margin: 0;
      border-radius: 12px;
      border: none;
      font-weight: bold;
      cursor: pointer;
      transition: 0.2s;
    }
    .tab-btn.active {
      background: #f5c842;
      color: #0f0f1a;
    }
    .tab-btn.inactive {
      background: #2a2a40;
      color: #888;
    }
    .achievement-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 6px 10px;
      margin-bottom: 4px;
      border-radius: 8px;
      border-left: 3px solid #2a2a40;
    }
    .achievement-item.done {
      background: #1a2e1a;
      border-left-color: #4caf50;
    }
    .achievement-item.undone {
      background: #1a1a2e;
      border-left-color: #2a2a40;
    }
    .achievement-item .left {
      display: flex;
      align-items: center;
      gap: 8px;
      text-align: left;
    }
    .achievement-item .left .icon { font-size: 20px; }
    .achievement-item .left .name { font-size: 12px; font-weight: bold; }
    .achievement-item .left .desc { font-size: 10px; opacity: 0.5; }
    .achievement-item .right { font-size: 11px; text-align: right; }
    .achievement-item .right .done-text { color: #4caf50; }
    .achievement-item .right .reward-text { color: #888; }
  </style>
</head>
<body>
<div class="app" id="app">
  <h1>💰 КЛИКЕР</h1>

  <!-- ВКЛАДКИ -->
  <div style="display:flex;gap:8px;margin:10px 0;background:#12121f;border-radius:16px;padding:6px;">
    <button class="tab-btn active" id="tabGame" onclick="switchToGame()">🎮 Игра</button>
    <button class="tab-btn inactive" id="tabSkins" onclick="switchToSkins()">🎨 Скины</button>
  </div>

  <!-- ИГРОВАЯ ВКЛАДКА -->
  <div id="gameTab">
    <div style="font-size:13px;opacity:0.5;margin-bottom:5px;">👥 Онлайн: <span id="onlineCount">0</span> игроков</div>
    <div class="score" id="score">0</div>
    <div id="coin3d"></div>
    <div class="coin" id="coin">🪙</div>
    <div class="energy">⚡ Энергия: <span id="energy">100</span>%</div>
    <div class="stats">
      <div>👆 Сила: <span id="tapPower">1</span></div>
      <div>📈 Уровень: <span id="level">1</span></div>
    </div>
    <button id="upgradeBtn">🔧 Улучшить тап (стоит 50)</button>

    <!-- АВТОКЛИКЕР -->
    <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
      <div class="row">
        <div>
          <div style="font-size:14px;font-weight:bold;">🤖 Автокликер</div>
          <div style="font-size:12px;opacity:0.6;">Уровень: <span id="autoLevel">0</span> (+<span id="autoIncome">0</span>/сек)</div>
        </div>
        <button id="autoBtn" class="btn-small" style="background:#4fc3f7;color:#000;">Купить (<span id="autoPrice">1000</span>🪙)</button>
      </div>
    </div>

    <!-- ЕЖЕДНЕВНЫЙ БОНУС -->
    <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
      <div style="font-size:14px;font-weight:bold;margin-bottom:8px;">🎁 Ежедневный бонус</div>
      <button id="bonusBtn" style="background:linear-gradient(135deg, #f7971e, #ffd200);padding:12px 16px;border-radius:50px;border:none;font-weight:700;font-size:15px;width:100%;color:#0f0f1a;margin-top:0;">🎁 Забрать бонус</button>
      <div style="font-size:12px;opacity:0.5;margin-top:5px;">Следующий бонус через: <span id="bonusTimer">--:--:--</span></div>
    </div>

    <!-- ЭКОНОМИКА -->
    <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
      <div style="font-size:14px;font-weight:bold;margin-bottom:8px;">💰 Экономика</div>

      <div class="economy-box">
        <div class="row">
          <div><div class="title">📈 Инвестиции</div><div class="sub">Доход: <span id="investIncome">0</span> 🪙/мин</div></div>
          <button id="investBtn" class="btn-small btn-green">Вложить (1000🪙)</button>
        </div>
        <div class="mini">Вложено: <span id="investAmount">0</span> 🪙</div>
      </div>

      <div class="economy-box">
        <div class="row">
          <div><div class="title">🎰 Рулетка</div><div class="sub">Шанс x2 или проигрыш</div></div>
          <button id="rouletteBtn" class="btn-small btn-red">Крутить (100🪙)</button>
        </div>
        <div class="mini" id="rouletteResult">Нажми "Крутить"</div>
      </div>

      <div class="economy-box">
        <div class="row">
          <div><div class="title">🏷️ Аукцион скинов</div><div class="sub">Торгуй скинами</div></div>
          <button id="auctionBtn" class="btn-small btn-gold">Торговать</button>
        </div>
        <div class="mini" id="auctionStatus">Сейчас торгуется: нет</div>
      </div>

      <div class="economy-box" style="margin-bottom:0;">
        <div class="row">
          <div><div class="title">💤 Офлайн-доход</div><div class="sub">Заработано: <span id="offlineEarned">0</span> 🪙</div></div>
          <button id="collectOfflineBtn" class="btn-small btn-gold">Забрать</button>
        </div>
        <div class="mini">Ты отсутствовал: <span id="offlineTime">0</span> мин</div>
      </div>
    </div>

    <!-- ДОСТИЖЕНИЯ -->
    <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
      <div style="font-size:14px;font-weight:bold;margin-bottom:8px;">🏆 Достижения</div>
      <div id="achievementsList" style="text-align:left;max-height:200px;overflow-y:auto;"></div>
      <div style="font-size:11px;opacity:0.4;margin-top:5px;">Выполняй задания и получай награды!</div>
    </div>

    <!-- ФОН -->
    <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
      <div style="font-size:14px;font-weight:bold;margin-bottom:8px;">🎨 Фон</div>
      <div style="display:flex;gap:6px;flex-wrap:wrap;justify-content:center;margin-bottom:8px;">
        <button onclick="setBackground('#0f0f1a')" style="width:30px;height:30px;border-radius:50%;background:#0f0f1a;border:2px solid #fff;padding:0;margin:0;"></button>
        <button onclick="setBackground('#1a1a2e')" style="width:30px;height:30px;border-radius:50%;background:#1a1a2e;border:2px solid #fff;padding:0;margin:0;"></button>
        <button onclick="setBackground('#16213e')" style="width:30px;height:30px;border-radius:50%;background:#16213e;border:2px solid #fff;padding:0;margin:0;"></button>
        <button onclick="setBackground('#0f3460')" style="width:30px;height:30px;border-radius:50%;background:#0f3460;border:2px solid #fff;padding:0;margin:0;"></button>
        <button onclick="setBackground('#2d132c')" style="width:30px;height:30px;border-radius:50%;background:#2d132c;border:2px solid #fff;padding:0;margin:0;"></button>
        <button onclick="setBackground('#1b4332')" style="width:30px;height:30px;border-radius:50%;background:#1b4332;border:2px solid #fff;padding:0;margin:0;"></button>
        <button onclick="setBackground('#4a1942')" style="width:30px;height:30px;border-radius:50%;background:#4a1942;border:2px solid #fff;padding:0;margin:0;"></button>
      </div>
      <div style="display:flex;gap:6px;">
        <button id="uploadBgBtn" style="width:auto;padding:8px 16px;font-size:12px;background:#4fc3f7;margin-top:0;border:none;border-radius:30px;font-weight:bold;color:#000;">📁 Из галереи</button>
        <button onclick="resetBackground()" style="width:auto;padding:8px 16px;font-size:12px;background:#ff6b6b;margin-top:0;border:none;border-radius:30px;font-weight:bold;color:white;">🔄 Сбросить</button>
      </div>
      <input type="file" id="bgFileInput" accept="image/*" style="display:none;">
    </div>

    <!-- ТАБЛИЦА ЛИДЕРОВ -->
    <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
      <div style="font-size:14px;font-weight:bold;margin-bottom:10px;">🏆 Таблица лидеров</div>
      <div style="display:flex;gap:6px;margin-bottom:8px;">
        <input id="playerNameInput" type="text" placeholder="Твоё имя" value="Игрок">
        <button id="saveScoreBtn" style="width:auto;padding:8px 16px;font-size:13px;background:#4fc3f7;margin-top:0;border:none;border-radius:30px;font-weight:bold;color:#000;">Сохранить</button>
      </div>
      <div id="leaderboardList" style="text-align:left;"></div>
    </div>

    <div class="info">⬆️ Автосохранение каждые 5 секунд</div>
  </div>

  <!-- ВКЛАДКА СКИНОВ -->
  <div id="skinsTab" style="display:none;">
    <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
      <div style="font-size:14px;font-weight:bold;margin-bottom:10px;">🎨 Все скины</div>
      <div id="skinShop" style="display:flex;gap:6px;justify-content:center;flex-wrap:wrap;"></div>
    </div>
  </div>
</div>

<script>
// ==================== ПЕРЕКЛЮЧЕНИЕ ВКЛАДОК ====================
function switchToGame() {
  document.getElementById('gameTab').style.display = 'block';
  document.getElementById('skinsTab').style.display = 'none';
  document.getElementById('tabGame').className = 'tab-btn active';
  document.getElementById('tabSkins').className = 'tab-btn inactive';
}
function switchToSkins() {
  document.getElementById('gameTab').style.display = 'none';
  document.getElementById('skinsTab').style.display = 'block';
  document.getElementById('tabSkins').className = 'tab-btn active';
  document.getElementById('tabGame').className = 'tab-btn inactive';
}

// ==================== ПЕРЕМЕННЫЕ ====================
let score = 0;
let energy = 100;
let tapPower = 1;
let level = 1;
const MAX_ENERGY = 100;

let leaderboard = [];
let playerName = 'Игрок';
let ownedSkins = ['gold'];
let activeSkin = 'gold';
let lastBonusDate = localStorage.getItem('lastBonusDate') || null;
let bonusStreak = parseInt(localStorage.getItem('bonusStreak')) || 0;
let autoLevel = parseInt(localStorage.getItem('autoLevel')) || 0;
const autoBasePrice = 1000;
let invested = parseInt(localStorage.getItem('invested')) || 0;
let investIncome = 0;
let lastOnlineTime = localStorage.getItem('lastOnlineTime') || Date.now().toString();
let offlineEarnings = parseInt(localStorage.getItem('offlineEarnings')) || 0;
let tapCount = parseInt(localStorage.getItem('tapCount')) || 0;
let rouletteWins = parseInt(localStorage.getItem('rouletteWins')) || 0;
let completedAchievements = JSON.parse(localStorage.getItem('completedAchievements')) || [];

const scoreEl = document.getElementById('score');
const energyEl = document.getElementById('energy');
const tapPowerEl = document.getElementById('tapPower');
const levelEl = document.getElementById('level');
const coin = document.getElementById('coin');
const upgradeBtn = document.getElementById('upgradeBtn');
const bonusBtn = document.getElementById('bonusBtn');
const bonusTimer = document.getElementById('bonusTimer');

// ==================== ФОРМАТИРОВАНИЕ ====================
function formatNumber(num) {
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
}

// ==================== СКИНЫ ====================
const SKINS = [
  { id: 'gold', emoji: '🪙', name: 'Золотая', price: 50, color: 0xf5c842 },
  { id: 'diamond', emoji: '💎', name: 'Алмазная', price: 150, color: 0xb9f2ff },
  { id: 'plasma', emoji: '🌀', name: 'Плазменная', price: 500, color: 0x00ffcc },
  { id: 'rainbow', emoji: '🌈', name: 'Радужная', price: 1500, color: 0xff6bff },
  { id: 'neon', emoji: '💜', name: 'Неоновая', price: 50000, color: 0xcc00ff },
  { id: 'crystal', emoji: '❄️', name: 'Хрустальная', price: 100000, color: 0x88ddff },
  { id: 'legend_gold', emoji: '👑', name: 'Королевская', price: 1000000, color: 0xffd700 },
  { id: 'legend_dark', emoji: '🌑', name: 'Тёмная звезда', price: 2500000, color: 0x2a0a2a },
  { id: 'legend_cosmic', emoji: '🌌', name: 'Космическая', price: 5000000, color: 0x4a00ff },
  { id: 'legend_god', emoji: '⚡', name: 'Божественная', price: 10000000, color: 0xffaa00 },
];

// ==================== ДОСТИЖЕНИЯ ====================
const ACHIEVEMENTS = [
  { id: 'first_tap', name: 'Первый шаг', desc: 'Сделай 100 тапов', icon: '👣', reward: 500, check: () => tapCount >= 100 },
  { id: 'tapper_1k', name: 'Тысячник', desc: 'Сделай 1 000 тапов', icon: '👆', reward: 1000, check: () => tapCount >= 1000 },
  { id: 'tapper_10k', name: 'Трудяга', desc: 'Сделай 10 000 тапов', icon: '💪', reward: 5000, check: () => tapCount >= 10000 },
  { id: 'tapper_100k', name: 'Тап-мастер', desc: 'Сделай 100 000 тапов', icon: '🔥', reward: 20000, check: () => tapCount >= 100000 },
  { id: 'tapper_1m', name: 'Легенда тапа', desc: 'Сделай 1 000 000 тапов', icon: '👑', reward: 100000, check: () => tapCount >= 1000000 },
  { id: 'rich_10k', name: 'Богач', desc: 'Накопи 10 000 монет', icon: '💰', reward: 2000, check: () => score >= 10000 },
  { id: 'rich_100k', name: 'Магнат', desc: 'Накопи 100 000 монет', icon: '💎', reward: 10000, check: () => score >= 100000 },
  { id: 'rich_1m', name: 'Миллионер', desc: 'Накопи 1 000 000 монет', icon: '🏦', reward: 50000, check: () => score >= 1000000 },
  { id: 'rich_10m', name: 'Легенда богатства', desc: 'Накопи 10 000 000 монет', icon: '🌟', reward: 200000, check: () => score >= 10000000 },
  { id: 'rich_100m', name: 'Финансовый гений', desc: 'Накопи 100 000 000 монет', icon: '🚀', reward: 500000, check: () => score >= 100000000 },
  { id: 'skin_collector_3', name: 'Коллекционер', desc: 'Купи 3 скина', icon: '🎨', reward: 3000, check: () => ownedSkins.length >= 3 },
  { id: 'skin_collector_5', name: 'Ценитель искусства', desc: 'Купи 5 скинов', icon: '🖼️', reward: 8000, check: () => ownedSkins.length >= 5 },
  { id: 'skin_collector_8', name: 'Знаток скинов', desc: 'Купи 8 скинов', icon: '🏛️', reward: 20000, check: () => ownedSkins.length >= 8 },
  { id: 'skin_legendary', name: 'Легендарный коллекционер', desc: 'Купи легендарный скин', icon: '👑', reward: 50000, check: () => ownedSkins.some(id => ['legend_gold','legend_dark','legend_cosmic','legend_god'].includes(id)) },
  { id: 'auto_5', name: 'Автоматизация', desc: 'Купи 5 уровень автокликера', icon: '⚙️', reward: 5000, check: () => autoLevel >= 5 },
  { id: 'auto_10', name: 'Машина для монет', desc: 'Купи 10 уровень автокликера', icon: '🤖', reward: 15000, check: () => autoLevel >= 10 },
  { id: 'auto_25', name: 'Кибер-фермер', desc: 'Купи 25 уровень автокликера', icon: '💻', reward: 50000, check: () => autoLevel >= 25 },
  { id: 'invest_10k', name: 'Инвестор', desc: 'Вложи 10 000 монет', icon: '📈', reward: 3000, check: () => invested >= 10000 },
  { id: 'invest_100k', name: 'Финансист', desc: 'Вложи 100 000 монет', icon: '🏦', reward: 15000, check: () => invested >= 100000 },
  { id: 'invest_1m', name: 'Банкир', desc: 'Вложи 1 000 000 монет', icon: '💰', reward: 100000, check: () => invested >= 1000000 },
  { id: 'roulette_win_5', name: 'Удачливый', desc: 'Выиграй в рулетку 5 раз', icon: '🍀', reward: 2000, check: () => rouletteWins >= 5 },
  { id: 'roulette_win_20', name: 'Счастливчик', desc: 'Выиграй в рулетку 20 раз', icon: '🎰', reward: 10000, check: () => rouletteWins >= 20 },
  { id: 'offline_10k', name: 'Офлайн-магнат', desc: 'Заработай 10 000 монет офлайн', icon: '💤', reward: 5000, check: () => offlineEarnings >= 10000 },
  { id: 'offline_100k', name: 'Спящий гигант', desc: 'Заработай 100 000 монет офлайн', icon: '🌟', reward: 25000, check: () => offlineEarnings >= 100000 },
];

// ==================== ФОН ====================
function setBack

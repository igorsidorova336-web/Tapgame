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
    .coin {
      display: none;
    }
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
  </style>
</head>
<body>
<div class="app" id="app">
  <h1>💰 КЛИКЕР</h1>
  <div style="font-size:13px;opacity:0.5;margin-bottom:5px;">👥 Онлайн: <span id="onlineCount">0</span> игроков</div>
  <div class="score" id="score">0</div>
  
  <!-- 3D МОНЕТКА -->
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
    <div style="display:flex;justify-content:space-between;align-items:center;">
      <div>
        <div style="font-size:14px;font-weight:bold;">🤖 Автокликер</div>
        <div style="font-size:12px;opacity:0.6;">Уровень: <span id="autoLevel">0</span> (+<span id="autoIncome">0</span>/сек)</div>
      </div>
      <button id="autoBtn" style="width:auto;padding:8px 16px;font-size:13px;background:#4fc3f7;margin-top:0;">
        Купить (<span id="autoPrice">1000</span>🪙)
      </button>
    </div>
  </div>

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

  <!-- МАГАЗИН СКИНОВ -->
  <div style="margin:15px 0;background:#12121f;border-radius:16px;padding:12px;">
    <div style="font-size:14px;font-weight:bold;margin-bottom:10px;">🎨 Скины</div>
    <div id="skinShop" style="display:flex;gap:6px;justify-content:center;flex-wrap:wrap;"></div>
  </div>

  <!-- НАСТРОЙКА ФОНА -->
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
      <button id="uploadBgBtn" style="width:auto;padding:8px 16px;font-size:12px;background:#4fc3f7;margin-top:0;">📁 Из галереи</button>
      <button onclick="resetBackground()" style="width:auto;padding:8px 16px;font-size:12px;background:#ff6b6b;margin-top:0;">🔄 Сбросить</button>
    </div>
    <input type="file" id="bgFileInput" accept="image/*" style="display:none;">
  </div>

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

  // ==================== СКИНЫ (с цветами для 3D) ====================
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

  // ==================== ФОН ====================
  function setBackground(color) {
    document.getElementById('app').style.background = color;
    localStorage.setItem('gameBackground', color);
  }

  function setBackgroundImage(imageData) {
    document.getElementById('app').style.backgroundImage = 'url(' + imageData + ')';
    document.getElementById('app').style.backgroundSize = 'cover';
    document.getElementById('app').style.backgroundPosition = 'center';
    localStorage.setItem('gameBackgroundImage', imageData);
  }

  function resetBackground() {
    document.getElementById('app').style.background = '#1a1a2e';
    document.getElementById('app').style.backgroundImage = 'none';
    localStorage.removeItem('gameBackground');
    localStorage.removeItem('gameBackgroundImage');
  }

  function loadBackground() {
    var savedImage = localStorage.getItem('gameBackgroundImage');
    if (savedImage) {
      document.getElementById('app').style.backgroundImage = 'url(' + savedImage + ')';
      document.getElementById('app').style.backgroundSize = 'cover';
      document.getElementById('app').style.backgroundPosition = 'center';
    } else if (localStorage.getItem('gameBackground')) {
      document.getElementById('app').style.background = localStorage.getItem('gameBackground');
    }
  }

  document.getElementById('uploadBgBtn').addEventListener('click', function() {
    document.getElementById('bgFileInput').click();
  });

  document.getElementById('bgFileInput').addEventListener('change', function(e) {
    var file = e.target.files[0];
    if (file) {
      var reader = new FileReader();
      reader.onload = function(event) {
        setBackgroundImage(event.target.result);
      };
      reader.readAsDataURL(file);
    }
    this.value = '';
  });

  // ==================== 3D МОНЕТКА ====================
  function init3DCoin() {
    const container = document.getElementById('coin3d');
    if (!container) return;
    
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(45, 1, 0.1, 100);
    camera.position.z = 3.5;
    
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(150, 150);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.shadowMap.enabled = true;
    container.appendChild(renderer.domElement);
    
    const light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(2, 3, 4);
    scene.add(light);
    
    const ambientLight = new THREE.AmbientLight(0x404060);
    scene.add(ambientLight);
    
    const backLight = new THREE.DirectionalLight(0xffdd88, 0.5);
    backLight.position.set(-2, -1, -3);
    scene.add(backLight);
    
    const geometry = new THREE.CylinderGeometry(1, 1, 0.3, 64);
    const material = new THREE.MeshStandardMaterial({
      color: 0xf5c842,
      metalness: 0.7,
      roughness: 0.3,
      emissive: 0x553300,
      emissiveIntensity: 0.1,
    });
    
    const coinMesh = new THREE.Mesh(geometry, material);
    coinMesh.rotation.x = Math.PI / 6;
    coinMesh.castShadow = true;
    scene.add(coinMesh);
    
    const textGeo = new THREE.TorusGeometry(0.6, 0.05, 16, 32);
    const textMat = new THREE.MeshStandardMaterial({
      color: 0xffaa00,
      metalness: 0.9,
      roughness: 0.2,
    });
    const ring = new THREE.Mesh(textGeo, textMat);
    ring.rotation.x = Math.PI / 2;
    ring.position.z = 0.16;
    coinMesh.add(ring);
    
    const starGeo = new THREE.OctahedronGeometry(0.15);
    const starMat = new THREE.MeshStandardMaterial({
      color: 0xffdd44,
      metalness: 0.8,
      roughness: 0.2,
      emissive: 0xff8800,
      emissiveIntensity: 0.2,
    });
    const star = new THREE.Mesh(starGeo, starMat);
    star.position.z = 0.17;
    star.scale.set(1, 1, 0.3);
    coinMesh.add(star);
    
    // Сохраняем для смены цвета
    window.coinMaterial = material;
    window.coinMesh = coinMesh;
    
    let targetScale = 1;
    
    function animate() {
      requestAnimationFrame(animate);
      coinMesh.rotation.y += 0.01;
      coinMesh.rotation.x = Math.PI / 6 + Math.sin(Date.now() * 0.001) * 0.02;
      
      const currentScale = coinMesh.scale.x;
      coinMesh.scale.set(
        currentScale + (targetScale - currentScale) * 0.1,
        currentScale + (targetScale - currentScale) * 0.1,
        currentScale + (targetScale - currentScale) * 0.1
      );
      
      renderer.render(scene, camera);
    }
    animate();
    
    container.addEventListener('click', function(e) {
      targetScale = 0.7;
      setTimeout(() => { targetScale = 1; }, 100);
      if (typeof handleTap === 'function') {
        handleTap(e);
      }
    });
    
    window.addEventListener('resize', function() {
      const rect = container.getBoundingClientRect();
      const size = Math.min(rect.width, rect.height, 150);
      renderer.setSize(size, size);
    });
  }

  // ==================== ОНЛАЙН ====================
  function updateOnline() {
    document.getElementById('onlineCount').textContent = Math.floor(Math.random() * 12) + 3;
  }
  updateOnline();
  setInterval(updateOnline, 15000);

  // ==================== СКИНЫ ====================
  function updateCoinSkin() {
    const skin = SKINS.find(s => s.id === activeSkin);
    if (skin) {
      coin.textContent = skin.emoji;
      if (window.coinMaterial && skin.color !== undefined) {
        window.coinMaterial.color.setHex(skin.color);
        window.coinMaterial.emissive.setHex(skin.color);
        window.coinMaterial.emissiveIntensity = 0.15;
      }
    }
  }

  function renderSkinShop() {
    const shop = document.getElementById('skinShop');
    if (!shop) return;
    shop.innerHTML = '';
    SKINS.forEach(skin => {
      const isOwned = ownedSkins.includes(skin.id);
      const isActive = activeSkin === skin.id;
      const isLegendary = skin.price >= 1000000;
      const isVip = skin.price >= 50000 && skin.price < 1000000;
      const div = document.createElement('div');
      div.className = 'skin-item ' + (isOwned ? 'owned' : 'locked') + (isActive ? ' active' : '') + (isVip ? ' vip' : '') + (isLegendary ? ' legendary' : '');
      let badge = '';
      if (isLegendary) badge = '<div class="skin-badge">🔥 LEGEND</div>';
      else if (isVip) badge = '<div class="skin-badge" style="background:#ff6b6b;">⭐ VIP</div>';
      const priceDisplay = isOwned ? (isActive ? '✅' : '📌') : (skin.price >= 1000000 ? '💎 ' + formatNumber(skin.price) : skin.price + '🪙');
      div.innerHTML = '<div style="font-size:28px;">' + skin.emoji + '</div><div class="skin-price">' + priceDisplay + '</div>' + badge;
      div.addEventListener('click', function(e) {
        e.stopPropagation();
        if (isOwned) {
          activeSkin = skin.id;
          localStorage.setItem('activeSkin', activeSkin);
          updateCoinSkin();
          renderSkinShop();
          saveGame();
        } else {
          if (score < skin.price) {
            alert('❌ Не хватает! Нужно ' + formatNumber(skin.price) + ' монет.');
            return;
          }
          if (confirm('Купить "' + skin.name + '" за ' + formatNumber(skin.price) + ' монет?')) {
            score -= skin.price;
            ownedSkins.push(skin.id);
            localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
            activeSkin = skin.id;
            localStorage.setItem('activeSkin', activeSkin);
            updateCoinSkin();
            updateUI();
            renderSkinShop();
            saveGame();
            if (isLegendary) alert('🎉✨ ПОЗДРАВЛЯЮ! Легендарный скин "' + skin.name + '" твой!');
          }
        }
      });
      shop.appendChild(div);
    });
  }

  // ==================== ТАБЛИЦА ЛИДЕРОВ ====================
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

  // ==================== ЕЖЕДНЕВНЫЙ БОНУС ====================
  function getBonusAmount() {
    return 50 + bonusStreak * 30;
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

  if (bonusBtn) {
    bonusBtn.addEventListener('click', claimBonus);
  }

  setInterval(updateBonusTimer, 1000);

  // ==================== АВТОКЛИКЕР ====================
  function getAutoPrice() {
    return Math.floor(autoBasePrice * Math.pow(1.5, autoLevel));
  }

  function getAutoIncome() {
    return autoLevel * 0.5;
  }

  function updateAutoUI() {
    document.getElementById('autoLevel').textContent = autoLevel;
    document.getElementById('autoIncome').textContent = getAutoIncome().toFixed(1);
    document.getElementById('autoPrice').textContent = getAutoPrice();
  }

  function buyAutoClicker() {
    var price = getAutoPrice();
    if (score < price) {
      alert('❌ Нужно ' + formatNumber(price) + ' монет!');
      return;
    }
    score -= price;
    autoLevel += 1;
    localStorage.setItem('autoLevel', autoLevel);
    updateUI();
    updateAutoUI();
    saveGame();
  }

  setInterval(function() {
    if (autoLevel > 0) {
      var income = getAutoIncome();
      score += income;
      updateUI();
    }
  }, 1000);

  document.getElementById('autoBtn').addEventListener('click', buyAutoClicker);

  // ==================== СОХРАНЕНИЕ ====================
  function saveGame() {
    var data = {
      score: score,
      energy: energy,
      tapPower: tapPower,
      level: level,
      leaderboard: leaderboard,
      ownedSkins: ownedSkins,
      activeSkin: activeSkin,
      lastBonusDate: lastBonusDate,
      bonusStreak: bonusStreak,
      autoLevel: autoLevel
    };
    localStorage.setItem('tapGameData', JSON.stringify(data));
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
        if (data.ownedSkins) ownedSkins = data.ownedSkins;
        if (data.activeSkin) activeSkin = data.activeSkin;
        if (data.lastBonusDate) lastBonusDate = data.lastBonusDate;
        if (data.bonusStreak) bonusStreak = data.bonusStreak;
        if (data.autoLevel) autoLevel = data.autoLevel;
      }
    } catch(e) {}
    updateUI();
    updateCoinSkin();
    renderSkinShop();
    renderLeaderboard();
    updateBonusUI();
    updateAutoUI();
    loadBackground();
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
  upgradeBtn.addEventListener('click', buyUpgrade);

  var nameInput = document.getElementById('playerNameInput');
  if (nameInput) {
    nameInput.value = playerName;
    nameInput.addEventListener('input', function() {
      playerName = this.value || 'Игрок';
      localStorage.setItem('playerName', playerName);
      renderLeaderboard();
    });
  }

  document.getElementById('saveScoreBtn').addEventListener('click', function() {
    var currentScore = Math.floor(score);
    var lastSaveDate = localStorage.getItem('lastLeaderboardSave');
    var today = new Date().toDateString();
    
    if (lastSaveDate === today) {
      alert('⏳ Ты уже сохранял результат сегодня! Возвращайся завтра.');
      return;
    }
    
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
    
    localStorage.setItem('lastLeaderboardSave', today);
    
    saveLeaderboard();
    saveGame();
    renderLeaderboard();
    alert('✅ Результат сохранён в таблицу лидеров! Возвращайся завтра за новым рекордом!');
  });

  // ==================== АВТОСОХРАНЕНИЕ ====================
  setInterval(saveGame, 5000);
  window.addEventListener('beforeunload', saveGame);

  // ==================== СТАРТ ====================
  setTimeout(init3DCoin, 100);
  loadGame();
</script>
</body>
</html>

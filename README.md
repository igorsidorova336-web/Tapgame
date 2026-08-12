 </div>

<script>
  // ---------- ИГРОВАЯ ЛОГИКА ----------
  let score = 0;
  let energy = 100;
  let tapPower = 1;
  let level = 1;
  const MAX_ENERGY = 100;
 // ---------- СКИНЫ ----------
const SKINS = [
  { id: 'gold', emoji: '🪙', name: 'Золотая', price: 50 },
  { id: 'diamond', emoji: '💎', name: 'Алмазная', price: 150 },
  { id: 'plasma', emoji: '🌀', name: 'Плазменная', price: 500 },
  { id: 'rainbow', emoji: '🌈', name: 'Радужная', price: 1500 },
  { id: 'neon', emoji: '💜', name: 'Неоновая', price: 50000 },
  { id: 'crystal', emoji: '❄️', name: 'Хрустальная', price: 100000 },
  { id: 'legend_gold', emoji: '👑', name: 'Королевская', price: 1000000 },
  { id: 'legend_dark', emoji: '🌑', name: 'Тёмная звезда', price: 2500000 },
  { id: 'legend_cosmic', emoji: '🌌', name: 'Космическая', price: 5000000 },
  { id: 'legend_god', emoji: '⚡', name: 'Божественная', price: 10000000 },
];

let ownedSkins = JSON.parse(localStorage.getItem('ownedSkins')) || ['gold'];
let activeSkin = localStorage.getItem('activeSkin') || 'gold';

function formatNumber(num) {
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
}

function updateCoinSkin() {
  const skin = SKINS.find(s => s.id === activeSkin);
  if (skin) coin.textContent = skin.emoji;
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
    div.className = `skin-item ${isOwned ? 'owned' : 'locked'} ${isActive ? 'active' : ''} ${isVip ? 'vip' : ''} ${isLegendary ? 'legendary' : ''}`;
    
    let badge = '';
    if (isLegendary) badge = '<div class="skin-badge">🔥 LEGEND</div>';
    else if (isVip) badge = '<div class="skin-badge" style="background:#ff6b6b;">⭐ VIP</div>';
    
    const priceDisplay = isOwned ? (isActive ? '✅' : '📌') : (skin.price >= 1000000 ? '💎 ' + formatNumber(skin.price) : skin.price + '🪙');
    
    div.innerHTML = `
      <div style="font-size: 32px;">${skin.emoji}</div>
      <div class="skin-price">${priceDisplay}</div>
      ${badge}
    `;
    
    div.addEventListener('click', () => {
      if (isOwned) {
        activeSkin = skin.id;
        localStorage.setItem('activeSkin', activeSkin);
        updateCoinSkin();
        renderSkinShop();
      } else {
        if (score < skin.price) {
          alert(`❌ Не хватает! Нужно ${formatNumber(skin.price)} монет.`);
          return;
        }
        if (confirm(`Купить "${skin.name}" за ${formatNumber(skin.price)} монет?`)) {
          score -= skin.price;
          ownedSkins.push(skin.id);
          localStorage.setItem('ownedSkins', JSON.stringify(ownedSkins));
          activeSkin = skin.id;
          localStorage.setItem('activeSkin', activeSkin);
          updateCoinSkin();
          updateUI();
          renderSkinShop();
          if (isLegendary) {
            alert(`🎉✨ ПОЗДРАВЛЯЮ! Легендарный скин "${skin.name}" твой!`);
          }
        }
      }
    });
    
    shop.appendChild(div);
  });
}

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
  const data = {
   score, energy, tapPower, level, ownedSkins: ownedSkins,
   activeSkin: activeSkin,
   leaderboard: leaderboard 
};
  localStorage.setItem('tapGameData',JSON.stringify(data));
}
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
 updateCoinSkin();
renderSkinShop();
 if (data.ownedSkins) ownedSkins = data.ownedSkins;
if (data.activeSkin) activeSkin = data.activeSkin;

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
 // Автосохранение рекорда (если текущий счёт больше минимального в топ-10)
const minScore = leaderboard.length >= 10 ? leaderboard[leaderboard.length - 1].score : 0;
if (Math.floor(score) > minScore && !leaderboard.some(e => e.name === playerName && e.score === Math.floor(score))) {
  // Сохраняем новый рекорд
}
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
 // ---------- ТАБЛИЦА ЛИДЕРОВ ----------
let playerName = localStorage.getItem('playerName') || 'Игрок';
let leaderboard = JSON.parse(localStorage.getItem('leaderboard')) || [];

// Загружаем имя игрока
document.getElementById('playerNameInput').value = playerName;

// Сохраняем имя при вводе
document.getElementById('playerNameInput').addEventListener('input', function() {
  playerName = this.value || 'Игрок';
  localStorage.setItem('playerName', playerName);
});

// Сохраняем результат в таблицу
document.getElementById('saveScoreBtn').addEventListener('click', function() {
  const currentScore = Math.floor(score);
  
  // Добавляем запись
  leaderboard.push({
    name: playerName,
    score: currentScore,
    date: new Date().toLocaleDateString()
  });
  
  // Сортируем по убыванию
  leaderboard.sort((a, b) => b.score - a.score);
  
  // Оставляем только топ-10
  if (leaderboard.length > 10) {
    leaderboard = leaderboard.slice(0, 10);
  }
  
  // Сохраняем
  localStorage.setItem('leaderboard', JSON.stringify(leaderboard));
  
  // Обновляем отображение
  renderLeaderboard();
  
  alert('✅ Твой результат сохранён в таблицу лидеров!');
});

// Отрисовка таблицы
function renderLeaderboard() {
  const list = document.getElementById('leaderboardList');
  
  if (leaderboard.length === 0) {
    list.innerHTML = '<div style="opacity: 0.4; text-align: center; padding: 10px;">Нет записей. Стань первым! 🚀</div>';
    return;
  }
  
  let html = '';
  leaderboard.forEach((entry, index) => {
    const medal = index === 0 ? '🥇' : index === 1 ? '🥈' : index === 2 ? '🥉' : `${index + 1}.`;
    const isYou = entry.name === playerName ? ' ⭐' : '';
    html += `
      <div style="display: flex; justify-content: space-between; padding: 6px 0; border-bottom: 1px solid #1a1a2e;">
        <span>${medal} ${entry.name}${isYou}</span>
        <span style="color: #f5c842; font-weight: bold;">${formatNumber(entry.score)} 🪙</span>
      </div>
    `;
  });
  
  list.innerHTML = html;
}

// Загружаем таблицу при старте
renderLeaderboard(); 
</script>
</body>
</html>

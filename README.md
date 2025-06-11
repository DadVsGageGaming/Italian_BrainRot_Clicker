<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Italian Brain Rot Clicker</title>
<style>
  body {
    margin: 0; padding: 0;
    background: #0a0a0a;
    color: #eee;
    font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    user-select: none;
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100vh;
  }
  h1 {
    margin: 20px 0 10px;
    color: #ff6f61;
    font-weight: 700;
  }
  #brainjuice {
    font-size: 1.5rem;
    margin-bottom: 15px;
  }
  #char-container {
    display: flex;
    align-items: center;
    gap: 20px;
    margin-bottom: 30px;
  }
  button.arrow-btn {
    font-size: 3rem;
    background: transparent;
    border: none;
    color: #ff6f61;
    cursor: pointer;
    padding: 8px 15px;
    transition: color 0.3s ease;
    user-select: none;
  }
  button.arrow-btn:disabled {
    color: #55111188;
    cursor: default;
  }
  #char-name {
    font-size: 1.6rem;
    font-weight: 700;
    color: #ff6f61;
    min-width: 230px;
    text-align: center;
  }
  #unlock-text {
    font-size: 1rem;
    font-weight: 600;
    color: #ff5555;
    min-height: 24px;
    text-align: center;
    margin-top: 4px;
  }
  #game-area {
    display: flex;
    align-items: center;
    gap: 30px;
  }
  #click-btn {
    font-size: 3rem;
    padding: 30px 60px;
    border: none;
    border-radius: 15px;
    background-color: #ff6f61;
    color: white;
    cursor: pointer;
    box-shadow: 0 0 12px #ff6f61cc;
    user-select: none;
    transition: transform 0.1s ease;
  }
  #click-btn:active {
    transform: scale(0.95);
  }
  #upgrade-panel {
    background: #222;
    border-radius: 18px;
    padding: 20px;
    width: 320px;
    box-sizing: border-box;
    user-select: none;
  }
  #upgrade-panel h3 {
    margin: 0 0 15px 0;
    color: #ff6f61;
    font-weight: 700;
    font-size: 1.3rem;
    text-align: center;
  }
  .upgrade-item {
    background: #333;
    border-radius: 12px;
    padding: 12px 15px;
    margin-bottom: 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    transition: background-color 0.2s ease;
  }
  .upgrade-item:hover:not(.disabled) {
    background-color: #ff6f61;
    color: white;
  }
  .upgrade-item.disabled {
    opacity: 0.4;
    pointer-events: none;
  }
  .upgrade-info {
    flex-grow: 1;
    text-align: left;
    user-select: none;
  }
  .upgrade-name {
    font-weight: 600;
    margin-bottom: 3px;
  }
  .upgrade-desc {
    font-size: 0.9rem;
    opacity: 0.85;
  }
  .upgrade-cost {
    font-weight: 700;
    min-width: 80px;
    text-align: right;
    user-select: none;
  }
  .upgrade-owned {
    font-size: 0.8rem;
    color: #aaa;
    margin-top: 4px;
    user-select: none;
  }
</style>
</head>
<body>

<h1>Italian Brain Rot Clicker</h1>
<div id="brainjuice">Brain Juice: 0</div>

<div id="char-container">
  <button id="left-arrow" class="arrow-btn" aria-label="Previous character">&larr;</button>
  <div>
    <div id="char-name">Loading...</div>
    <div id="unlock-text"></div>
  </div>
  <button id="right-arrow" class="arrow-btn" aria-label="Next character">&rarr;</button>
</div>

<div id="game-area">
  <button id="click-btn" aria-label="Gain Brain Juice">Click Me!</button>
  <div id="upgrade-panel">
    <h3>Upgrades</h3>
    <div id="upgrade-list"></div>
  </div>
</div>

<audio id="click-sound" src="https://freesound.org/data/previews/514/514202_10211442-lq.mp3" preload="auto"></audio>

<script>
  let brainJuice = 0;
  let clickPower = 1;
  let autoClickPower = 0;
  let currentCharIndex = 0;

  const brainjuiceEl = document.getElementById("brainjuice");
  const charNameEl = document.getElementById("char-name");
  const unlockTextEl = document.getElementById("unlock-text");
  const leftArrow = document.getElementById("left-arrow");
  const rightArrow = document.getElementById("right-arrow");
  const clickBtn = document.getElementById("click-btn");
  const upgradeList = document.getElementById("upgrade-list");
  const clickSound = document.getElementById("click-sound");

  const characters = [
    {
      name: "Cappuccino Assassino",
      unlockAt: 0,
      upgrades: [
        { name: "Strong Espresso", desc: "+1 Brain Juice/click", baseCost: 10, cost: 10, power: 1, type: "click", bought: 0 },
        { name: "Creamy Milk", desc: "+5 Brain Juice/click", baseCost: 100, cost: 100, power: 5, type: "click", bought: 0 },
        { name: "Sweet Sugar", desc: "+15 Brain Juice/click", baseCost: 500, cost: 500, power: 15, type: "click", bought: 0 },
        { name: "Double Shot", desc: "+50 Brain Juice/click", baseCost: 2000, cost: 2000, power: 50, type: "click", bought: 0 },
        { name: "Espresso Machine", desc: "+150 Brain Juice/click", baseCost: 10000, cost: 10000, power: 150, type: "click", bought: 0 },
        { name: "Auto Brewer", desc: "+100 Brain Juice/sec", baseCost: 50000, cost: 50000, power: 100, type: "auto", bought: 0 },
        { name: "Espresso Robot", desc: "+500 Brain Juice/sec", baseCost: 200000, cost: 200000, power: 500, type: "auto", bought: 0 },
        { name: "Barista Team", desc: "+2000 Brain Juice/sec", baseCost: 1000000, cost: 1000000, power: 2000, type: "auto", bought: 0 },
      ],
    },
    {
      name: "Tung Tung Tung Sahur",
      unlockAt: 1_000_000,  // 1M
      upgrades: [
        { name: "Sahur Prep", desc: "+100 Brain Juice/click", baseCost: 50000, cost: 50000, power: 100, type: "click", bought: 0 },
        { name: "Energy Drink", desc: "+500 Brain Juice/click", baseCost: 200000, cost: 200000, power: 500, type: "click", bought: 0 },
        { name: "Night Chef", desc: "+1500 Brain Juice/click", baseCost: 1000000, cost: 1000000, power: 1500, type: "click", bought: 0 },
        { name: "Sahur Delivery", desc: "+10,000 Brain Juice/sec", baseCost: 5_000_000, cost: 5_000_000, power: 10000, type: "auto", bought: 0 },
        { name: "Sahur Robot", desc: "+50,000 Brain Juice/sec", baseCost: 20_000_000, cost: 20_000_000, power: 50000, type: "auto", bought: 0 },
        { name: "Sahur Fleet", desc: "+200,000 Brain Juice/sec", baseCost: 100_000_000, cost: 100_000_000, power: 200000, type: "auto", bought: 0 },
      ],
    },
    {
      name: "Simian Scream",
      unlockAt: 1_000_000_000, // 1B
      upgrades: [
        { name: "Monkey Grip", desc: "+1,000 Brain Juice/click", baseCost: 500_000_000, cost: 500_000_000, power: 1000, type: "click", bought: 0 },
        { name: "Jungle Juice", desc: "+5,000 Brain Juice/click", baseCost: 2_000_000_000, cost: 2_000_000_000, power: 5000, type: "click", bought: 0 },
        { name: "Banana Split", desc: "+20,000 Brain Juice/click", baseCost: 10_000_000_000, cost: 10_000_000_000, power: 20000, type: "click", bought: 0 },
        { name: "Simian Squad", desc: "+100,000 Brain Juice/sec", baseCost: 50_000_000_000, cost: 50_000_000_000, power: 100000, type: "auto", bought: 0 },
        { name: "Monkey Bots", desc: "+500,000 Brain Juice/sec", baseCost: 200_000_000_000, cost: 200_000_000_000, power: 500000, type: "auto", bought: 0 },
        { name: "Banana Empire", desc: "+2,000,000 Brain Juice/sec", baseCost: 1_000_000_000_000, cost: 1_000_000_000_000, power: 2000000, type: "auto", bought: 0 },
      ],
    },
    {
      name: "The Joker Focker",
      unlockAt: 1_000_000_000_000, // 1Q
      upgrades: [
        { name: "Joker's Card", desc: "+10,000 Brain Juice/click", baseCost: 5_000_000_000_000, cost: 5_000_000_000_000, power: 10000, type: "click", bought: 0 },
        { name: "Focker's Deck", desc: "+50,000 Brain Juice/click", baseCost: 20_000_000_000_000, cost: 20_000_000_000_000, power: 50000, type: "click", bought: 0 },
        { name: "Laugh Riot", desc: "+200,000 Brain Juice/click", baseCost: 100_000_000_000_000, cost: 100_000_000_000_000, power: 200000, type: "click", bought: 0 },
        { name: "Joker's Army", desc: "+1,000,000 Brain Juice/sec", baseCost: 500_000_000_000_000, cost: 500_000_000_000_000, power: 1000000, type: "auto", bought: 0 },
        { name: "Focker's Bots", desc: "+5,000,000 Brain Juice/sec", baseCost: 2_000_000_000_000_000, cost: 2_000_000_000_000_000, power: 5000000, type: "auto", bought: 0 },
        { name: "Ultimate Focker", desc: "+20,000,000 Brain Juice/sec", baseCost: 10_000_000_000_000_000, cost: 10_000_000_000_000_000, power: 20000000, type: "auto", bought: 0 },
      ],
    },
  ];

  function formatNumber(num) {
    if (num < 1000) return num.toString();
    const units = ["K", "M", "B", "T", "Q"];
    let unitIndex = -1;
    let n = num;
    while (n >= 1000 && unitIndex < units.length - 1) {
      n /= 1000;
      unitIndex++;
    }
    return n.toFixed(2) + units[unitIndex];
  }

  function updateBrainJuiceDisplay() {
    brainjuiceEl.textContent = `Brain Juice: ${formatNumber(brainJuice)}`;
  }

  function updateCharacterDisplay() {
    const char = characters[currentCharIndex];
    if (brainJuice >= char.unlockAt) {
      charNameEl.textContent = char.name;
      unlockTextEl.textContent = "";
      leftArrow.disabled = currentCharIndex === 0;
      rightArrow.disabled = currentCharIndex === characters.length - 1;
      clickBtn.disabled = false;
      upgradeList.innerHTML = "";
      char.upgrades.forEach((upgrade, i) => {
        const canAfford = brainJuice >= upgrade.cost;
        const upgradeDiv = document.createElement("div");
        upgradeDiv.className = "upgrade-item" + (canAfford ? "" : " disabled");
        upgradeDiv.tabIndex = 0;
        upgradeDiv.setAttribute("role", "button");
        upgradeDiv.setAttribute("aria-pressed", "false");
        upgradeDiv.setAttribute("aria-label", `${upgrade.name}, ${upgrade.desc}, costs ${formatNumber(upgrade.cost)} Brain Juice. Owned: ${upgrade.bought}`);
        upgradeDiv.innerHTML = `
          <div class="upgrade-info">
            <div class="upgrade-name">${upgrade.name}</div>
            <div class="upgrade-desc">${upgrade.desc}</div>
            <div class="upgrade-owned">Owned: ${upgrade.bought}</div>
          </div>
          <div class="upgrade-cost">${formatNumber(upgrade.cost)}</div>
        `;
        upgradeDiv.onclick = () => buyUpgrade(i);
        upgradeDiv.onkeydown = (e) => { if (e.key === "Enter" || e.key === " ") { buyUpgrade(i); e.preventDefault(); } };
        upgradeList.appendChild(upgradeDiv);
      });
      updateUpgradesState();
    } else {
      charNameEl.textContent = "???";
      unlockTextEl.textContent = `Unlocks at ${formatNumber(char.unlockAt)} Brain Juice`;
      leftArrow.disabled = currentCharIndex === 0;
      rightArrow.disabled = currentCharIndex === characters.length - 1;
      clickBtn.disabled = true;
      upgradeList.innerHTML = "";
    }
  }

  function buyUpgrade(index) {
    const char = characters[currentCharIndex];
    const upgrade = char.upgrades[index];
    if (brainJuice >= upgrade.cost) {
      brainJuice -= upgrade.cost;
      upgrade.bought++;
      // Increase clickPower or autoClickPower accordingly
      if (upgrade.type === "click") {
        clickPower += upgrade.power;
      } else if (upgrade.type === "auto") {
        autoClickPower += upgrade.power;
      }
      // Increase cost for next purchase by 20%
      upgrade.cost = Math.floor(upgrade.baseCost * Math.pow(1.2, upgrade.bought));
      updateBrainJuiceDisplay();
      updateCharacterDisplay();
    }
  }

  function updateUpgradesState() {
    const char = characters[currentCharIndex];
    const upgradeDivs = upgradeList.children;
    for (let i = 0; i < upgradeDivs.length; i++) {
      const upgrade = char.upgrades[i];
      if (brainJuice >= upgrade.cost) {
        upgradeDivs[i].classList.remove("disabled");
      } else {
        upgradeDivs[i].classList.add("disabled");
      }
      upgradeDivs[i].querySelector(".upgrade-owned").textContent = `Owned: ${upgrade.bought}`;
      upgradeDivs[i].querySelector(".upgrade-cost").textContent = formatNumber(upgrade.cost);
    }
  }

  function switchCharacter(direction) {
    if (direction === "left" && currentCharIndex > 0) {
      currentCharIndex--;
    } else if (direction === "right" && currentCharIndex < characters.length - 1) {
      currentCharIndex++;
    }
    updateCharacterDisplay();
  }

  function gainBrainJuice(amount) {
    brainJuice += amount;
    updateBrainJuiceDisplay();
    updateUpgradesState();
  }

  leftArrow.onclick = () => switchCharacter("left");
  rightArrow.onclick = () => switchCharacter("right");

  clickBtn.onclick = () => {
    gainBrainJuice(clickPower);
    clickSound.currentTime = 0;
    clickSound.play().catch(() => {});
  };

  // Auto click increment per second
  setInterval(() => {
    if (autoClickPower > 0) {
      gainBrainJuice(autoClickPower);
    }
  }, 1000);

  // Initialize display
  updateBrainJuiceDisplay();
  updateCharacterDisplay();
</script>

</body>
</html>


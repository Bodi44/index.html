<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Betting Game + Real Ad</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(to bottom, #e0f7fa, #fff);
      text-align: center;
      padding: 30px;
      color: #006064;
    }

    h1 { font-size: 2.5em; }
    .balance { font-size: 1.2em; margin-bottom: 20px; }

    input[type=number] {
      padding: 10px;
      width: 100px;
      font-size: 1em;
    }

    .team-button {
      padding: 10px 20px;
      margin: 5px;
      border: none;
      border-radius: 8px;
      font-size: 1em;
      cursor: pointer;
      background-color: #00796b;
      color: white;
    }

    .team-button:hover {
      background-color: #004d40;
    }

    .result, .history {
      margin-top: 20px;
      font-weight: bold;
    }

    .disabled {
      opacity: 0.5;
      pointer-events: none;
    }

    #ad-button {
      margin-top: 20px;
      background-color: #f57c00;
    }

    /* Рекламное окно */
    #ad-modal {
      display: none;
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.8);
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }

    #ad-box {
      background: white;
      padding: 20px;
      width: 320px;
      border-radius: 12px;
      text-align: center;
      position: relative;
    }

    #ad-box iframe {
      width: 100%;
      height: 180px;
      border: none;
      border-radius: 8px;
    }

    #ad-close {
      margin-top: 10px;
      padding: 8px 20px;
      background-color: #388e3c;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      display: none;
    }

    #ad-timer {
      font-weight: bold;
      margin-top: 10px;
    }
  </style>
</head>
<body>

  <h1>📈 Betting Game Deluxe</h1>

  <div class="balance">Balance: <span id="balance">0</span> 💵</div>

  <label>Bet Amount:</label><br>
  <input type="number" id="bet-amount" min="1" value="100"><br><br>

  <div>
    <button class="team-button" onclick="placeBet('A', 1.5)">Bet on Team A (x1.5)</button>
    <button class="team-button" onclick="placeBet('B', 2)">Bet on Team B (x2)</button>
    <button class="team-button" onclick="placeBet('C', 3)">Bet on Team C (x3)</button>
  </div>

  <div class="result" id="result"></div>
  <div class="history" id="history"></div>

  <button class="team-button" id="ad-button" onclick="watchAd()">📺 Watch Ad to Get +200 💵</button>
  <div id="ad-status"></div>

  <!-- Рекламное модальное окно -->
  <div id="ad-modal">
    <div id="ad-box">
      <h3>Sponsored Ad</h3>
      <!-- YouTube Shorts (вставлено твоё видео) -->
      <a href="https://youtube.com/shorts/OkIKhCYcHSg?si=v4GjjQSj8exgUD02" target="_blank">
        <iframe src="https://www.youtube.com/embed/OkIKhCYcHSg?autoplay=1&mute=1" allow="autoplay"></iframe>
      </a>
      <div id="ad-timer">Ad ends in 5 seconds...</div>
      <button id="ad-close" onclick="closeAd()">Close & Claim Reward</button>
    </div>
  </div>

  <script>
    let balance = parseInt(localStorage.getItem("betBalance")) || 1000;
    let winStreak = 0;
    let adCooldown = false;

    function updateBalanceDisplay() {
      document.getElementById("balance").textContent = balance;
      localStorage.setItem("betBalance", balance);
    }

    function placeBet(team, multiplier) {
      const betAmount = parseInt(document.getElementById("bet-amount").value);
      const resultEl = document.getElementById("result");
      const historyEl = document.getElementById("history");

      if (isNaN(betAmount) || betAmount <= 0 || betAmount > balance) {
        resultEl.textContent = "❌ Invalid bet!";
        return;
      }

      const teams = ['A', 'B', 'C'];
      const winner = teams[Math.floor(Math.random() * teams.length)];
      let message = `🏁 Match result: Team ${winner} wins. `;

      if (team === winner) {
        const winAmount = Math.floor(betAmount * multiplier);
        balance += winAmount;
        winStreak++;
        message += `🎉 You won ${winAmount}! (Streak: ${winStreak})`;

        if (winStreak >= 3) {
          balance += 100;
          message += ` 🏆 Bonus +100 for 3 wins in a row!`;
        }
      } else {
        balance -= betAmount;
        winStreak = 0;
        message += `😢 You lost ${betAmount}.`;
      }

      resultEl.textContent = message;
      historyEl.innerHTML += `Bet on ${team}, Winner: ${winner}<br>`;
      updateBalanceDisplay();

      if (balance <= 0) {
        resultEl.textContent += " 💀 You're broke! Game Over.";
        document.querySelectorAll(".team-button").forEach(btn => btn.classList.add("disabled"));
      }
    }

    function watchAd() {
      if (adCooldown) return;

      document.getElementById("ad-modal").style.display = "flex";
      let seconds = 5;
      document.getElementById("ad-timer").textContent = `Ad ends in ${seconds} seconds...`;
      document.getElementById("ad-close").style.display = "none";

      const timer = setInterval(() => {
        seconds--;
        document.getElementById("ad-timer").textContent = `Ad ends in ${seconds} seconds...`;
        if (seconds <= 0) {
          clearInterval(timer);
          document.getElementById("ad-close").style.display = "inline-block";
        }
      }, 1000);
    }

    function closeAd() {
      document.getElementById("ad-modal").style.display = "none";
      balance += 200;
      updateBalanceDisplay();
      document.getElementById("ad-status").textContent = "✅ You received 200 💵 from ad!";
      adCooldown = true;

      const adBtn = document.getElementById("ad-button");
      adBtn.disabled = true;

      let cooldown = 60;
      const countdown = setInterval(() => {
        cooldown--;
        document.getElementById("ad-status").textContent = `⏳ Next ad in ${cooldown}s`;
        if (cooldown <= 0) {
          clearInterval(countdown);
          adBtn.disabled = false;
          adCooldown = false;
          document.getElementById("ad-status").textContent = "🎁 Ready to watch ad again!";
        }
      }, 1000);
    }

    updateBalanceDisplay();
  </script>

</body>
</html>

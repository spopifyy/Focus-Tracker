<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Focus Tracker</title>
  <style>
    /* Reset margin/padding and full height on html and body */
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      background-color: #f0f4f8;
      font-family: 'Segoe UI', sans-serif;
      display: flex;
      justify-content: center;
      align-items: flex-start; /* align container to top */
      padding-top: 40px;
    }

    /* Main container with reduced padding bottom */
    .container {
      max-width: 600px;
      width: 100%;
      background: #fff;
      padding: 25px 25px 15px 25px; /* less bottom padding */
      border-radius: 16px;
      box-shadow: 0 6px 16px rgba(0,0,0,0.1);
      box-sizing: border-box;
    }

    h1 {
      text-align: center;
      margin: 0 0 20px 0; /* remove bottom margin if large */
    }

    form {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
      gap: 10px;
      margin-bottom: 15px;
    }

    input, select, button {
      padding: 10px;
      font-size: 14px;
      border-radius: 8px;
      border: 1px solid #ccc;
      box-sizing: border-box;
    }

    button {
      background: #4CAF50;
      color: white;
      border: none;
      cursor: pointer;
      grid-column: span 2;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: #45a049;
    }

    /* Reset button fixed in top right corner */
    #resetBtn {
      position: fixed;
      top: 20px;
      right: 20px;
      background-color: #e53935;
      color: white;
      border: none;
      padding: 12px 18px;
      border-radius: 50px;
      font-weight: bold;
      cursor: pointer;
      box-shadow: 0 3px 8px rgba(0,0,0,0.2);
      transition: background-color 0.3s ease, transform 0.2s ease;
      z-index: 1000;
    }

    #resetBtn:hover {
      background-color: #b71c1c;
      transform: scale(1.1);
    }

    .focus-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 10px;
    }

    .focus-cell {
      width: 100%;
      aspect-ratio: 1;
      border-radius: 12px;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
      font-size: 14px;
      color: white;
      cursor: pointer;
      transition: transform 0.2s ease;
      user-select: none;
    }

    .focus-cell:hover {
      transform: scale(1.05);
    }

    .focus-cell.high { background-color: #4caf50; }
    .focus-cell.medium { background-color: #fbc02d; }
    .focus-cell.low { background-color: #f44336; }

    .tooltip {
      position: absolute;
      bottom: 110%;
      left: 50%;
      transform: translateX(-50%);
      background: #333;
      padding: 8px 12px;
      border-radius: 6px;
      font-size: 12px;
      color: white;
      white-space: nowrap;
      display: none;
      z-index: 10;
    }

    .focus-cell:hover .tooltip {
      display: block;
    }

    .summary-card {
      margin-top: 20px;
      margin-bottom: 10px; /* reduce bottom margin */
      background-color: #eef6ff;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.05);
      font-size: 16px;
    }

    .summary-card h3 {
      margin-bottom: 10px;
      color: #333;
    }
  </style>
</head>
<body>
  <button id="resetBtn" title="Reset all data">Reset Data</button>
  
  <div class="container">
    <a href="https://spopifyy.github.io/Focus-Tracker/">Focus-Tracker</a>
    <h1>🧠 Weekly Focus Tracker</h1>

    <form id="focus-form">
      <select id="time" required>
        <option value="" disabled selected>Time of Day</option>
        <option>Morning</option>
        <option>Afternoon</option>
        <option>Evening</option>
      </select>
      <input type="number" id="sleep" min="0" max="12" placeholder="Sleep Hours" required />
      <select id="caffeine" required>
        <option value="" disabled selected>Caffeine Intake</option>
        <option>None</option>
        <option>A little</option>
        <option>A lot</option>
      </select>
      <button type="submit">Add</button>
    </form>

    <div class="focus-grid" id="focusGrid"></div>

    <div id="summary" class="summary-card">
      <h3>📊 Weekly Summary</h3>
      <p>Average Focus Score: <span id="avgScore">-</span></p>
      <p>Average Sleep: <span id="avgSleep">-</span> hours</p>
      <p>Most Caffeine: <span id="commonCaffeine">-</span></p>
    </div>
  </div>

  <script>
    const form = document.getElementById("focus-form");
    const grid = document.getElementById("focusGrid");
    const resetBtn = document.getElementById("resetBtn");

    form.addEventListener("submit", function (e) {
      e.preventDefault();

      const time = document.getElementById("time").value;
      const sleep = parseFloat(document.getElementById("sleep").value);
      const caffeine = document.getElementById("caffeine").value;

      // Focus Score Calculation
      let score = 0;

      // Sleep score out of 40
      if (sleep === 8) score += 40;
      else if (sleep >= 7 && sleep <= 9) score += 35;
      else if (sleep >= 6 && sleep <= 10) score += 25;
      else score += 15;

      // Caffeine score out of 20
      if (caffeine === "None") score += 20;
      else if (caffeine === "A little") score += 10;
      else score -= 10;

      // Time of day out of 20
      if (time === "Morning") score += 20;
      else if (time === "Afternoon") score += 10;
      else score += 0;

      // Clamp score between 0–100
      score = Math.max(0, Math.min(100, score));

      const entry = {
        date: new Date().toLocaleDateString(),
        time,
        sleep,
        caffeine,
        score
      };

      let log = JSON.parse(localStorage.getItem("focusLog")) || [];
      log.push(entry);
      if (log.length > 7) log = log.slice(log.length - 7);
      localStorage.setItem("focusLog", JSON.stringify(log));

      renderGrid();
      form.reset();
    });

    function getScoreLevel(score) {
      if (score >= 75) return "high";
      if (score >= 50) return "medium";
      return "low";
    }

    function renderGrid() {
      const log = JSON.parse(localStorage.getItem("focusLog")) || [];
      grid.innerHTML = "";

      log.forEach(entry => {
        const cell = document.createElement("div");
        cell.className = `focus-cell ${getScoreLevel(entry.score)}`;
        cell.innerHTML = `
          ${new Date(entry.date).toLocaleDateString("en-US", { weekday: "short" })}
          <div class="tooltip">
            Time: ${entry.time}<br>
            Sleep: ${entry.sleep}h<br>
            Caffeine: ${entry.caffeine}<br>
            Score: ${entry.score}
          </div>
        `;
        grid.appendChild(cell);
      });

      calculateAverages(log);
    }

    function calculateAverages(log) {
      if (log.length === 0) {
        document.getElementById("avgScore").textContent = "-";
        document.getElementById("avgSleep").textContent = "-";
        document.getElementById("commonCaffeine").textContent = "-";
        return;
      }

      const totalScore = log.reduce((sum, e) => sum + e.score, 0);
      const totalSleep = log.reduce((sum, e) => sum + parseFloat(e.sleep), 0);

      const caffeineCounts = {};
      log.forEach(e => {
        caffeineCounts[e.caffeine] = (caffeineCounts[e.caffeine] || 0) + 1;
      });

      const mostCommonCaffeine = Object.entries(caffeineCounts)
        .sort((a, b) => b[1] - a[1])[0][0];

      document.getElementById("avgScore").textContent = (totalScore / log.length).toFixed(1);
      document.getElementById("avgSleep").textContent = (totalSleep / log.length).toFixed(1);
      document.getElementById("commonCaffeine").textContent = mostCommonCaffeine;
    }

    resetBtn.addEventListener("click", () => {
      if (confirm("Are you sure you want to reset all focus data?")) {
        localStorage.removeItem("focusLog");
        renderGrid();
      }
    });

    // Initial render on page load
    renderGrid();
  </script>
  
</body>
</html>

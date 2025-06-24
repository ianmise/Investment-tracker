# Investment-tracker<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>My Investment Tracker</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: Arial; background: #f0f0f0; max-width: 800px; margin: auto; padding: 20px; }
    input, button { padding: 10px; margin: 5px 0; width: 100%; }
    .entry { background: #fff; padding: 10px; margin-bottom: 10px; border-left: 5px solid #4CAF50; }
    .summary { background: #e0ffe0; padding: 10px; font-weight: bold; margin-top: 20px; }
    #loginBox { max-width: 400px; margin: auto; margin-top: 100px; }
  </style>
</head>
<body>

<div id="loginBox">
  <h2>🔐 Login</h2>
  <input type="password" id="passwordInput" placeholder="Enter password">
  <button onclick="checkLogin()">Login</button>
  <p id="loginError" style="color:red;"></p>
</div>

<div id="mainApp" style="display:none;">
  <h1>📊 My Investment Tracker</h1>
  <input type="text" id="company" placeholder="Company or Bank Name" />
  <input type="number" id="amount" placeholder="Amount Invested (KES)" />
  <input type="date" id="date" />
  <input type="number" id="current" placeholder="Current Value (KES)" />
  <button onclick="addInvestment()">Add Investment</button>

  <div id="investmentList"></div>
  <div class="summary" id="summary"></div>
  <canvas id="chartCanvas" height="100"></canvas>
</div>

<script>
  const PASSWORD = "mySecret123"; // change to your private password

  function checkLogin() {
    const entered = document.getElementById("passwordInput").value;
    if (entered === PASSWORD) {
      document.getElementById("loginBox").style.display = "none";
      document.getElementById("mainApp").style.display = "block";
      displayInvestments();
    } else {
      document.getElementById("loginError").textContent = "Wrong password!";
    }
  }

  let investments = JSON.parse(localStorage.getItem("investments")) || [];

  function saveInvestments() {
    localStorage.setItem("investments", JSON.stringify(investments));
  }

  function addInvestment() {
    const company = document.getElementById("company").value;
    const amount = parseFloat(document.getElementById("amount").value);
    const date = document.getElementById("date").value;
    const current = parseFloat(document.getElementById("current").value);

    if (!company || !amount || !date || !current) {
      alert("Fill in all fields.");
      return;
    }

    investments.push({ company, amount, date, current });
    saveInvestments();
    displayInvestments();
  }

  function removeInvestment(index) {
    if (confirm("Delete this investment?")) {
      investments.splice(index, 1);
      saveInvestments();
      displayInvestments();
    }
  }

  function displayInvestments() {
    const list = document.getElementById("investmentList");
    const summary = document.getElementById("summary");
    list.innerHTML = "";

    let totalInvested = 0;
    let totalCurrent = 0;

    const companyTotals = {};

    investments.forEach((inv, i) => {
      totalInvested += inv.amount;
      totalCurrent += inv.current;

      if (!companyTotals[inv.company]) companyTotals[inv.company] = 0;
      companyTotals[inv.company] += inv.amount;

      const div = document.createElement("div");
      div.className = "entry";
      div.innerHTML = `
        <strong>${inv.company}</strong><br>
        Invested: KES ${inv.amount}<br>
        Current: KES ${inv.current}<br>
        Date: ${inv.date}<br>
        <button onclick="removeInvestment(${i})">Remove</button>
      `;
      list.appendChild(div);
    });

    const profit = totalCurrent - totalInvested;
    summary.innerHTML = `
      📌 Total Invested: KES ${totalInvested.toFixed(2)}<br>
      📈 Current Value: KES ${totalCurrent.toFixed(2)}<br>
      💰 Profit: KES ${profit.toFixed(2)}
    `;

    updateChart(companyTotals);
  }

  let chart;
  function updateChart(data) {
    const ctx = document.getElementById("chartCanvas").getContext("2d");
    const labels = Object.keys(data);
    const values = Object.values(data);

    if (chart) chart.destroy();

    chart = new Chart(ctx, {
      type: 'pie',
      data: {
        labels: labels,
        datasets: [{
          label: "Investment by Company",
          data: values,
          backgroundColor: [
            '#4caf50', '#2196f3', '#ffc107', '#e91e63', '#9c27b0'
          ],
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        plugins: {
          legend: { position: 'top' },
          title: { display: true, text: 'Your Investment Distribution' }
        }
      }
    });
  }
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>6M EAT FRESH CORNER - Multiuser Dashboard</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  body { font-family: Arial, sans-serif; background: #f2f2f2; margin: 0; padding: 0;}
  .container { max-width: 900px; margin: 40px auto; background: #fff; padding: 20px; border-radius: 8px; box-shadow: 0 0 12px rgba(0,0,0,0.1);}
  h1 { text-align: center; color: #4caf50; }
  label { display: block; margin-top: 10px; }
  input, button, select { width: 100%; padding: 10px; margin-top: 5px; font-size: 16px; box-sizing: border-box; }
  button { background: #4caf50; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; }
  button:hover { background: #45a049; }
  .delete-btn { background: #e74c3c; color: white; border: none; border-radius: 5px; cursor: pointer; padding: 5px 10px; font-weight: bold; }
  .logout-btn { background: #c0392b; margin-top: 10px; }
  .link-btn { background: transparent; border: none; color: #4caf50; cursor: pointer; margin-top: 10px; font-size: 14px; text-decoration: underline; }
  table { width: 100%; border-collapse: collapse; margin-top: 20px; }
  th, td { border: 1px solid #ccc; padding: 10px; text-align: center; }
  th { background: #f9f9f9; }
  .result { margin-top: 20px; background: #e3ffe3; padding: 10px; border-radius: 5px; font-weight: bold; text-align: center; }
  #monthFilter { margin-top: 20px; }
  .hidden { display: none; }
</style>
</head>
<body>

<div class="container" id="loginView">
  <h1>Login</h1>
  <label>Username</label>
  <input type="text" id="loginUsername" placeholder="Enter username" />
  <label>Password</label>
  <input type="password" id="loginPassword" placeholder="Enter password" />
  <button onclick="loginUser()">Login</button>
  <button class="link-btn" onclick="showRegister()">Don't have an account? Register</button>
  <div id="loginMessage" style="color:red; margin-top:10px;"></div>
</div>

<div class="container hidden" id="registerView">
  <h1>Register</h1>
  <label>Username</label>
  <input type="text" id="registerUsername" placeholder="Choose a username" />
  <label>Password</label>
  <input type="password" id="registerPassword" placeholder="Choose a password" />
  <button onclick="registerUser()">Register</button>
  <button class="link-btn" onclick="showLogin()">Already have an account? Login</button>
  <div id="registerMessage" style="color:red; margin-top:10px;"></div>
</div>

<div class="container hidden" id="dashboardView">
  <h1>6M EAT FRESH CORNER</h1>
  <button class="logout-btn" onclick="logoutUser()">Logout</button>

  <label>Date</label>
  <input type="date" id="recordDate" />

  <label>Total Sales (LKR)</label>
  <input type="number" id="sales" placeholder="e.g. 10000" />

  <label>Total Purchase (LKR)</label>
  <input type="number" id="purchase" placeholder="e.g. 4000" />

  <label>Salary (LKR)</label>
  <input type="number" id="salary" placeholder="e.g. 2000" />

  <label>Rent (LKR)</label>
  <input type="number" id="rent" placeholder="e.g. 1000" />

  <label>Electricity (LKR)</label>
  <input type="number" id="electricity" placeholder="e.g. 500" />

  <button onclick="calculateProfit()">Calculate & Save</button>
  <button class="clear-all-btn" onclick="clearAllRecords()">Clear All Records</button>

  <label>Filter by Month</label>
  <select id="monthFilter" onchange="applyMonthFilter()">
    <option value="all">All Records</option>
  </select>

  <div class="result" id="result"></div>

  <canvas id="profitChart"></canvas>

  <table id="historyTable">
    <thead>
      <tr>
        <th>Date</th>
        <th>Sales</th>
        <th>Expenses</th>
        <th>Profit</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
</div>

<script>
  // Show/hide views
  function showLogin() {
    document.getElementById('loginView').classList.remove('hidden');
    document.getElementById('registerView').classList.add('hidden');
    document.getElementById('dashboardView').classList.add('hidden');
    clearMessages();
  }
  function showRegister() {
    document.getElementById('loginView').classList.add('hidden');
    document.getElementById('registerView').classList.remove('hidden');
    document.getElementById('dashboardView').classList.add('hidden');
    clearMessages();
  }
  function showDashboard() {
    document.getElementById('loginView').classList.add('hidden');
    document.getElementById('registerView').classList.add('hidden');
    document.getElementById('dashboardView').classList.remove('hidden');
    clearMessages();
    loadUserData();
  }
  function clearMessages() {
    document.getElementById('loginMessage').textContent = '';
    document.getElementById('registerMessage').textContent = '';
  }

  // Simple localStorage user system
  function getUsers() {
    return JSON.parse(localStorage.getItem('users') || '{}');
  }
  function saveUsers(users) {
    localStorage.setItem('users', JSON.stringify(users));
  }

  // Register new user
  function registerUser() {
    const username = document.getElementById('registerUsername').value.trim();
    const password = document.getElementById('registerPassword').value.trim();
    if (!username || !password) {
      document.getElementById('registerMessage').textContent = 'Please enter username and password.';
      return;
    }
    let users = getUsers();
    if (users[username]) {
      document.getElementById('registerMessage').textContent = 'Username already exists.';
      return;
    }
    users[username] = password; // For security: consider hashing passwords in real apps
    saveUsers(users);
    alert('Registration successful! Please login.');
    showLogin();
  }

  // Login user
  function loginUser() {
    const username = document.getElementById('loginUsername').value.trim();
    const password = document.getElementById('loginPassword').value.trim();
    let users = getUsers();
    if (users[username] && users[username] === password) {
      localStorage.setItem('loggedInUser', username);
      showDashboard();
    } else {
      document.getElementById('loginMessage').textContent = 'Invalid username or password.';
    }
  }

  // Logout
  function logoutUser() {
    localStorage.removeItem('loggedInUser');
    showLogin();
  }

  // Check if logged in on load
  window.onload = function() {
    const loggedInUser = localStorage.getItem('loggedInUser');
    if (loggedInUser) {
      showDashboard();
    } else {
      showLogin();
    }
    document.getElementById('recordDate').value = new Date().toISOString().substring(0,10);
  }

  // Get user-specific keys for localStorage
  function getKey(keyBase) {
    const user = localStorage.getItem('loggedInUser');
    return `${keyBase}_${user}`;
  }

  // Load user data from localStorage
  function loadUserData() {
    profitHistory = JSON.parse(localStorage.getItem(getKey('profitHistory')) || '[]');
    labels = JSON.parse(localStorage.getItem(getKey('labels')) || '[]');
    salesHistory = JSON.parse(localStorage.getItem(getKey('salesHistory')) || '[]');
    expensesHistory = JSON.parse(localStorage.getItem(getKey('expensesHistory')) || '[]');
    populateMonthFilter();
    updateHistoryTable();
    updateChart();
    updateTotalBalanceDisplay();
  }

  // Save user data to localStorage
  function saveUserData() {
    localStorage.setItem(getKey('profitHistory'), JSON.stringify(profitHistory));
    localStorage.setItem(getKey('labels'), JSON.stringify(labels));
    localStorage.setItem(getKey('salesHistory'), JSON.stringify(salesHistory));
    localStorage.setItem(getKey('expensesHistory'), JSON.stringify(expensesHistory));
  }

  // Variables for current user data
  let profitHistory = [];
  let labels = [];
  let salesHistory = [];
  let expensesHistory = [];

  // Calculate profit and save entry
  function calculateProfit() {
    const sales = Number(document.getElementById('sales').value);
    const purchase = Number(document.getElementById('purchase').value);
    const salary = Number(document.getElementById('salary').value);
    const rent = Number(document.getElementById('rent').value);
    const electricity = Number(document.getElementById('electricity').value);
    const date = document.getElementById('recordDate').value;

    if (!date) {
      alert('Please select a valid date.');
      return;
    }
    if (
      isNaN(sales) || isNaN(purchase) || isNaN(salary) || isNaN(rent) || isNaN(electricity) ||
      sales < 0 || purchase < 0 || salary < 0 || rent < 0 || electricity < 0
    ) {
      alert('Please enter valid non-negative numbers for all fields.');
      return;
    }

    const expenses = purchase + salary + rent + electricity;
    const profit = sales - expenses;

    const existingIndex = labels.findIndex(d => d === date);
    if (existingIndex !== -1) {
      salesHistory[existingIndex] = sales;
      expensesHistory[existingIndex] = expenses;
      profitHistory[existingIndex] = profit;
    } else {
      labels.push(date);
      salesHistory.push(sales);
      expensesHistory.push(expenses);
      profitHistory.push(profit);
    }

    saveUserData();
    populateMonthFilter();
    updateHistoryTable();
    updateChart();
    clearInputs();

    updateTotalBalanceDisplay();

    document.getElementById('result').innerHTML += `
      <br>🔹 Expenses: LKR ${expenses}<br>
      ✅ Profit: LKR ${profit}<br>
    `;
  }

  // Clear input fields
  function clearInputs() {
    document.getElementById('sales').value = '';
    document.getElementById('purchase').value = '';
    document.getElementById('salary').value = '';
    document.getElementById('rent').value = '';
    document.getElementById('electricity').value = '';
    document.getElementById('recordDate').value = new Date().toISOString().substring(0,10);
  }

  // Add one row to table by index
  function addRowToTable(index) {
    const tbody = document.getElementById("historyTable").getElementsByTagName("tbody")[0];
    const row = tbody.insertRow();
    row.setAttribute('data-index', index);
    row.innerHTML = `
      <td>${labels[index]}</td>
      <td>${salesHistory[index]}</td>
      <td>${expensesHistory[index]}</td>
      <td>${profitHistory[index]}</td>
      <td><button class="delete-btn" onclick="deleteRecord(${index})">Delete</button></td>
    `;
  }

  // Update table, optionally filtered by indices
  function updateHistoryTable(filteredIndices = null) {
    const tbody = document.getElementById("historyTable").getElementsByTagName("tbody")[0];
    tbody.innerHTML = '';
    const indicesToShow = filteredIndices || labels.map((_, i) => i);
    indicesToShow.forEach(i => addRowToTable(i));
  }

  // Update chart, optionally filtered by indices
  let chart;
  function updateChart(filteredIndices = null) {
    const ctx = document.getElementById('profitChart').getContext('2d');
    if (chart) chart.destroy();

    let chartLabels, chartData;
    if (filteredIndices) {
      chartLabels = filteredIndices.map(i => labels[i]);
      chartData = filteredIndices.map(i => profitHistory[i]);
    } else {
      chartLabels = labels;
      chartData = profitHistory;
    }

    chart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: chartLabels,
        datasets: [{
          label: 'Profit (LKR)',
          data: chartData,
          backgroundColor: 'rgba(76, 175, 80, 0.2)',
          borderColor: 'rgba(76, 175, 80, 1)',
          borderWidth: 2,
          fill: true,
          tension: 0.1
        }]
      },
      options: {
        responsive: true,
        scales: {
          y: { beginAtZero: true }
        }
      }
    });
  }

  // Delete a record
  function deleteRecord(index) {
    labels.splice(index, 1);
    salesHistory.splice(index, 1);
    expensesHistory.splice(index, 1);
    profitHistory.splice(index, 1);
    saveUserData();
    populateMonthFilter();
    updateHistoryTable();
    updateChart();
    updateTotalBalanceDisplay();
    document.getElementById('result').innerHTML = '';
  }

  // Clear all records
  function clearAllRecords() {
    if (confirm('Are you sure you want to delete all records?')) {
      labels = [];
      salesHistory = [];
      expensesHistory = [];
      profitHistory = [];
      saveUserData();
      populateMonthFilter();
      updateHistoryTable();
      updateChart();
      updateTotalBalanceDisplay();
      document.getElementById('result').innerHTML = '';
    }
  }

  // Month filter functions
  function populateMonthFilter() {
    const filter = document.getElementById('monthFilter');
    const monthsSet = new Set(labels.map(d => {
      const dt = new Date(d);
      return dt.getFullYear() + '-' + String(dt.getMonth() + 1).padStart(2, '0');
    }));
    const options = ['<option value="all">All Records</option>'];
    Array.from(monthsSet).sort((a,b) => b.localeCompare(a)).forEach(m => {
      options.push(`<option value="${m}">${m}</option>`);
    });
    filter.innerHTML = options.join('');
  }

  function applyMonthFilter() {
    const filter = document.getElementById('monthFilter').value;
    if (filter === 'all') {
      updateHistoryTable();
      updateChart();
      updateTotalBalanceDisplay();
      return;
    }
    const filteredIndices = labels.reduce((arr, d, i) => {
      const dt = new Date(d);
      const m = dt.getFullYear() + '-' + String(dt.getMonth() + 1).padStart(2, '0');
      if (m === filter) arr.push(i);
      return arr;
    }, []);
    updateHistoryTable(filteredIndices);
    updateChart(filteredIndices);
    updateTotalBalanceDisplayFiltered(filteredIndices);
  }

  // Show total account balance (all data)
  function updateTotalBalanceDisplay() {
    const totalBalance = profitHistory.reduce((acc, val) => acc + val, 0);
    let balanceColor = 'green';
    if (totalBalance <= 0) balanceColor = 'red';

    document.getElementById('result').innerHTML = `
      <strong>Total Account Balance:</strong> <span style="color:${balanceColor}; font-size:1.2em;">LKR ${totalBalance.toFixed(2)}</span>
    `;
  }

  // Show total balance for filtered data
  function updateTotalBalanceDisplayFiltered(indices) {
    const totalBalance = indices.reduce((acc, i) => acc + profitHistory[i], 0);
    let balanceColor = 'green';
    if (totalBalance <= 0) balanceColor = 'red';

    document.getElementById('result').innerHTML = `
      <strong>Total Account Balance (Filtered):</strong> <span style="color:${balanceColor}; font-size:1.2em;">LKR ${totalBalance.toFixed(2)}</span>
    `;
  }
</script>

</body>
</html>

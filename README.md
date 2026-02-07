<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sahiwal Dairy Farm - Andauli</title>
    <style>
        :root { --primary: #1b5e20; --secondary: #2e7d32; --expense: #c62828; --bg: #f0f4f1; }
        body { font-family: 'Segoe UI', Arial, sans-serif; margin: 0; background-color: var(--bg); color: #2c3e50; padding-bottom: 50px; }
        header { background: var(--primary); color: white; padding: 1rem; text-align: center; }
        .container { max-width: 900px; margin: 15px auto; padding: 15px; background: white; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .summary-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px; }
        .card { padding: 15px; border-radius: 8px; color: white; text-align: center; }
        .card-milk { background: var(--secondary); }
        .card-expense { background: var(--expense); }
        .tabs { display: flex; margin-bottom: 20px; border-bottom: 2px solid #ddd; }
        .tab { flex: 1; padding: 10px; text-align: center; cursor: pointer; font-weight: bold; background: #eee; }
        .tab.active { background: var(--secondary); color: white; }
        .tracker-section { display: none; }
        .tracker-section.active { display: block; }
        .input-group { display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 15px; }
        input, select { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 5px; min-width: 120px; }
        .btn { padding: 10px 15px; border-radius: 5px; border: none; cursor: pointer; font-weight: bold; }
        .add-btn { background: var(--secondary); color: white; }
        .expense-btn { background: var(--expense); color: white; }
        .download-btn { background: #1976d2; color: white; width: 100%; margin-top: 10px; }
        .delete-btn { background: #ff5252; color: white; padding: 3px 8px; font-size: 0.8rem; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 0.9rem; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
        th { background: #f4f4f4; }
    </style>
</head>
<body>
<header>
    <h1>Sahiwal Farm Management</h1>
    <div style="font-size: 0.9rem;">📍 Andauli, Darbhanga, Bihar</div>
</header>
<div class="container">
    <div class="summary-grid">
        <div class="card card-milk">
            <small>Total Milk</small><br>
            <span id="statMilk">0</span> L
        </div>
        <div class="card card-expense">
            <small>Total Expense</small><br>
            ₹ <span id="statExpense">0</span>
        </div>
    </div>
    <div class="tabs">
        <div class="tab active" onclick="showTab('milk')">Milk Records</div>
        <div class="tab" onclick="showTab('expense')">Expenses</div>
    </div>
    <div id="milkSection" class="tracker-section active">
        <div class="input-group">
            <input type="date" id="milkDate">
            <input type="text" id="cowName" placeholder="Cow Name">
            <input type="number" id="liters" placeholder="Liters">
            <button class="btn add-btn" onclick="addItem('milk')">Save Milk</button>
        </div>
        <table id="milkTable">
            <thead><tr><th>Date</th><th>Cow</th><th>Liters</th><th>Action</th></tr></thead>
            <tbody id="milkBody"></tbody>
        </table>
    </div>
    <div id="expenseSection" class="tracker-section">
        <div class="input-group">
            <input type="date" id="expDate">
            <select id="expCategory">
                <option value="Feed (Chara)">Feed (Chara)</option>
                <option value="Medicine">Medicine</option>
                <option value="Labor">Labor</option>
                <option value="Other">Other</option>
            </select>
            <input type="number" id="amount" placeholder="Amount (₹)">
            <button class="btn expense-btn" onclick="addItem('expense')">Save Expense</button>
        </div>
        <table id="expenseTable">
            <thead><tr><th>Date</th><th>Category</th><th>Amount</th><th>Action</th></tr></thead>
            <tbody id="expenseBody"></tbody>
        </table>
    </div>
    <button class="btn download-btn" onclick="downloadData()">⬇️ Download All Data (CSV)</button>
    <button class="btn" style="width:100%; margin-top:5px; background:#607d8b; color:white;" onclick="clearAll()">🗑️ Reset All Data</button>
</div>
<script>
    let data = JSON.parse(localStorage.getItem('farmData')) || { milk: [], expense: [] };
    window.onload = () => {
        document.getElementById('milkDate').valueAsDate = new Date();
        document.getElementById('expDate').valueAsDate = new Date();
        refreshUI();
    };
    function showTab(type) {
        document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        document.querySelectorAll('.tracker-section').forEach(s => s.classList.remove('active'));
        event.currentTarget.classList.add('active');
        document.getElementById(type + 'Section').classList.add('active');
    }
    function addItem(type) {
        if (type === 'milk') {
            const date = document.getElementById('milkDate').value;
            const name = document.getElementById('cowName').value;
            const liters = parseFloat(document.getElementById('liters').value);
            if(!date || !name || isNaN(liters)) return alert("Check inputs");
            data.milk.unshift({ id: Date.now(), date, name, liters });
        } else {
            const date = document.getElementById('expDate').value;
            const cat = document.getElementById('expCategory').value;
            const amount = parseFloat(document.getElementById('amount').value);
            if(!date || isNaN(amount)) return alert("Check inputs");
            data.expense.unshift({ id: Date.now(), date, cat, amount });
        }
        saveAndRefresh();
    }
    function removeItem(type, id) {
        data[type] = data[type].filter(item => item.id !== id);
        saveAndRefresh();
    }
    function saveAndRefresh() {
        localStorage.setItem('farmData', JSON.stringify(data));
        refreshUI();
    }
    function refreshUI() {
        const mBody = document.getElementById('milkBody');
        const eBody = document.getElementById('expenseBody');
        mBody.innerHTML = ''; eBody.innerHTML = '';
        let totalMilk = 0;
        data.milk.forEach(r => {
            totalMilk += r.liters;
            mBody.innerHTML += `<tr><td>${r.date}</td><td>${r.name}</td><td>${r.liters}L</td><td><button class="delete-btn" onclick="removeItem('milk', ${r.id})">X</button></td></tr>`;
        });
        let totalExp = 0;
        data.expense.forEach(e => {
            totalExp += e.amount;
            eBody.innerHTML += `<tr><td>${e.date}</td><td>${e.cat}</td><td>₹${e.amount}</td><td><button class="delete-btn" onclick="removeItem('expense', ${e.id})">X</button></td></tr>`;
        });
        document.getElementById('statMilk').innerText = totalMilk.toFixed(1);
        document.getElementById('statExpense').innerText = totalExp.toFixed(0);
    }
    function downloadData() {
        let csv = "TYPE,DATE,DETAIL,VALUE\n";
        data.milk.forEach(r => csv += `MILK,${r.date},${r.name},${r.liters}\n`);
        data.expense.forEach(e => csv += `EXPENSE,${e.date},${e.cat},${e.amount}\n`);
        const blob = new Blob([csv], { type: 'text/csv' });
        const a = document.createElement('a');
        a.href = URL.createObjectURL(blob);
        a.download = 'Andauli_Farm_Full_Report.csv';
        a.click();
    }
    function clearAll() {
        if(confirm("Delete everything?")) {
            data = { milk: [], expense: [] };
            saveAndRefresh();
        }
    }
</script>
</body>
</html>

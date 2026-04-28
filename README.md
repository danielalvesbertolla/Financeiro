<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financial Goals Tracker</title>
    <style>
        body { font-family: Arial, sans-serif; }
        #expense-list { margin-top: 20px; }
        .fixed { color: blue; }
        .variable { color: green; }
        #notification { display: none; }
    </style>
</head>
<body>
    <h1>Financial Goals Tracker</h1>
    <form id="expense-form">
        <input type="text" id="expense-name" placeholder="Expense Name" required />
        <input type="number" id="expense-amount" placeholder="Amount" required />
        <select id="expense-type">
            <option value="fixed">Fixed Expense</option>
            <option value="variable">Variable Expense</option>
        </select>
        <button type="submit">Add Expense</button>
    </form>
    <div id="expense-list">
        <h2>Expenses</h2>
        <ul id="expenses"></ul>
    </div>
    <button id="reallocate">Reallocate Expenses</button>
    <div id="notification">Daily Bruto notification dialog!</div>
    <script>
        const expenses = [];
        document.getElementById('expense-form').onsubmit = function(e) {
            e.preventDefault();
            const name = document.getElementById('expense-name').value;
            const amount = parseFloat(document.getElementById('expense-amount').value);
            const type = document.getElementById('expense-type').value;
            expenses.push({name, amount, type});
            updateExpenseList();
            document.getElementById('expense-form').reset();
        };

        function updateExpenseList() {
            const expensesList = document.getElementById('expenses');
            expensesList.innerHTML = '';
            expenses.forEach(expense => {
                const li = document.createElement('li');
                li.className = expense.type;
                li.textContent = `${expense.name}: $${expense.amount}`;
                expensesList.appendChild(li);
            });
        }

        document.getElementById('reallocate').onclick = function() {
            // Logic for percentage redistribution
            // Priority-based reallocation would go here
        };

        function notifyDailyBruto() {
            document.getElementById('notification').style.display = 'block';
        }

        function calculateDizimo() {
            const dizimo = expenses.filter(exp => exp.name !== 'Fuel').reduce((sum, exp) => sum + exp.amount, 0) * 0.1;
            return dizimo;
        }

        function undoLast() {
            expenses.pop();
            updateExpenseList();
        }
    </script>
</body>
</html>

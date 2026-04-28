<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financial Goals Tracker</title>
    <style>
        /* Add necessary styles for the tracker */
        body { font-family: Arial, sans-serif; }
        .completed { text-decoration: line-through; }
    </style>
</head>
<body>
    <h1>Financial Goals Tracker</h1>
    <div id="tracker">
        <form id="expense-form">
            <input type="text" id="expense-name" placeholder="Expense Name" required />
            <input type="number" id="expense-amount" placeholder="Amount" required />
            <button type="submit">Add Expense</button>
        </form>
        <h2>Expenses</h2>
        <ul id="expense-list"></ul>
        <h2>Total Expenses: <span id="total-expenses">0</span></h2>
        <h2>Remaining Income: <span id="remaining-income">0</span></h2>
        <button id="calculate-tithe">Calculate Tithe</button>
        <div id="tithe-dialog" style="display:none;">
            <h3>Tithe Calculation</h3>
            <p>Your tithe is <span id="tithe-amount"></span></p>
            <button id="close-dialog">Close</button>
        </div>
    </div>
    <script>
        let expenses = [];
        let totalExpenses = 0;

        document.getElementById('expense-form').addEventListener('submit', function(event) {
            event.preventDefault();
            const name = document.getElementById('expense-name').value;
            const amount = parseFloat(document.getElementById('expense-amount').value);
            addExpense(name, amount);
            updateTotalExpenses();
            document.getElementById('expense-form').reset();
        });

        function addExpense(name, amount) {
            expenses.push({ name, amount, completed: false });
            renderExpenses();
        }

        function renderExpenses() {
            const list = document.getElementById('expense-list');
            list.innerHTML = '';
            expenses.forEach((expense, index) => {
                const item = document.createElement('li');
                item.textContent = expense.name + ': $' + expense.amount;
                if (expense.completed) item.classList.add('completed');
                item.addEventListener('click', function() {
                    expense.completed = !expense.completed;
                    updateTotalExpenses();
                    renderExpenses();
                });
                list.appendChild(item);
            });
        }

        function updateTotalExpenses() {
            totalExpenses = expenses.reduce((total, expense) => total + (expense.completed ? expense.amount : 0), 0);
            document.getElementById('total-expenses').textContent = totalExpenses;
            calculateRemainingIncome();
        }

        function calculateRemainingIncome() {
            // Assume a fixed income (replace with actual value)
            const fixedIncome = 5000;
            const remainingIncome = fixedIncome - totalExpenses;
            document.getElementById('remaining-income').textContent = remainingIncome;
        }

        document.getElementById('calculate-tithe').addEventListener('click', function() {
            const remainingIncome = parseFloat(document.getElementById('remaining-income').textContent);
            const tithe = remainingIncome * 0.10;
            document.getElementById('tithe-amount').textContent = '$' + tithe.toFixed(2);
            document.getElementById('tithe-dialog').style.display = 'block';
        });

        document.getElementById('close-dialog').addEventListener('click', function() {
            document.getElementById('tithe-dialog').style.display = 'none';
        });
    </script>
</body>
</html>

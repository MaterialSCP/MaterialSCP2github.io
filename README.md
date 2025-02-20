<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Materialverwaltung</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 20px; max-width: 800px; margin: auto; background-color: white; color: black; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid black; padding: 8px; text-align: left; }
        .hidden { display: none; }
        #adminLoginBtn { position: absolute; top: 10px; right: 10px; font-size: 12px; padding: 5px; transition: all 0.3s ease; }
        #adminLoginBtn:hover { font-size: 16px; padding: 10px; }
    </style>
</head>
<body>
    <h1>Materialverwaltung</h1>
    <button id="adminLoginBtn" onclick="toggleLogin()">Admin Login</button>
    <div id="adminLogin" class="hidden">
        <input type="password" id="adminPassword" placeholder="Passwort eingeben">
        <button onclick="handleLogin()">Anmelden</button>
    </div>
    <div id="adminPanel" class="hidden">
        <h2>Material hinzufügen</h2>
        <input type="text" id="newMaterial" placeholder="Materialname">
        <input type="number" id="materialQuantity" placeholder="Menge" min="1" value="1">
        <button onclick="addMaterial()">Hinzufügen</button>
        <h2>Material verwalten</h2>
        <ul id="availableMaterials" style="list-style-type: none;"></ul>
    </div>
    <h2>Materialanfrage</h2>
    <select id="materialSelect">
        <option value="" disabled selected>Wähle ein Material</option>
    </select>
    <input type="number" id="borrowQuantity" placeholder="Menge" min="1" value="1">
    <input type="text" id="borrower" placeholder="Entleiher">
    <input type="date" id="loanDate">
    <input type="date" id="returnDate">
    <button onclick="requestItem()">Anfrage senden</button>
    <h2>Verfügbare Materialien</h2>
    <ul id="publicAvailableMaterials" style="list-style-type: none;"></ul>
    <h2>Verliehenes Material</h2>
    <table id="loanedTable"></table>

    <script>
        let isAdmin = false;
        let availableMaterials = JSON.parse(localStorage.getItem('availableMaterials') || '{}');
        let loanedItems = JSON.parse(localStorage.getItem('loanedItems') || '[]');

        function saveData() {
            localStorage.setItem('availableMaterials', JSON.stringify(availableMaterials));
            localStorage.setItem('loanedItems', JSON.stringify(loanedItems));
        }

        function toggleLogin() {
            document.getElementById("adminLogin").classList.toggle("hidden");
        }

        function handleLogin() {
            let password = document.getElementById("adminPassword").value;
            if (password === "SCP07!") {
                isAdmin = true;
                document.getElementById("adminPanel").classList.remove("hidden");
                updateLoanedList();
            } else {
                alert("Falsches Passwort");
            }
        }

        function addMaterial() {
            let material = document.getElementById("newMaterial").value;
            let quantity = parseInt(document.getElementById("materialQuantity").value);
            if (!material || quantity < 1) return;
            availableMaterials[material] = (availableMaterials[material] || 0) + quantity;
            saveData();
            updateMaterialList();
        }

        function requestItem() {
            let material = document.getElementById("materialSelect").value;
            let quantity = parseInt(document.getElementById("borrowQuantity").value);
            let borrower = document.getElementById("borrower").value;
            let loanDate = document.getElementById("loanDate").value;
            let returnDate = document.getElementById("returnDate").value;
            
            if (!material || !borrower || quantity < 1 || !loanDate || !returnDate) {
                alert("Bitte alle Felder ausfüllen.");
                return;
            }

            if (availableMaterials[material] < quantity) {
                alert("Nicht genügend Material verfügbar.");
                return;
            }

            availableMaterials[material] -= quantity;
            let existingLoan = loanedItems.find(item => item.material === material && item.borrower === borrower && item.loanDate === loanDate && item.returnDate === returnDate);
            if (existingLoan) {
                existingLoan.quantity += quantity;
            } else {
                loanedItems.push({ material, quantity, borrower, loanDate, returnDate });
            }
            saveData();
            updateMaterialList();
            updateLoanedList();
        }

        function returnItem(index) {
            let item = loanedItems[index];
            availableMaterials[item.material] = (availableMaterials[item.material] || 0) + item.quantity;
            loanedItems.splice(index, 1);
            saveData();
            updateMaterialList();
            updateLoanedList();
        }

        function updateLoanedList() {
            let table = document.getElementById("loanedTable");
            table.innerHTML = `<tr><th>Menge</th><th>Material</th><th>Entleiher</th><th>Ausleihe</th><th>Rückgabe</th>${isAdmin ? '<th>Aktion</th>' : ''}</tr>`;
            loanedItems.forEach((item, index) => {
                let row = table.insertRow();
                row.innerHTML = `<td>${item.quantity}</td><td>${item.material}</td><td>${item.borrower}</td><td>${item.loanDate}</td><td>${item.returnDate}</td>${isAdmin ? `<td><button onclick="returnItem(${index})">Zurückgeben</button></td>` : ''}`;
            });
        }

        function updateMaterialList() {
            let list = document.getElementById("availableMaterials");
            let publicList = document.getElementById("publicAvailableMaterials");
            let materialSelect = document.getElementById("materialSelect");
            
            list.innerHTML = "";
            publicList.innerHTML = "";
            materialSelect.innerHTML = "<option value=\"\" disabled selected>Wähle ein Material</option>";
            
            for (let mat in availableMaterials) {
                let li = document.createElement("li");
                li.innerHTML = `${availableMaterials[mat]}x ${mat}`;
                list.appendChild(li);
                publicList.appendChild(li.cloneNode(true));
                materialSelect.appendChild(new Option(`${availableMaterials[mat]}x ${mat}`, mat));
            }
        }

        updateMaterialList();
        updateLoanedList();
    </script>
</body>
</html>

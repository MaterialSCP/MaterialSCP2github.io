<!DOCTYPE html>
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
        <h2>Material entfernen</h2>
        <select id="removeMaterialSelect"></select>
        <input type="number" id="removeMaterialQuantity" placeholder="Menge" min="1" value="1">
        <button onclick="removeMaterial()">Entfernen</button>
    </div>
    <h2>Materialanfrage</h2>
    <select id="materialSelect"></select>
    <input type="number" id="borrowQuantity" placeholder="Menge" min="1" value="1">
    <input type="text" id="borrower" placeholder="Entleiher">
    <input type="date" id="loanDate">
    <input type="date" id="returnDate">
    <button onclick="requestItem()">Anfrage senden</button>
    <h2>Verfügbare Materialien</h2>
    <ul id="publicAvailableMaterials"></ul>
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
                updateMaterialList();
                updateLoanedList();
            } else {
                alert("Falsches Passwort");
            }
        }
        
        function addMaterial() {
            let material = document.getElementById("newMaterial").value.trim();
            let quantity = parseInt(document.getElementById("materialQuantity").value);
            if (!material || quantity < 1) return;
            availableMaterials[material] = (availableMaterials[material] || 0) + quantity;
            saveData();
            updateMaterialList();
        }
        
        function removeMaterial() {
            let material = document.getElementById("removeMaterialSelect").value;
            let quantity = parseInt(document.getElementById("removeMaterialQuantity").value);
            if (!material || quantity < 1) return;
            availableMaterials[material] -= quantity;
            if (availableMaterials[material] <= 0) delete availableMaterials[material];
            saveData();
            updateMaterialList();
        }
        
        function requestItem() {
            let material = document.getElementById("materialSelect").value;
            let quantity = parseInt(document.getElementById("borrowQuantity").value);
            let borrower = document.getElementById("borrower").value.trim();
            let loanDate = document.getElementById("loanDate").value;
            let returnDate = document.getElementById("returnDate").value;
            if (!material || quantity < 1 || !borrower || !loanDate || !returnDate) return;
            if (availableMaterials[material] < quantity) {
                alert("Nicht genügend Material verfügbar");
                return;
            }
            availableMaterials[material] -= quantity;
            loanedItems.push({ material, quantity, borrower, loanDate, returnDate });
            saveData();
            updateMaterialList();
            updateLoanedList();
        }
        
        function returnLoanedMaterial(index) {
            if (!isAdmin) return;
            let item = loanedItems.splice(index, 1)[0];
            availableMaterials[item.material] = (availableMaterials[item.material] || 0) + item.quantity;
            saveData();
            updateMaterialList();
            updateLoanedList();
        }
        
        function updateLoanedList() {
            let table = document.getElementById("loanedTable");
            table.innerHTML = "<tr><th>Material</th><th>Menge</th><th>Entleiher</th><th>Ausleihe</th><th>Rückgabe</th>" + (isAdmin ? "<th>Aktion</th>" : "") + "</tr>";
            loanedItems.forEach((item, index) => {
                let row = table.insertRow();
                row.innerHTML = `<td>${item.material}</td><td>${item.quantity}</td><td>${item.borrower}</td><td>${item.loanDate}</td><td>${item.returnDate}</td>` + (isAdmin ? `<td><button onclick='returnLoanedMaterial(${index})'>Zurückgeben</button></td>` : "");
            });
        }
        
        function updateMaterialList() {
            let materialSelect = document.getElementById("materialSelect");
            let publicList = document.getElementById("publicAvailableMaterials");
            let removeMaterialSelect = document.getElementById("removeMaterialSelect");
            materialSelect.innerHTML = '<option value="" disabled selected>Wähle ein Material</option>';
            publicList.innerHTML = '';
            removeMaterialSelect.innerHTML = '';
            
            for (let material in availableMaterials) {
                let option = document.createElement("option");
                option.value = material;
                option.textContent = `${material} (${availableMaterials[material]})`;
                materialSelect.appendChild(option);
                
                let removeOption = document.createElement("option");
                removeOption.value = material;
                removeOption.textContent = material;
                removeMaterialSelect.appendChild(removeOption);
                
                let listItem = document.createElement("li");
                listItem.textContent = `${material}: ${availableMaterials[material]}`;
                publicList.appendChild(listItem);
            }
            
            if (Object.keys(availableMaterials).length === 0) {
                publicList.innerHTML = "Keine Materialien verfügbar.";
            }
        }
        
        updateMaterialList();
        updateLoanedList();
    </script>
</body>
</html>

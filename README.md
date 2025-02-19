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
        <ul id="availableMaterials"></ul>
    </div>
    <h2>Materialanfrage</h2>
    <select id="materialSelect"></select>
    <input type="number" id="borrowQuantity" placeholder="Menge" min="1" value="1">
    <input type="text" id="borrower" placeholder="Entleiher">
    <input type="date" id="loanDate">
    <input type="date" id="returnDate">
    <button onclick="requestItem()">Anfrage senden</button>
    <h2>Verliehenes Material</h2>
    <table id="loanedTable"></table>

    <script>
        let isAdmin = false;
        let availableMaterials = JSON.parse(localStorage.getItem("availableMaterials")) || {};
        let loanedItems = JSON.parse(localStorage.getItem("loanedItems")) || [];

        function toggleLogin() {
            document.getElementById("adminLogin").classList.toggle("hidden");
        }

        function handleLogin() {
            let password = document.getElementById("adminPassword").value;
            if (password === "SCP07!") {
                isAdmin = true;
                document.getElementById("adminPanel").classList.remove("hidden");
            } else {
                alert("Falsches Passwort");
            }
        }

        function addMaterial() {
            let material = document.getElementById("newMaterial").value;
            let quantity = parseInt(document.getElementById("materialQuantity").value);
            if (!material || quantity < 1) return;
            availableMaterials[material] = (availableMaterials[material] || 0) + quantity;
            localStorage.setItem("availableMaterials", JSON.stringify(availableMaterials));
            updateMaterialList();
        }

        function removeMaterial(material) {
            if (availableMaterials[material] > 1) {
                availableMaterials[material]--;
            } else {
                delete availableMaterials[material];
            }
            localStorage.setItem("availableMaterials", JSON.stringify(availableMaterials));
            updateMaterialList();
        }

        function updateMaterialList() {
            let list = document.getElementById("availableMaterials");
            list.innerHTML = "";
            for (let mat in availableMaterials) {
                let li = document.createElement("li");
                li.innerHTML = `${mat} (${availableMaterials[mat]}) <button onclick='removeMaterial("${mat}")'>Löschen</button>`;
                list.appendChild(li);
            }
            let select = document.getElementById("materialSelect");
            select.innerHTML = "<option value=''>Material auswählen</option>";
            for (let mat in availableMaterials) {
                let option = document.createElement("option");
                option.value = mat;
                option.textContent = `${mat} (${availableMaterials[mat]})`;
                select.appendChild(option);
            }
        }

        function requestItem() {
            let material = document.getElementById("materialSelect").value;
            let quantity = parseInt(document.getElementById("borrowQuantity").value);
            let borrower = document.getElementById("borrower").value;
            let loanDate = document.getElementById("loanDate").value;
            let returnDate = document.getElementById("returnDate").value;
            
            if (!material || quantity < 1 || !borrower || !loanDate || !returnDate) return;
            
            if (availableMaterials[material] >= quantity) {
                availableMaterials[material] -= quantity;
                if (availableMaterials[material] === 0) {
                    delete availableMaterials[material];
                }
                loanedItems.push({ material, quantity, borrower, loanDate, returnDate });
                localStorage.setItem("availableMaterials", JSON.stringify(availableMaterials));
                localStorage.setItem("loanedItems", JSON.stringify(loanedItems));
                updateMaterialList();
            } else {
                alert("Nicht genügend Material verfügbar");
            }
        }

        updateMaterialList();
    </script>
</body>
</html>

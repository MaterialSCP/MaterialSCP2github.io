<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Materialverwaltung</title>
    <script src="https://unpkg.com/@supabase/supabase-js"></script>
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
    </div>
    <h2>Verfügbare Materialien</h2>
    <ul id="availableMaterials" style="list-style-type: none;"></ul>
    <h2>Materialanfrage</h2>
    <select id="materialSelect">
        <option value="" disabled selected>Wähle ein Material</option>
    </select>
    <input type="number" id="borrowQuantity" placeholder="Menge" min="1" value="1">
    <input type="text" id="borrower" placeholder="Entleiher">
    <input type="date" id="loanDate">
    <input type="date" id="returnDate">
    <button onclick="requestItem()">Anfrage senden</button>
    <h2>Verliehenes Material</h2>
    <table id="loanedTable"></table>

    <script>
        const supabaseUrl = "https://xyzcompany.supabase.co";  // Ersetze durch deine Supabase-URL
        const supabaseKey = "public-anon-key";  // Ersetze durch deinen API-Schlüssel
        const supabase = supabase.createClient(supabaseUrl, supabaseKey);

        let isAdmin = false;

        async function fetchMaterials() {
            let { data, error } = await supabase.from("materials").select("*");
            if (error) return console.error(error);
            updateMaterialList(data);
        }

        async function addMaterial() {
            let material = document.getElementById("newMaterial").value;
            let quantity = parseInt(document.getElementById("materialQuantity").value);
            if (!material || quantity < 1) return;
            await supabase.from("materials").insert([{ name: material, quantity: quantity }]);
            fetchMaterials();
        }

        async function requestItem() {
            let material = document.getElementById("materialSelect").value;
            let quantity = parseInt(document.getElementById("borrowQuantity").value);
            let borrower = document.getElementById("borrower").value;
            let loanDate = document.getElementById("loanDate").value;
            let returnDate = document.getElementById("returnDate").value;
            
            if (!material || !borrower || quantity < 1 || !loanDate || !returnDate) {
                alert("Bitte alle Felder ausfüllen.");
                return;
            }
            await supabase.from("loans").insert([{ material, quantity, borrower, loanDate, returnDate }]);
            fetchMaterials();
        }

        async function fetchLoanedItems() {
            let { data, error } = await supabase.from("loans").select("*");
            if (error) return console.error(error);
            updateLoanedList(data);
        }

        async function returnItem(id) {
            await supabase.from("loans").delete().match({ id: id });
            fetchLoanedItems();
            fetchMaterials();
        }

        function updateMaterialList(materials) {
            let list = document.getElementById("availableMaterials");
            let materialSelect = document.getElementById("materialSelect");
            list.innerHTML = "";
            materialSelect.innerHTML = "<option value=\"\" disabled selected>Wähle ein Material</option>";
            
            materials.forEach(mat => {
                let li = document.createElement("li");
                li.innerHTML = `${mat.quantity}x ${mat.name}`;
                list.appendChild(li);
                materialSelect.appendChild(new Option(`${mat.quantity}x ${mat.name}`, mat.name));
            });
        }

        function updateLoanedList(loanedItems) {
            let table = document.getElementById("loanedTable");
            table.innerHTML = `<tr><th>Menge</th><th>Material</th><th>Entleiher</th><th>Ausleihe</th><th>Rückgabe</th>${isAdmin ? '<th>Aktion</th>' : ''}</tr>`;
            loanedItems.forEach(item => {
                let row = table.insertRow();
                row.innerHTML = `<td>${item.quantity}</td><td>${item.material}</td><td>${item.borrower}</td><td>${item.loanDate}</td><td>${item.returnDate}</td>${isAdmin ? `<td><button onclick="returnItem(${item.id})">Zurückgeben</button></td>` : ''}`;
            });
        }

        function toggleLogin() {
            document.getElementById("adminLogin").classList.toggle("hidden");
        }

        function handleLogin() {
            let password = document.getElementById("adminPassword").value;
            if (password === "SCP07!") {
                isAdmin = true;
                document.getElementById("adminPanel").classList.remove("hidden");
                fetchLoanedItems();
            } else {
                alert("Falsches Passwort");
            }
        }

        fetchMaterials();
        fetchLoanedItems();
    </script>
</body>
</html>

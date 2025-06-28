<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>ELCOIN Wallet</title>
  <style>
    body { font-family: Arial, sans-serif; background: #eef2f7; padding: 20px; text-align: center; }
    .box { background: white; padding: 20px; border-radius: 12px; max-width: 600px; margin: auto; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
    input, button, select { padding: 10px; margin: 10px 0; width: 100%; border-radius: 6px; border: 1px solid #ccc; }
    button { background: #0984e3; color: white; font-weight: bold; border: none; cursor: pointer; }
    .info { text-align: left; margin-top: 15px; font-size: 14px; }
    .qr { margin: 10px auto; }
    .success { color: green; }
    .error { color: red; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: center; font-size: 13px; }
    .pending { background: #f9f9c5; padding: 5px; margin-top: 5px; border-radius: 5px; }
  </style>
</head>
<body>

<div class="box" id="loginBox">
  <h2>Connexion ELCOIN</h2>
  <input type="text" id="username" placeholder="Numéro de téléphone">
  <input type="number" id="userpass" placeholder="Mot de passe (chiffres uniquement)">
  <button onclick="login()">Connexion</button>
  <p id="loginMsg"></p>
</div>

<div class="box" id="dashboard" style="display:none">
  <h2>ELCOIN Wallet</h2>
  <p><strong>Solde ELCOIN:</strong> <span id="balance">60</span> EL</p>
  <div class="info">
    <p><strong>Cours:</strong> 1 EL = 100 MGA</p>
    <p><strong>Valeur en MGA:</strong> <span id="valueMGA"></span> Ar</p>
    <p><strong>Valeur en USDT:</strong> <span id="valueUSD"></span> USDT</p>
  </div>
  <h3>Dépôt ELCOIN</h3>
  <p>Adresse Dépôt: <span id="elAddress">0x3eBB0f120555E8BCE089040bCCF974ae515e5F34</span></p>
  <button onclick="copyToClipboard('elAddress')">Copier Adresse</button>
  <input type="text" id="txId" placeholder="Tx ID ou Hash de Transaction">
  <button onclick="confirmDeposit()">Confirmer Dépôt</button>

  <h3>Dépôt Orange Money</h3>
  <p>Numéro Dépôt: <span id="omNumber">0327240893</span></p>
  <button onclick="copyToClipboard('omNumber')">Copier Numéro</button>
  <input type="text" id="omTx" placeholder="ID de Transaction Orange Money">
  <button onclick="confirmOM()">Confirmer Dépôt OM</button>

  <h3>Retrait</h3>
  <select id="retType">
    <option value="EL">Retrait ELCOIN</option>
    <option value="OM">Retrait MGA (Orange Money)</option>
  </select>
  <input type="text" id="retDest" placeholder="Adresse EL ou Numéro OM">
  <input type="number" id="retAmount" placeholder="Montant EL à retirer">
  <button onclick="submitWithdraw()">Demander Retrait</button>
  <div class="info">Minimum: 50 EL | Max: 95% du solde | Frais: 5% automatique</div>

  <h3>Staking</h3>
  <select id="stakePlan">
    <option value="7">7 jours - 2%</option>
    <option value="15">15 jours - 5%</option>
    <option value="30">30 jours - 10%</option>
    <option value="90">3 mois - 15%</option>
    <option value="180">6 mois - 25%</option>
    <option value="270">9 mois - 35%</option>
    <option value="360">12 mois - 50%</option>
  </select>
  <input type="number" id="stakeAmount" placeholder="Montant EL à staker">
  <button onclick="stake()">Valider Staking</button>
  <div class="info">Minimum de staking: 1000 EL</div>

  <h3>Historique</h3>
  <div id="history"></div>
</div>

<script>
  let balance = 60;
  const coursEL = 100;
  const usdRate = 4600;
  let pending = [];
  const minDepositEL = 100;
  const minWithdrawEL = 50;
  const maxWithdrawPercent = 0.95;
  const minStakeEL = 1000;
  const allowedUsers = [
    { user: "0327240893", pass: "1234" },
    { user: "0345678901", pass: "4321" }
  ];

  function login() {
    const u = document.getElementById("username").value.trim();
    const p = document.getElementById("userpass").value.trim();
    const found = allowedUsers.find(acc => acc.user === u && acc.pass === p);
    if (found) {
      document.getElementById("loginBox").style.display = "none";
      document.getElementById("dashboard").style.display = "block";
      updateValues();
    } else {
      document.getElementById("loginMsg").innerHTML = '<span class="error">Identifiants incorrects</span>';
    }
  }

  function updateValues() {
    document.getElementById("balance").innerText = balance;
    document.getElementById("valueMGA").innerText = (balance * coursEL).toLocaleString();
    document.getElementById("valueUSD").innerText = ((balance * coursEL) / usdRate).toFixed(2);
    renderHistory();
  }

  function copyToClipboard(id) {
    const text = document.getElementById(id).innerText;
    navigator.clipboard.writeText(text);
    alert("Copié: " + text);
  }

  function confirmDeposit() {
    const tx = document.getElementById("txId").value;
    if (tx) {
      balance += 100; // simulation d'ajout
      pending.push({ type: "Dépôt EL", tx, amount: 100 });
      updateValues();
    }
  }

  function confirmOM() {
    const tx = document.getElementById("omTx").value;
    if (tx) {
      balance += 100; // simulation d'ajout
      pending.push({ type: "Dépôt OM", tx, amount: 100 });
      updateValues();
    }
  }

  function submitWithdraw() {
    const type = document.getElementById("retType").value;
    const dest = document.getElementById("retDest").value;
    const amt = parseFloat(document.getElementById("retAmount").value);
    const max = Math.floor(balance * maxWithdrawPercent);
    const frais = Math.floor(amt * 0.05);
    if (!dest || amt < minWithdrawEL || amt > max) return alert("Montant invalide ou destination manquante.");
    balance -= amt;
    pending.push({ type: `Retrait ${type}`, dest, amount: amt - frais });
    updateValues();
  }

  function stake() {
    const plan = parseInt(document.getElementById("stakePlan").value);
    const amt = parseFloat(document.getElementById("stakeAmount").value);
    if (amt < minStakeEL || amt > balance) return alert("Montant invalide.");
    let rate = plan <= 7 ? 0.02 : plan <= 15 ? 0.05 : plan <= 30 ? 0.10 : plan <= 90 ? 0.15 : plan <= 180 ? 0.25 : plan <= 270 ? 0.35 : 0.5;
    let gain = amt * rate;
    balance -= amt;
    pending.push({ type: `Staking ${plan}j`, amount: amt, gain });
    updateValues();
  }

  function renderHistory() {
    const hist = pending.map(p => `<div class='pending'>${p.type} - ${p.amount} EL</div>`).join("");
    document.getElementById("history").innerHTML = hist;
  }
</script>

</body>
</html>

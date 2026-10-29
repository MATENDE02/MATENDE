<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="theme-color" content="#0757d5">
<title>MATENDE PAY — Épargne</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#f3f6fa;color:#172033;font-family:Arial,sans-serif}
.app{max-width:480px;margin:auto;min-height:100vh}.top{background:linear-gradient(135deg,#0757d5,#13a2ff);color:white;padding:24px 18px 30px;border-radius:0 0 28px 28px}
.logo{font-weight:800;font-size:27px}.subtitle{opacity:.9;margin-top:4px}
.card{background:white;border-radius:18px;padding:18px;margin:16px;box-shadow:0 5px 20px #00000010}
.balance{background:#ffffff18;padding:18px;border-radius:16px;margin-top:20px}.balance small{opacity:.8}.amount{font-size:30px;font-weight:bold;margin-top:6px}
h2{font-size:19px;margin:0 0 15px}label{display:block;font-size:13px;font-weight:bold;margin:12px 0 6px}
input{width:100%;padding:14px;border:1px solid #d7dde6;border-radius:11px;font-size:16px}
button{width:100%;border:0;border-radius:12px;padding:15px;background:#0757d5;color:white;font-size:16px;font-weight:bold;margin-top:15px}
button.secondary{background:#eaf1ff;color:#0757d5}.row{display:flex;justify-content:space-between;margin:8px 0}.summary{background:#f4f7fb;padding:13px;border-radius:12px;margin-top:14px}
.history{padding:12px 0;border-bottom:1px solid #eee}.history:last-child{border:0}
.success{text-align:center}.check{font-size:52px;color:#16a34a}.ref{font-family:monospace;background:#f0f5ff;padding:11px;border-radius:10px;margin:12px 0}
.notice{font-size:12px;color:#687386;line-height:1.45}
nav{position:sticky;bottom:0;background:white;border-top:1px solid #ddd;padding:8px;display:flex}nav button{margin:0 5px;background:white;color:#687386;font-size:12px;padding:9px}nav button.active{background:#eef5ff;color:#0757d5}
.hidden{display:none!important}
</style>
</head>
<body>
<div class="app">
<header class="top">
  <div class="logo">MATENDE PAY</div>
  <div class="subtitle">Application d'épargne digitale</div>
  <div class="balance">
    <small>Solde de l'épargnant</small>
    <div class="amount" id="balance">0 CDF</div>
  </div>
</header>

<section id="home">
<div class="card">
<h2>👤 Épargnant</h2>
<label>Nom complet</label><input id="name" placeholder="Nom, postnom et prénom">
<label>Numéro de téléphone</label><input id="phone" type="tel" placeholder="+243 ...">
<label>Numéro de compte</label><input id="account" readonly>
<button id="saveClient">Enregistrer l'épargnant</button>
</div>

<div class="card">
<h2>💰 Nouveau dépôt</h2>
<label>Montant du dépôt</label>
<input id="deposit" type="number" min="1" placeholder="Ex. 50000">
<button id="depositBtn">Effectuer le dépôt</button>
<p class="notice">Cette version enregistre les opérations localement sur l'appareil. L'envoi réel de SMS nécessite une API SMS côté serveur.</p>
</div>
</section>

<section id="confirmation" class="card hidden success">
<div class="check">✓</div>
<h2>Dépôt confirmé</h2>
<div id="message"></div>
<div class="ref" id="ref"></div>
<button id="printBtn">Imprimer le reçu</button>
<button class="secondary" id="closeConfirm">Nouveau dépôt</button>
</section>

<section id="historyPage" class="card hidden">
<h2>📜 Historique des dépôts</h2>
<div id="history"></div>
</section>

<nav>
<button class="active" id="homeNav">🏠<br>Accueil</button>
<button id="historyNav">📜<br>Historique</button>
</nav>
</div>

<script>
const $=id=>document.getElementById(id);
const money=n=>new Intl.NumberFormat("fr-FR").format(Number(n)||0)+" CDF";

let client=JSON.parse(localStorage.getItem("matendeClient")||"null");
let transactions=JSON.parse(localStorage.getItem("matendeTransactions")||"[]");
let balance=Number(localStorage.getItem("matendeBalance")||0);

function accountNumber(){
  let n=Number(localStorage.getItem("matendeAccountCounter")||1000)+1;
  localStorage.setItem("matendeAccountCounter",n);
  return "MP-EP-"+String(n).padStart(6,"0");
}
function refresh(){
  $("balance").textContent=money(balance);
  if(client){
    $("name").value=client.name;$("phone").value=client.phone;$("account").value=client.account;
  }else $("account").value="Sera attribué automatiquement";
}
refresh();

$("saveClient").onclick=()=>{
  const name=$("name").value.trim(),phone=$("phone").value.trim();
  if(!name||!phone){alert("Veuillez compléter le nom et le numéro de téléphone.");return}
  if(!client) client={name,phone,account:accountNumber()};
  else {client.name=name;client.phone=phone}
  localStorage.setItem("matendeClient",JSON.stringify(client));
  refresh();
  alert("Épargnant enregistré avec succès. Compte : "+client.account);
};

$("depositBtn").onclick=()=>{
  if(!client){alert("Enregistrez d'abord l'épargnant.");return}
  const amount=Number($("deposit").value)||0;
  if(amount<=0){alert("Entrez un montant de dépôt valide.");return}
  balance+=amount;
  localStorage.setItem("matendeBalance",balance);

  const now=new Date();
  const date=now.toLocaleString("fr-FR");
  const ref="DEP-"+now.getFullYear()+String(now.getMonth()+1).padStart(2,"0")+String(now.getDate()).padStart(2,"0")+"-"+Date.now().toString().slice(-6);
  const tx={ref,amount,date,balance};
  transactions.unshift(tx);
  localStorage.setItem("matendeTransactions",JSON.stringify(transactions));

  $("message").innerHTML=`Cher(e) <b>${client.name}</b>,<br><br>
  Votre dépôt de <b>${money(amount)}</b> a été enregistré avec succès.<br>
  Compte : <b>${client.account}</b><br>
  Nouveau solde : <b>${money(balance)}</b><br>
  Date : ${date}`;
  $("ref").textContent=ref;
  $("deposit").value="";
  $("home").classList.add("hidden");
  $("confirmation").classList.remove("hidden");
  refresh();
};

$("closeConfirm").onclick=()=>{
  $("confirmation").classList.add("hidden");$("home").classList.remove("hidden");
};
$("printBtn").onclick=()=>window.print();

function renderHistory(){
  $("history").innerHTML=transactions.length?transactions.map(t=>
    `<div class="history"><b>Dépôt : ${money(t.amount)}</b><br>
    <small>${t.date}<br>Référence : ${t.ref}<br>Solde après dépôt : ${money(t.balance)}</small></div>`
  ).join(""):"<p>Aucun dépôt enregistré.</p>";
}
$("historyNav").onclick=()=>{
  $("home").classList.add("hidden");$("confirmation").classList.add("hidden");
  $("historyPage").classList.remove("hidden");renderHistory();
};
$("homeNav").onclick=()=>{
  $("historyPage").classList.add("hidden");$("home").classList.remove("hidden");
};
</script>
</body>
</html>
Percepteur

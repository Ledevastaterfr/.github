<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>SIGP-GN Terminal</title>
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f2f6fc;
      height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .login-container {
      background: white;
      padding: 40px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      width: 350px;
      text-align: center;
    }
    input, select {
      width: 100%;
      padding: 10px;
      margin: 10px 0;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      background: #0077b6;
      color: white;
      border: none;
      width: 100%;
      padding: 10px;
      margin-top: 10px;
      cursor: pointer;
      border-radius: 5px;
    }
    button:hover {
      background: #005f8a;
    }
    .link {
      margin-top: 10px;
    }
    .link a {
      color: #0077b6;
      text-decoration: none;
      font-size: 14px;
    }
    .hidden {
      display: none;
    }
    .error, .success {
      margin-top: 10px;
      font-size: 14px;
    }
    .error {
      color: red;
    }
    .success {
      color: green;
    }
  </style>
</head>

<body>

<!-- Authentification : Connexion / Création / Mot de passe oublié -->
<div id="auth-container" class="login-container">
  <h2>Connexion SIGP-GN</h2>
  <input type="text" id="login-username" placeholder="Identifiant">
  <input type="password" id="login-password" placeholder="Mot de passe">
  <select id="login-service">
    <option value="">-- Sélectionner votre service --</option>
    <option>Police Nationale</option>
    <option>Gendarmerie Nationale</option>
  </select>
  <button onclick="login()">Se connecter</button>
  <div class="link">
    <a href="#" onclick="showCreate()">Créer un compte</a> | 
    <a href="#" onclick="showForgot()">Mot de passe oublié ?</a>
  </div>
  <div id="login-error" class="error hidden">Identifiant ou mot de passe incorrect.</div>
</div>

<div id="create-container" class="login-container hidden">
  <h2>Créer un compte</h2>
  <input type="text" id="create-username" placeholder="Identifiant">
  <input type="password" id="create-password" placeholder="Mot de passe">
  <select id="create-service">
    <option value="">-- Sélectionner votre service --</option>
    <option>Police Nationale</option>
    <option>Gendarmerie Nationale</option>
  </select>
  <button onclick="createAccount()">Créer</button>
  <div class="link">
    <a href="#" onclick="showLogin()">Retour connexion</a>
  </div>
  <div id="create-success" class="success hidden">Compte créé avec succès !</div>
</div>

<div id="forgot-container" class="login-container hidden">
  <h2>Mot de passe oublié</h2>
  <input type="text" id="forgot-username" placeholder="Votre identifiant">
  <button onclick="forgotPassword()">Envoyer</button>
  <div class="link">
    <a href="#" onclick="showLogin()">Retour connexion</a>
  </div>
  <div id="forgot-success" class="success hidden">Un email fictif a été envoyé !</div>
</div>
<script>
// Afficher la création de compte
function showCreate() {
  document.getElementById('auth-container').classList.add('hidden');
  document.getElementById('create-container').classList.remove('hidden');
  document.getElementById('forgot-container').classList.add('hidden');
}

// Afficher le mot de passe oublié
function showForgot() {
  document.getElementById('auth-container').classList.add('hidden');
  document.getElementById('create-container').classList.add('hidden');
  document.getElementById('forgot-container').classList.remove('hidden');
}

// Retour à la connexion
function showLogin() {
  document.getElementById('auth-container').classList.remove('hidden');
  document.getElementById('create-container').classList.add('hidden');
  document.getElementById('forgot-container').classList.add('hidden');
}

// Créer un compte
function createAccount() {
  const username = document.getElementById('create-username').value.trim();
  const password = document.getElementById('create-password').value.trim();
  const service = document.getElementById('create-service').value;
  if (username && password && service) {
    let users = JSON.parse(localStorage.getItem('users') || '[]');
    users.push({ username, password, service });
    localStorage.setItem('users', JSON.stringify(users));
    document.getElementById('create-success').classList.remove('hidden');
    setTimeout(() => showLogin(), 1500);
  }
}

// Connexion
function login() {
  const username = document.getElementById('login-username').value.trim();
  const password = document.getElementById('login-password').value.trim();
  const service = document.getElementById('login-service').value;
  const users = JSON.parse(localStorage.getItem('users') || '[]');
  const user = users.find(u => u.username === username && u.password === password && u.service === service);

  if (user) {
    localStorage.setItem('loggedUser', JSON.stringify(user));
    loadDashboard();
  } else {
    document.getElementById('login-error').classList.remove('hidden');
  }
}

// Mot de passe oublié
function forgotPassword() {
  document.getElementById('forgot-success').classList.remove('hidden');
}

// Déconnexion
function logout() {
  localStorage.removeItem('loggedUser');
  location.reload();
}

// Charger automatiquement si connecté
document.addEventListener('DOMContentLoaded', () => {
  const user = JSON.parse(localStorage.getItem('loggedUser'));
  if (user) {
    loadDashboard();
  }
});

// Chargement du dashboard
function loadDashboard() {
  document.body.innerHTML = `
    <header style="background: #0077b6; color: white; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center;">
      <div style="display: flex; align-items: center;">
        <img src="https://upload.wikimedia.org/wikipedia/commons/0/04/Logo_Gendarmerie_Nationale.svg" style="height:40px; margin-right:10px;">
        <span>SIGP-GN Terminal v1.0 — <span id="user-service"></span></span>
      </div>
      <div id="clock"></div>
    </header>

    <div style="display: flex; height: calc(100vh - 60px);">
      <nav style="width: 250px; background: #ffffff; padding: 20px; border-right: 1px solid #ccc; overflow-y: auto;">
        <button onclick="openModule('accueil')">🏠 Accueil</button>
        <button onclick="openModule('fpr')">🛡️ FPR</button>
        <button onclick="openModule('fnaeg')">🧬 FNAEG</button>
        <button onclick="openModule('faed')">✋ FAED</button>
        <button onclick="openModule('foves')">🚗 FOVeS</button>
        <button onclick="openModule('fsprt')">🔍 FSPRT</button>
        <button onclick="openModule('urgences')">📞 Urgences</button>
        <button onclick="openModule('radio')">📻 Radio</button>
        <button onclick="openModule('interventions')">🚨 Interventions</button>
        <button onclick="openModule('annuaire')">📒 Annuaire</button>
        <button onclick="openModule('bases')">📚 Bases Nationales</button>
        <button onclick="logout()" style="margin-top:30px; background:#ff6b6b;">❌ Déconnexion</button>
      </nav>

      <main id="main-content" style="flex-grow: 1; padding: 30px; background: #f2f6fc; overflow-y: auto;"></main>
    </div>
  `;

  document.getElementById('user-service').innerText = JSON.parse(localStorage.getItem('loggedUser')).service;
  setInterval(updateClock, 1000);
  updateClock();
  openModule('accueil');
}

// Mise à jour de l'horloge
function updateClock() {
  const clock = document.getElementById('clock');
  if (clock) clock.innerText = new Date().toLocaleTimeString('fr-FR');
}

// Gestion de la navigation
function openModule(name) {
  const main = document.getElementById('main-content');
  main.innerHTML = `<h1>Chargement...</h1>`;
  setTimeout(() => {
    if (name === 'accueil') {
      main.innerHTML = `<h1>Bienvenue sur SIGP-GN Terminal</h1><p>Sélectionnez un module à gauche pour commencer.</p>`;
    }
    if (name === 'fpr') loadFPR();
    if (name === 'fnaeg') loadFNAEG();
    if (name === 'faed') loadFAED();
    if (name === 'foves') loadFOVES();
    if (name === 'fsprt') loadFSPRT();
    if (name === 'urgences') loadUrgences();
    if (name === 'radio') loadRadio();
    if (name === 'interventions') loadInterventions();
    if (name === 'annuaire') loadAnnuaire();
    if (name === 'bases') loadBases();
  }, 300);
}
</script>
<script>
// Afficher la création de compte
function showCreate() {
  document.getElementById('auth-container').classList.add('hidden');
  document.getElementById('create-container').classList.remove('hidden');
  document.getElementById('forgot-container').classList.add('hidden');
}

// Afficher le mot de passe oublié
function showForgot() {
  document.getElementById('auth-container').classList.add('hidden');
  document.getElementById('create-container').classList.add('hidden');
  document.getElementById('forgot-container').classList.remove('hidden');
}

// Retour à la connexion
function showLogin() {
  document.getElementById('auth-container').classList.remove('hidden');
  document.getElementById('create-container').classList.add('hidden');
  document.getElementById('forgot-container').classList.add('hidden');
}

// Créer un compte
function createAccount() {
  const username = document.getElementById('create-username').value.trim();
  const password = document.getElementById('create-password').value.trim();
  const service = document.getElementById('create-service').value;
  if (username && password && service) {
    let users = JSON.parse(localStorage.getItem('users') || '[]');
    users.push({ username, password, service });
    localStorage.setItem('users', JSON.stringify(users));
    document.getElementById('create-success').classList.remove('hidden');
    setTimeout(() => showLogin(), 1500);
  }
}

// Connexion
function login() {
  const username = document.getElementById('login-username').value.trim();
  const password = document.getElementById('login-password').value.trim();
  const service = document.getElementById('login-service').value;
  const users = JSON.parse(localStorage.getItem('users') || '[]');
  const user = users.find(u => u.username === username && u.password === password && u.service === service);

  if (user) {
    localStorage.setItem('loggedUser', JSON.stringify(user));
    loadDashboard();
  } else {
    document.getElementById('login-error').classList.remove('hidden');
  }
}

// Mot de passe oublié
function forgotPassword() {
  document.getElementById('forgot-success').classList.remove('hidden');
}

// Déconnexion
function logout() {
  localStorage.removeItem('loggedUser');
  location.reload();
}

// Charger automatiquement si connecté
document.addEventListener('DOMContentLoaded', () => {
  const user = JSON.parse(localStorage.getItem('loggedUser'));
  if (user) {
    loadDashboard();
  }
});

// Chargement du dashboard
function loadDashboard() {
  document.body.innerHTML = `
    <header style="background: #0077b6; color: white; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center;">
      <div style="display: flex; align-items: center;">
        <img src="https://upload.wikimedia.org/wikipedia/commons/0/04/Logo_Gendarmerie_Nationale.svg" style="height:40px; margin-right:10px;">
        <span>SIGP-GN Terminal v1.0 — <span id="user-service"></span></span>
      </div>
      <div id="clock"></div>
    </header>

    <div style="display: flex; height: calc(100vh - 60px);">
      <nav style="width: 250px; background: #ffffff; padding: 20px; border-right: 1px solid #ccc; overflow-y: auto;">
        <button onclick="openModule('accueil')">🏠 Accueil</button>
        <button onclick="openModule('fpr')">🛡️ FPR</button>
        <button onclick="openModule('fnaeg')">🧬 FNAEG</button>
        <button onclick="openModule('faed')">✋ FAED</button>
        <button onclick="openModule('foves')">🚗 FOVeS</button>
        <button onclick="openModule('fsprt')">🔍 FSPRT</button>
        <button onclick="openModule('urgences')">📞 Urgences</button>
        <button onclick="openModule('radio')">📻 Radio</button>
        <button onclick="openModule('interventions')">🚨 Interventions</button>
        <button onclick="openModule('annuaire')">📒 Annuaire</button>
        <button onclick="openModule('bases')">📚 Bases Nationales</button>
        <button onclick="logout()" style="margin-top:30px; background:#ff6b6b;">❌ Déconnexion</button>
      </nav>

      <main id="main-content" style="flex-grow: 1; padding: 30px; background: #f2f6fc; overflow-y: auto;"></main>
    </div>
  `;

  document.getElementById('user-service').innerText = JSON.parse(localStorage.getItem('loggedUser')).service;
  setInterval(updateClock, 1000);
  updateClock();
  openModule('accueil');
}

// Mise à jour de l'horloge
function updateClock() {
  const clock = document.getElementById('clock');
  if (clock) clock.innerText = new Date().toLocaleTimeString('fr-FR');
}

// Gestion de la navigation
function openModule(name) {
  const main = document.getElementById('main-content');
  main.innerHTML = `<h1>Chargement...</h1>`;
  setTimeout(() => {
    if (name === 'accueil') {
      main.innerHTML = `<h1>Bienvenue sur SIGP-GN Terminal</h1><p>Sélectionnez un module à gauche pour commencer.</p>`;
    }
    if (name === 'fpr') loadFPR();
    if (name === 'fnaeg') loadFNAEG();
    if (name === 'faed') loadFAED();
    if (name === 'foves') loadFOVES();
    if (name === 'fsprt') loadFSPRT();
    if (name === 'urgences') loadUrgences();
    if (name === 'radio') loadRadio();
    if (name === 'interventions') loadInterventions();
    if (name === 'annuaire') loadAnnuaire();
    if (name === 'bases') loadBases();
  }, 300);
}
</script>
<script>
// Module FPR - Fichier des Personnes Recherchées
function loadFPR() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Fichier FPR 🛡️</h2>
    <input type="text" id="fpr-nom" placeholder="Nom">
    <input type="text" id="fpr-prenom" placeholder="Prénom">
    <input type="text" id="fpr-motif" placeholder="Motif de recherche">
    <button onclick="addFPR()">Ajouter</button>
    <table id="fpr-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Nom</th><th>Prénom</th><th>Motif</th><th>Action</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderFPR();
}

function addFPR() {
  const list = JSON.parse(localStorage.getItem('fprList') || '[]');
  list.push({
    nom: document.getElementById('fpr-nom').value,
    prenom: document.getElementById('fpr-prenom').value,
    motif: document.getElementById('fpr-motif').value
  });
  localStorage.setItem('fprList', JSON.stringify(list));
  renderFPR();
}

function renderFPR() {
  const list = JSON.parse(localStorage.getItem('fprList') || '[]');
  const tbody = document.querySelector('#fpr-table tbody');
  tbody.innerHTML = '';
  list.forEach((item, index) => {
    tbody.innerHTML += `
      <tr>
        <td>${item.nom}</td>
        <td>${item.prenom}</td>
        <td>${item.motif}</td>
        <td><button onclick="deleteFPR(${index})">Supprimer</button></td>
      </tr>
    `;
  });
}

function deleteFPR(index) {
  const list = JSON.parse(localStorage.getItem('fprList') || '[]');
  list.splice(index, 1);
  localStorage.setItem('fprList', JSON.stringify(list));
  renderFPR();
}

// Module FNAEG - ADN
function loadFNAEG() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Fichier FNAEG 🧬</h2>
    <input type="text" id="fnaeg-nom" placeholder="Nom">
    <input type="text" id="fnaeg-prenom" placeholder="Prénom">
    <input type="text" id="fnaeg-profil" placeholder="Profil ADN">
    <button onclick="addFNAEG()">Ajouter</button>
    <table id="fnaeg-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Nom</th><th>Prénom</th><th>Profil ADN</th><th>Action</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderFNAEG();
}

function addFNAEG() {
  const list = JSON.parse(localStorage.getItem('fnaegList') || '[]');
  list.push({
    nom: document.getElementById('fnaeg-nom').value,
    prenom: document.getElementById('fnaeg-prenom').value,
    profil: document.getElementById('fnaeg-profil').value
  });
  localStorage.setItem('fnaegList', JSON.stringify(list));
  renderFNAEG();
}

function renderFNAEG() {
  const list = JSON.parse(localStorage.getItem('fnaegList') || '[]');
  const tbody = document.querySelector('#fnaeg-table tbody');
  tbody.innerHTML = '';
  list.forEach((item, index) => {
    tbody.innerHTML += `
      <tr>
        <td>${item.nom}</td>
        <td>${item.prenom}</td>
        <td>${item.profil}</td>
        <td><button onclick="deleteFNAEG(${index})">Supprimer</button></td>
      </tr>
    `;
  });
}

function deleteFNAEG(index) {
  const list = JSON.parse(localStorage.getItem('fnaegList') || '[]');
  list.splice(index, 1);
  localStorage.setItem('fnaegList', JSON.stringify(list));
  renderFNAEG();
}

// Module FAED - Empreintes digitales
function loadFAED() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Fichier FAED ✋</h2>
    <input type="text" id="faed-nom" placeholder="Nom">
    <input type="text" id="faed-prenom" placeholder="Prénom">
    <input type="text" id="faed-reference" placeholder="Référence Empreinte">
    <button onclick="addFAED()">Ajouter</button>
    <table id="faed-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Nom</th><th>Prénom</th><th>Référence</th><th>Action</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderFAED();
}

function addFAED() {
  const list = JSON.parse(localStorage.getItem('faedList') || '[]');
  list.push({
    nom: document.getElementById('faed-nom').value,
    prenom: document.getElementById('faed-prenom').value,
    reference: document.getElementById('faed-reference').value
  });
  localStorage.setItem('faedList', JSON.stringify(list));
  renderFAED();
}

function renderFAED() {
  const list = JSON.parse(localStorage.getItem('faedList') || '[]');
  const tbody = document.querySelector('#faed-table tbody');
  tbody.innerHTML = '';
  list.forEach((item, index) => {
    tbody.innerHTML += `
      <tr>
        <td>${item.nom}</td>
        <td>${item.prenom}</td>
        <td>${item.reference}</td>
        <td><button onclick="deleteFAED(${index})">Supprimer</button></td>
      </tr>
    `;
  });
}

function deleteFAED(index) {
  const list = JSON.parse(localStorage.getItem('faedList') || '[]');
  list.splice(index, 1);
  localStorage.setItem('faedList', JSON.stringify(list));
  renderFAED();
}

// Module FOVES - Objets/Véhicules signalés
function loadFOVES() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Fichier FOVES 🚗</h2>
    <input type="text" id="foves-objet" placeholder="Objet/Véhicule">
    <input type="text" id="foves-detail" placeholder="Détail Signalement">
    <button onclick="addFOVES()">Ajouter</button>
    <table id="foves-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Objet/Véhicule</th><th>Détail</th><th>Action</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderFOVES();
}

function addFOVES() {
  const list = JSON.parse(localStorage.getItem('fovesList') || '[]');
  list.push({
    objet: document.getElementById('foves-objet').value,
    detail: document.getElementById('foves-detail').value
  });
  localStorage.setItem('fovesList', JSON.stringify(list));
  renderFOVES();
}

function renderFOVES() {
  const list = JSON.parse(localStorage.getItem('fovesList') || '[]');
  const tbody = document.querySelector('#foves-table tbody');
  tbody.innerHTML = '';
  list.forEach((item, index) => {
    tbody.innerHTML += `
      <tr>
        <td>${item.objet}</td>
        <td>${item.detail}</td>
        <td><button onclick="deleteFOVES(${index})">Supprimer</button></td>
      </tr>
    `;
  });
}

function deleteFOVES(index) {
  const list = JSON.parse(localStorage.getItem('fovesList') || '[]');
  list.splice(index, 1);
  localStorage.setItem('fovesList', JSON.stringify(list));
  renderFOVES();
}

// Module FSPRT - Radicalisation
function loadFSPRT() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Fichier FSPRT 🔍</h2>
    <input type="text" id="fsprt-nom" placeholder="Nom">
    <input type="text" id="fsprt-prenom" placeholder="Prénom">
    <input type="text" id="fsprt-niveau" placeholder="Niveau de Risque">
    <button onclick="addFSPRT()">Ajouter</button>
    <table id="fsprt-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Nom</th><th>Prénom</th><th>Niveau</th><th>Action</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderFSPRT();
}

function addFSPRT() {
  const list = JSON.parse(localStorage.getItem('fsprtList') || '[]');
  list.push({
    nom: document.getElementById('fsprt-nom').value,
    prenom: document.getElementById('fsprt-prenom').value,
    niveau: document.getElementById('fsprt-niveau').value
  });
  localStorage.setItem('fsprtList', JSON.stringify(list));
  renderFSPRT();
}

function renderFSPRT() {
  const list = JSON.parse(localStorage.getItem('fsprtList') || '[]');
  const tbody = document.querySelector('#fsprt-table tbody');
  tbody.innerHTML = '';
  list.forEach((item, index) => {
    tbody.innerHTML += `
      <tr>
        <td>${item.nom}</td>
        <td>${item.prenom}</td>
        <td>${item.niveau}</td>
        <td><button onclick="deleteFSPRT(${index})">Supprimer</button></td>
      </tr>
    `;
  });
}

function deleteFSPRT(index) {
  const list = JSON.parse(localStorage.getItem('fsprtList') || '[]');
  list.splice(index, 1);
  localStorage.setItem('fsprtList', JSON.stringify(list));
  renderFSPRT();
}
</script>
<script>
// Module URGENCES 📞
function loadUrgences() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Appels d'Urgence 📞</h2>
    <input type="text" id="urgence-nom" placeholder="Nom Appelant">
    <input type="text" id="urgence-incident" placeholder="Description Incident">
    <input type="text" id="urgence-localisation" placeholder="Lien Google Maps">
    <button onclick="addUrgence()">Ajouter Appel</button>
    <table id="urgence-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Nom</th><th>Incident</th><th>Localisation</th><th>Action</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderUrgences();
}

function addUrgence() {
  const urgences = JSON.parse(localStorage.getItem('urgenceList') || '[]');
  urgences.push({
    nom: document.getElementById('urgence-nom').value,
    incident: document.getElementById('urgence-incident').value,
    localisation: document.getElementById('urgence-localisation').value
  });
  localStorage.setItem('urgenceList', JSON.stringify(urgences));
  renderUrgences();
}

function renderUrgences() {
  const urgences = JSON.parse(localStorage.getItem('urgenceList') || '[]');
  const tbody = document.querySelector('#urgence-table tbody');
  tbody.innerHTML = '';
  urgences.forEach((u, index) => {
    tbody.innerHTML += `
      <tr>
        <td>${u.nom}</td>
        <td>${u.incident}</td>
        <td><iframe src="${u.localisation}" width="200" height="100" style="border:0;"></iframe></td>
        <td><button onclick="createIntervention(${index})">Créer Intervention</button></td>
      </tr>
    `;
  });
}

// Créer une intervention à partir d'un appel
function createIntervention(index) {
  const urgences = JSON.parse(localStorage.getItem('urgenceList') || '[]');
  const interventions = JSON.parse(localStorage.getItem('interventionList') || '[]');
  const appel = urgences[index];
  interventions.push({
    nom: appel.nom,
    incident: appel.incident,
    statut: "En attente",
    heure: new Date().toLocaleString()
  });
  localStorage.setItem('interventionList', JSON.stringify(interventions));
  alert("Intervention créée !");
}

// Module RADIO 📻
function loadRadio() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Radio 📻</h2>
    <textarea id="radio-message" placeholder="Votre message radio..." style="width:100%; height:100px;"></textarea><br>
    <button onclick="addRadio()">Envoyer Message</button>
    <div id="radio-history" style="margin-top:20px;"></div>
  `;
  renderRadio();
}

function addRadio() {
  const radios = JSON.parse(localStorage.getItem('radioList') || '[]');
  radios.push({
    message: document.getElementById('radio-message').value,
    date: new Date().toLocaleString()
  });
  localStorage.setItem('radioList', JSON.stringify(radios));
  renderRadio();
}

function renderRadio() {
  const radios = JSON.parse(localStorage.getItem('radioList') || '[]');
  const div = document.getElementById('radio-history');
  div.innerHTML = radios.map(r => `<p><b>${r.date}:</b> ${r.message}</p>`).join('');
}

// Module INTERVENTIONS 🚨
function loadInterventions() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Suivi des Interventions 🚨</h2>
    <table id="intervention-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Nom</th><th>Incident</th><th>Heure</th><th>Statut</th><th>Action</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderInterventions();
}

function renderInterventions() {
  const interventions = JSON.parse(localStorage.getItem('interventionList') || '[]');
  const tbody = document.querySelector('#intervention-table tbody');
  tbody.innerHTML = '';
  interventions.forEach((i, index) => {
    tbody.innerHTML += `
      <tr>
        <td>${i.nom}</td>
        <td>${i.incident}</td>
        <td>${i.heure}</td>
        <td>${i.statut}</td>
        <td><button onclick="nextStatus(${index})">Changer Statut</button></td>
      </tr>
    `;
  });
}

function nextStatus(index) {
  const interventions = JSON.parse(localStorage.getItem('interventionList') || '[]');
  if (interventions[index].statut === "En attente") interventions[index].statut = "En cours";
  else if (interventions[index].statut === "En cours") interventions[index].statut = "Terminé";
  localStorage.setItem('interventionList', JSON.stringify(interventions));
  renderInterventions();
}

// Module ANNUAIRE 📒
function loadAnnuaire() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Annuaire des Agents 📒</h2>
    <input type="text" id="search-agent" placeholder="Rechercher un agent..." oninput="renderAnnuaire()">
    <table id="annuaire-table" style="margin-top:20px; width:100%;">
      <thead><tr><th>Nom</th><th>Prénom</th><th>Service</th></tr></thead>
      <tbody></tbody>
    </table>
  `;
  renderAnnuaire();
}

function renderAnnuaire() {
  const agents = JSON.parse(localStorage.getItem('agentList') || '[]');
  const search = document.getElementById('search-agent').value.toLowerCase();
  const tbody = document.querySelector('#annuaire-table tbody');
  tbody.innerHTML = '';
  agents
    .filter(a => a.nom.toLowerCase().includes(search) || a.prenom.toLowerCase().includes(search) || a.service.toLowerCase().includes(search))
    .forEach(agent => {
      tbody.innerHTML += `
        <tr>
          <td>${agent.nom}</td>
          <td>${agent.prenom}</td>
          <td>${agent.service}</td>
        </tr>
      `;
    });
}

// Ajouter faux agents si vide
(function initFakeAgents() {
  if (!localStorage.getItem('agentList')) {
    localStorage.setItem('agentList', JSON.stringify([
      { nom: "DUPONT", prenom: "Jean", service: "Police Nationale" },
      { nom: "MARTIN", prenom: "Claire", service: "Gendarmerie Nationale" }
    ]));
  }
})();
</script>
<script>
// Module BASES NATIONALES 📚
function loadBases() {
  const main = document.getElementById('main-content');
  main.innerHTML = `
    <h2>Bases Nationales 📚</h2>
    <table style="width:100%; margin-top:20px;">
      <thead><tr><th>Fichier</th><th>Description</th><th>Accéder</th></tr></thead>
      <tbody>
        <tr><td>TAJ</td><td>Antécédents judiciaires</td><td><button onclick="alert('Accès fictif à TAJ')">Accéder</button></td></tr>
        <tr><td>FNAEG</td><td>Empreintes ADN</td><td><button onclick="openModule('fnaeg')">Accéder</button></td></tr>
        <tr><td>FAED</td><td>Empreintes digitales</td><td><button onclick="openModule('faed')">Accéder</button></td></tr>
        <tr><td>FOVES</td><td>Objets / Véhicules signalés</td><td><button onclick="openModule('foves')">Accéder</button></td></tr>
        <tr><td>FPR</td><td>Personnes recherchées</td><td><button onclick="openModule('fpr')">Accéder</button></td></tr>
        <tr><td>FSPRT</td><td>Radicalisation</td><td><button onclick="openModule('fsprt')">Accéder</button></td></tr>
        <tr><td>FICOVIE</td><td>Assurance-vie</td><td><button onclick="alert('Accès fictif à FICOVIE')">Accéder</button></td></tr>
        <tr><td>RPF</td><td>Personnes fichées renseignement</td><td><button onclick="alert('Accès fictif à RPF')">Accéder</button></td></tr>
        <tr><td>EASP</td><td>Informations judiciaires automatisées</td><td><button onclick="alert('Accès fictif à EASP')">Accéder</button></td></tr>
        <tr><td>SIV</td><td>Immatriculations véhicules</td><td><button onclick="alert('Accès fictif à SIV')">Accéder</button></td></tr>
        <tr><td>VISABIO</td><td>Données biométriques visas</td><td><button onclick="alert('Accès fictif à VISABIO')">Accéder</button></td></tr>
        <tr><td>AGDREF</td><td>Suivi ressortissants étrangers</td><td><button onclick="alert('Accès fictif à AGDREF')">Accéder</button></td></tr>
        <tr><td>TES</td><td>Passeports et CNI biométriques</td><td><button onclick="alert('Accès fictif à TES')">Accéder</button></td></tr>
      </tbody>
    </table>
  `;
}
</script>


<!DOCTYPE html>
<html lang="es">

<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, initial-scale=1.0">

<title>Panel SaaS</title>

<style>

body{
  margin:0;
  font-family:Arial;
  background:#0b1220;
  color:white;
}

/* LOGIN */

#login{
  height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
}

.box{
  background:#111a2e;
  padding:30px;
  border-radius:12px;
  width:320px;
}

input,select{
  width:100%;
  padding:10px;
  margin-top:10px;
  border:none;
  border-radius:6px;
  box-sizing:border-box;
}

button{
  width:100%;
  padding:10px;
  margin-top:10px;
  border:none;
  border-radius:6px;
  background:#3b82f6;
  color:white;
  cursor:pointer;
}

button:hover{
  background:#2563eb;
}

/* DASH */

#dash{
  display:none;
  padding:20px;
}

.panel{
  display:grid;
  grid-template-columns:1fr 2fr;
  gap:20px;
}

.section{
  background:#111a2e;
  padding:15px;
  border-radius:10px;
}

.item{
  background:#0f172a;
  padding:10px;
  border-radius:6px;
  margin-top:10px;
}

.small{
  opacity:.7;
  font-size:12px;
}

.actions{
  display:flex;
  gap:10px;
  margin-top:10px;
}

.actions button{
  flex:1;
}

/* STATS */

.stats-grid{
  display:grid;
  grid-template-columns:
  repeat(auto-fit,minmax(160px,1fr));
  gap:15px;
  margin-bottom:20px;
}

.stat-card{
  background:#111a2e;
  padding:20px;
  border-radius:12px;
  text-align:center;
}

.stat-card h3{
  margin:0;
  font-size:28px;
  color:#3b82f6;
}

.stat-card p{
  margin-top:10px;
  opacity:.8;
}

</style>
</head>

<body>

<!-- LOGIN -->

<div id="login">

  <div class="box">

    <h2>Admin Login</h2>

    <input
      id="email"
      placeholder="Email">

    <input
      id="password"
      type="password"
      placeholder="Contraseña">

    <button onclick="login()">
      Entrar
    </button>

    <p id="error"></p>

  </div>

</div>

<!-- DASH -->

<div id="dash">

  <h2>Panel SaaS</h2>

  <h3 id="welcomeAdmin"></h3>

  <!-- STATS -->

  <div class="stats-grid">

    <div class="stat-card">
      <h3 id="totalKeys">0</h3>
      <p>🔑 Keys Totales</p>
    </div>

    <div class="stat-card">
      <h3 id="activeKeys">0</h3>
      <p>✅ Keys Activas</p>
    </div>

    <div class="stat-card">
      <h3 id="expiredKeys">0</h3>
      <p>❌ Expiradas</p>
    </div>

    <div class="stat-card">
      <h3 id="usedKeys">0</h3>
      <p>🔒 Keys Usadas</p>
    </div>

    <div class="stat-card">
      <h3 id="unusedKeys">0</h3>
      <p>🟢 Sin usar</p>
    </div>

    <div class="stat-card">
      <h3 id="onlineUsersStat">0</h3>
      <p>🟢 Online</p>
    </div>

    <div class="stat-card">
      <h3 id="bannedCount">0</h3>
      <p>🚫 Baneados</p>
    </div>

  </div>

  <h3 id="onlineCount">
    Usuarios online: 0
  </h3>

  <button onclick="logout()">
    Cerrar sesión
  </button>

  <button onclick="deleteExpired()">
    Borrar expiradas
  </button>

  <br><br>

  <div class="panel">

    <!-- CREAR -->

    <div class="section">

      <h3>Crear Key</h3>

      <select id="duration">

        <option value="1">1 día</option>
        <option value="7">7 días</option>
        <option value="14">2 semanas</option>
        <option value="30">1 mes</option>
        <option value="365">1 año</option>

      </select>

      <button onclick="createKey()">
        Generar Key
      </button>

      <p id="newKey"></p>

    </div>

    <!-- KEYS -->

    <div class="section">

      <h3>Mis Keys</h3>

      <div id="list"></div>

    </div>

  </div>

  <br>

  <!-- BANS -->

  <div class="section">

    <h3>Banear dispositivo</h3>

    <input
      id="banDevice"
      placeholder="device-id">

    <button onclick="banDevice()">
      Banear
    </button>

  </div>

  <br>

  <!-- LOGS -->

  <div class="section">

    <h3>Logs Usuarios</h3>

    <div id="logs"></div>

  </div>

</div>

<script type="module">

/* FIREBASE */

import { initializeApp } from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {

  getAuth,
  signInWithEmailAndPassword,
  onAuthStateChanged,
  signOut

} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

import {

  getDatabase,
  ref,
  set,
  get,
  child,
  remove,
  onValue

} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* CONFIG */

const firebaseConfig = {

  apiKey:
  "TU_API_KEY",

  authDomain:
  "TU_AUTH_DOMAIN",

  databaseURL:
  "TU_DATABASE_URL",

  projectId:
  "TU_PROJECT_ID",

  storageBucket:
  "TU_STORAGE",

  messagingSenderId:
  "TU_SENDER_ID",

  appId:
  "TU_APP_ID"

};

/* INIT */

const app =
initializeApp(firebaseConfig);

const auth =
getAuth(app);

const db =
getDatabase(app);

let currentUID = "";

/* LOGIN */

window.login =
async function(){

  const email =
    document.getElementById(
      "email"
    ).value;

  const password =
    document.getElementById(
      "password"
    ).value;

  try{

    await signInWithEmailAndPassword(
      auth,
      email,
      password
    );

  }catch(err){

    document.getElementById(
      "error"
    ).innerHTML =
      err.message;

  }

};

/* SESSION */

onAuthStateChanged(auth,
(user) => {

  const loginDiv =
    document.getElementById(
      "login"
    );

  const dashDiv =
    document.getElementById(
      "dash"
    );

  if(user){

    currentUID = user.uid;

    const emailName =
      user.email
      .split("@")[0];

    document.getElementById(
      "welcomeAdmin"
    ).innerHTML =
      "Bienvenido, " +
      emailName;

    loginDiv.style.display =
      "none";

    dashDiv.style.display =
      "block";

    loadKeys();

    loadLogs();

    loadStats();

  }else{

    loginDiv.style.display =
      "flex";

    dashDiv.style.display =
      "none";

  }

});

/* LOGOUT */

window.logout =
async function(){

  await signOut(auth);

};

/* GEN KEY */

function genKey(){

  const chars =
  "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

  let key = "";

  for(let i=0;i<16;i++){

    key += chars[
      Math.floor(
        Math.random() *
        chars.length
      )
    ];

    if(
      i % 4 === 3 &&
      i < 15
    ){

      key += "-";

    }

  }

  return key;

}

/* CREATE KEY */

window.createKey =
async function(){

  const days =
    parseInt(
      document.getElementById(
        "duration"
      ).value
    );

  const now =
    Date.now();

  const expiresAt =
    now +
    (
      days *
      24 *
      60 *
      60 *
      1000
    );

  const newKey =
    genKey();

  const keyData = {

    key:newKey,

    days:days,

    createdAt:now,

    expiresAt:expiresAt,

    used:false,

    usedBy:"",

    active:true

  };

  await set(

    ref(

      db,

      "users/" +

      currentUID +

      "/keys/" +

      newKey

    ),

    keyData

  );

  document.getElementById(
    "newKey"
  ).innerHTML =
    "<b>" +
    newKey +
    "</b>";

  loadKeys();

  loadStats();

};

/* LOAD KEYS */

async function loadKeys(){

  const list =
    document.getElementById(
      "list"
    );

  list.innerHTML = "";

  const snapshot =
    await get(

      child(

        ref(db),

        "users/" +

        currentUID +

        "/keys"

      )

    );

  if(snapshot.exists()){

    const data =
      snapshot.val();

    Object.values(data)
    .reverse()
    .forEach(k => {

      const div =
        document.createElement(
          "div"
        );

      div.className =
        "item";

      const exp =
        new Date(
          k.expiresAt
        );

      div.innerHTML = `

        <b>${k.key}</b>

        <div class="small">
          ${k.days} días
        </div>

        <div class="small">
          Expira:
          ${exp.toLocaleString()}
        </div>

      `;

      list.appendChild(div);

    });

  }

}

/* STATS */

async function loadStats(){

  let total = 0;
  let active = 0;
  let expired = 0;
  let used = 0;
  let unused = 0;

  const snapshot =
    await get(

      child(

        ref(db),

        "users/" +

        currentUID +

        "/keys"

      )

    );

  if(snapshot.exists()){

    const data =
      snapshot.val();

    Object.values(data)
    .forEach(k => {

      total++;

      if(
        Date.now() >
        k.expiresAt
      ){

        expired++;

      }else{

        active++;

      }

      if(k.used){

        used++;

      }else{

        unused++;

      }

    });

  }

  document.getElementById(
    "totalKeys"
  ).innerHTML = total;

  document.getElementById(
    "activeKeys"
  ).innerHTML = active;

  document.getElementById(
    "expiredKeys"
  ).innerHTML = expired;

  document.getElementById(
    "usedKeys"
  ).innerHTML = used;

  document.getElementById(
    "unusedKeys"
  ).innerHTML = unused;

  /* ONLINE */

  const onlineSnap =
    await get(

      child(
        ref(db),
        "onlineUsers"
      )

    );

  if(onlineSnap.exists()){

    document.getElementById(
      "onlineUsersStat"
    ).innerHTML =

      Object.keys(
        onlineSnap.val()
      ).length;

  }

  /* BANNED */

  const bannedSnap =
    await get(

      child(
        ref(db),
        "bannedDevices"
      )

    );

  if(bannedSnap.exists()){

    document.getElementById(
      "bannedCount"
    ).innerHTML =

      Object.keys(
        bannedSnap.val()
      ).length;

  }

}

/* LOGS */

async function loadLogs(){

  const logsDiv =
    document.getElementById(
      "logs"
    );

  logsDiv.innerHTML = "";

  const snapshot =
    await get(

      child(
        ref(db),
        "logs"
      )

    );

  if(snapshot.exists()){

    const data =
      snapshot.val();

    Object.values(data)
    .reverse()
    .forEach(log => {

      const div =
        document.createElement(
          "div"
        );

      div.className =
        "item";

      div.innerHTML = `

        📱 ${log.device}

        <div class="small">
          🔑 ${log.key}
        </div>

      `;

      logsDiv.appendChild(div);

    });

  }

}

/* DELETE EXPIRED */

window.deleteExpired =
async function(){

  const snapshot =
    await get(

      child(

        ref(db),

        "users/" +
        currentUID +
        "/keys"

      )

    );

  if(snapshot.exists()){

    const data =
      snapshot.val();

    for(
      const k of
      Object.values(data)
    ){

      if(
        Date.now() >
        k.expiresAt
      ){

        await remove(

          ref(

            db,

            "users/" +

            currentUID +

            "/keys/" +

            k.key

          )

        );

      }

    }

  }

  loadKeys();

  loadStats();

};

/* BAN */

window.banDevice =
async function(){

  const device =
    document.getElementById(
      "banDevice"
    ).value;

  await set(

    ref(

      db,

      "bannedDevices/" +
      device

    ),

    true

  );

  alert(
    "Dispositivo baneado"
  );

};

</script>

</body>
</html>

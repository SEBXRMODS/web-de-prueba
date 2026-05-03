<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, initial-scale=1.0">

<title>Simulador App</title>

<style>

body{
  margin:0;
  font-family:Arial;
  background:#0b1220;
  color:white;
  display:flex;
  justify-content:center;
  align-items:center;
  min-height:100vh;
  padding:20px;
  box-sizing:border-box;
}

.box{
  width:380px;
  background:#111a2e;
  padding:25px;
  border-radius:12px;
}

input{
  width:100%;
  padding:12px;
  margin-top:10px;
  border:none;
  border-radius:6px;
  box-sizing:border-box;
}

button{
  width:100%;
  padding:12px;
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

#panel{
  display:none;
  margin-top:20px;
}

.card{
  background:#0f172a;
  padding:15px;
  border-radius:8px;
  margin-top:10px;
}

.small{
  font-size:12px;
  opacity:.7;
}

.feature{
  display:flex;
  justify-content:space-between;
  align-items:center;
}

.switch{
  color:#22c55e;
  font-weight:bold;
}

</style>
</head>

<body>

<div class="box">

  <h2>Simulador App</h2>

  <input
    id="keyInput"
    placeholder="Ingresa tu key">

  <button onclick="loginKey()">
    Entrar
  </button>

  <p id="status"></p>

  <div id="panel">

    <div class="card">
      ✅ Usuario autenticado
    </div>

    <div class="card">
      🎮 Menú Principal
    </div>

    <div class="card feature">
      <span>🟢 Icono Flotante</span>
      <span class="switch">ON</span>
    </div>

    <div class="card feature">
      <span>⚡ Aim Assist</span>
      <span class="switch">ON</span>
    </div>

    <div class="card feature">
      <span>🎯 Aimbot</span>
      <span class="switch">ON</span>
    </div>

    <div class="card feature">
      <span>👀 ESP Players</span>
      <span class="switch">ON</span>
    </div>

    <div class="card feature">
      <span>🔫 No Recoil</span>
      <span class="switch">ON</span>
    </div>

    <div class="card feature">
      <span>🚀 Speed Boost</span>
      <span class="switch">ON</span>
    </div>

    <div class="card feature">
      <span>🛡️ Anti Ban</span>
      <span class="switch">ON</span>
    </div>

    <div class="card">
      <div id="expireInfo"></div>
      <div id="deviceInfo"
      class="small"></div>
    </div>

  </div>

</div>

<script type="module">

/* FIREBASE */
import { initializeApp } from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {

  getDatabase,
  ref,
  get,
  child,
  update,
  set,
  onDisconnect

} from
"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* CONFIG */
const firebaseConfig = {

  apiKey:
  "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

  authDomain:
  "panelsebxrmods.firebaseapp.com",

  databaseURL:
  "https://panelsebxrmods-default-rtdb.firebaseio.com",

  projectId:
  "panelsebxrmods",

  storageBucket:
  "panelsebxrmods.firebasestorage.app",

  messagingSenderId:
  "717339227525",

  appId:
  "1:717339227525:web:e3ee653c3d2aeb1b5800ec"

};

/* INIT */
const app = initializeApp(firebaseConfig);

const db = getDatabase(app);

/* DEVICE ID */
let deviceId =
localStorage.getItem("deviceId");

if(!deviceId){

  deviceId =
    "device-" +
    Math.random()
    .toString(36)
    .substring(2,10);

  localStorage.setItem(
    "deviceId",
    deviceId
  );

}

/* LOGIN KEY */
window.loginKey = async function(){

  const key =
    document.getElementById(
      "keyInput"
    ).value.trim();

  const status =
    document.getElementById(
      "status"
    );

  const panel =
    document.getElementById(
      "panel"
    );

  status.innerHTML =
    "Verificando key...";

  try{

    /* BUSCAR USERS */
    const usersSnapshot =
      await get(
        child(ref(db), "users")
      );

    let found = false;

    let data = null;

    let ownerUID = "";

    if(usersSnapshot.exists()){

      const usersData =
        usersSnapshot.val();

      for(const uid in usersData){

        if(

          usersData[uid].keys &&

          usersData[uid].keys[key]

        ){

          data =
            usersData[uid]
            .keys[key];

          ownerUID = uid;

          found = true;

          break;

        }

      }

    }

    /* INVALIDA */
    if(!found){

      status.innerHTML =
        "❌ Key inválida";

      return;

    }

    /* BAN CHECK */
    const banned =
      await get(

        child(
          ref(db),
          "bannedDevices/" +
          deviceId
        )

      );

    if(banned.exists()){

      status.innerHTML =
        "🚫 Dispositivo baneado";

      return;

    }

    /* DESACTIVADA */
    if(!data.active){

      status.innerHTML =
        "⛔ Key desactivada";

      return;

    }

    /* EXPIRADA */
    if(Date.now() >
      data.expiresAt){

      status.innerHTML =
        "❌ Key expirada";

      return;

    }

    /* ANTI SHARE */
    if(

      data.used &&

      data.usedBy !== deviceId

    ){

      status.innerHTML =
        "🚫 Key usada en otro dispositivo";

      return;

    }

    /* ACTIVAR KEY */
    await update(

      ref(

        db,

        "users/" +

        ownerUID +

        "/keys/" +

        key

      ),

      {

        used:true,

        usedBy:deviceId

      }

    );

    /* ONLINE USERS */
    await set(

      ref(
        db,
        "onlineUsers/" +
        deviceId
      ),

      {

        key:key,

        loginAt:Date.now()

      }

    );

    onDisconnect(

      ref(
        db,
        "onlineUsers/" +
        deviceId
      )

    ).remove();

    /* LOG */
    const logId =
      Date.now();

    await set(

      ref(
        db,
        "logs/" +
        logId
      ),

      {

        device:deviceId,

        key:key,

        time:Date.now()

      }

    );

    /* UI */
    status.innerHTML =
      "✅ Acceso concedido";

    panel.style.display =
      "block";

    const exp =
      new Date(
        data.expiresAt
      );

    document.getElementById(
      "expireInfo"
    ).innerHTML =

      "⏳ Expira: " +

      exp.toLocaleString();

    document.getElementById(
      "deviceInfo"
    ).innerHTML =

      "📱 Device ID: " +

      deviceId;

  }catch(err){

    status.innerHTML =
      "❌ Error: " +
      err.message;

  }

}

</script>

</body>
</html>

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
  background:#0b1220;
  color:white;
  font-family:Arial;
  display:flex;
  justify-content:center;
  align-items:center;
  min-height:100vh;
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

.card{
  background:#0f172a;
  padding:15px;
  border-radius:8px;
  margin-top:10px;
}

#panel{
  display:none;
}

.small{
  font-size:12px;
  opacity:.7;
}

.online{
  color:#00ff88;
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

<div class="card">
🟢 Icono Flotante
</div>

<div class="card">
⚡ Aim Assist
</div>

<div class="card">
🎯 Aimbot
</div>

<div class="card">
👀 ESP Players
</div>

<div class="card">
🔫 No Recoil
</div>

<div class="card">
🚀 Speed Boost
</div>

<div class="card">
🛡️ Anti Ban
</div>

<div class="card">

<div id="expireInfo"></div>

<div id="deviceInfo"
class="small"></div>

<div id="onlineInfo"
class="small online"></div>

</div>

</div>

</div>

<script type="module">

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

apiKey:"TU_API_KEY",

authDomain:"TU_AUTH_DOMAIN",

databaseURL:"TU_DATABASE_URL",

projectId:"TU_PROJECT_ID",

storageBucket:"TU_STORAGE_BUCKET",

messagingSenderId:"TU_SENDER_ID",

appId:"TU_APP_ID"

};

/* INIT */

const app =
initializeApp(firebaseConfig);

const db =
getDatabase(app);

/* DEVICE */

let deviceId =
localStorage.getItem(
"deviceId"
);

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

/* LOGIN */

window.loginKey =
async function(){

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

if(!found){

status.innerHTML =
"❌ Key inválida";

return;

}

/* BAN */

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

/* ACTIVE */

if(!data.active){

status.innerHTML =
"⛔ Key desactivada";

return;

}

/* EXPIRE */

if(
Date.now() >
data.expiresAt
){

status.innerHTML =
"❌ Key expirada";

return;

}

/* ANTI SHARE */

if(

data.used &&

data.usedBy !== deviceId

){

await update(

ref(

db,

"users/" +

ownerUID +

"/keys/" +

key

),

{

shared:true,

sharedAttemptBy:
deviceId

}

);

status.innerHTML =
"🚫 Key compartida detectada";

return;

}

/* ACTIVATE */

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

/* ONLINE */

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
"logs/" + logId
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

"📱 Device: " +
deviceId;

document.getElementById(
"onlineInfo"
).innerHTML =

"🟢 Online conectado";

}catch(err){

console.log(err);

status.innerHTML =
"❌ " + err.message;

}

};

</script>

</body>
</html>

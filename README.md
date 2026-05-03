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
font-family:Arial;
color:white;
display:flex;
justify-content:center;
align-items:center;
height:100vh;
overflow:hidden;
}

.box{
background:#111a2e;
padding:30px;
border-radius:12px;
width:350px;
text-align:center;
box-shadow:0 0 30px rgba(0,0,0,.4);
}

input{
width:100%;
padding:12px;
margin-top:10px;
border:none;
border-radius:6px;
box-sizing:border-box;
background:#0f172a;
color:white;
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
font-weight:bold;
}

button:hover{
background:#2563eb;
}

#error{
color:#ff4d4d;
margin-top:10px;
}

#app{
display:none;
}

.menu{
margin-top:20px;
display:grid;
gap:10px;
}

.card{
background:#0f172a;
padding:15px;
border-radius:10px;
transition:.2s;
cursor:pointer;
}

.card:hover{
transform:scale(1.03);
}

.float{
position:fixed;
bottom:20px;
right:20px;
width:60px;
height:60px;
border-radius:50%;
background:#3b82f6;
display:flex;
justify-content:center;
align-items:center;
font-size:28px;
cursor:pointer;
box-shadow:0 0 20px rgba(0,0,0,.4);
z-index:999;
}

.panel{
position:fixed;
right:20px;
bottom:90px;
background:#111a2e;
padding:15px;
border-radius:12px;
width:220px;
display:none;
box-shadow:0 0 20px rgba(0,0,0,.5);
}

.toggle{
display:flex;
justify-content:space-between;
align-items:center;
margin-top:10px;
background:#0f172a;
padding:10px;
border-radius:8px;
}

.status{
margin-top:15px;
font-size:14px;
opacity:.8;
}

</style>

</head>

<body>

<div class="box">

<div id="loginBox">

<h2>🔑 Activar Key</h2>

<input
id="keyInput"
placeholder="Ingresa tu key">

<button id="activateBtn">
Activar
</button>

<p id="error"></p>

</div>

<div id="app">

<h2>
✅ App Activada
</h2>

<p id="status"></p>

<div class="menu">

<div class="card">
📦 ESP Menu
</div>

<div class="card">
🎯 Aim Assist
</div>

<div class="card">
👁️ Wallhack
</div>

<div class="card">
⚡ Speed Boost
</div>

</div>

<div class="status">
🟢 Sistema funcionando correctamente
</div>

</div>

</div>

<div
id="floatBtn"
class="float"
style="display:none;">

⚙️

</div>

<div
id="floatPanel"
class="panel">

<h3>
⚙️ Menu Mod
</h3>

<div class="toggle">
<span>ESP</span>
<input type="checkbox">
</div>

<div class="toggle">
<span>AIM</span>
<input type="checkbox">
</div>

<div class="toggle">
<span>WALL</span>
<input type="checkbox">
</div>

<div class="toggle">
<span>SPEED</span>
<input type="checkbox">
</div>

</div>

<script type="module">

import { initializeApp }

from

"https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";

import {

getDatabase,
ref,
get,
set,
child,
update,
remove

}

from

"https://www.gstatic.com/firebasejs/10.12.2/firebase-database.js";

/* FIREBASE */

const firebaseConfig = {

apiKey: "AIzaSyBs3WgavHMxywN7GMr6Lp6CSmU_NRZOSYU",

authDomain: "panelsebxrmods.firebaseapp.com",

databaseURL: "https://panelsebxrmods-default-rtdb.firebaseio.com",

projectId: "panelsebxrmods",

storageBucket: "panelsebxrmods.firebasestorage.app",

messagingSenderId: "717339227525",

appId: "1:717339227525:web:98101a11654e25a45800ec"

};

const app =
initializeApp(firebaseConfig);

const db =
getDatabase(app);

/* DEVICE ID */

let deviceId =
localStorage.getItem(
"deviceId"
);

if(!deviceId){

deviceId =
"DEV-" +
Math.random()
.toString(36)
.substring(2,10);

localStorage.setItem(
"deviceId",
deviceId
);

}

/* ACTIVATE KEY */

async function activateKey(){

const key =
document.getElementById(
"keyInput"
).value
.trim();

document.getElementById(
"error"
).innerHTML = "";

if(!key){

return;

}

try{

/* CHECK BANNED */

const bannedSnap =
await get(
child(
ref(db),
"bannedDevices/" +
deviceId
)
);

if(
bannedSnap.exists()
){

document.getElementById(
"error"
).innerHTML =
"🚫 Dispositivo baneado";

return;

}

/* GET USERS */

const usersSnap =
await get(
child(ref(db),
"users")
);

if(!usersSnap.exists()){

document.getElementById(
"error"
).innerHTML =
"❌ Key inválida";

return;

}

const usersData =
usersSnap.val();

let found = false;

/* SEARCH KEY */

for (const uid in usersData) {

const keys =
usersData[uid].keys;

if (!keys) continue;

for (const keyId in keys) {

const data =
keys[keyId];

if (data.key === key) {

found = true;

/* DESACTIVADA */

if (!data.active) {

document.getElementById(
"error"
).innerHTML =
"❌ Key desactivada";

return;

}

/* EXPIRADA */

if (
Date.now() >
data.expiresAt
) {

document.getElementById(
"error"
).innerHTML =
"⌛ Key expirada";

await remove(

ref(
db,
"users/" +
uid +
"/keys/" +
key
)

);

return;

}

/* ANTI SHARE */

if (
data.used &&
data.usedBy !== deviceId
) {

await update(

ref(
db,
"users/" +
uid +
"/keys/" +
key
),

{
shared:true
}

);

document.getElementById(
"error"
).innerHTML =
"🚫 Key compartida";

return;

}

/* ACTIVATE */

await update(

ref(
db,
"users/" +
uid +
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
time:Date.now()
}

);

/* LOG */

await set(

ref(
db,
"logs/" +
Date.now()
),

{
device:deviceId,
key:key,
time:Date.now()
}

);

/* SHOW APP */

document.getElementById(
"loginBox"
).style.display =
"none";

document.getElementById(
"app"
).style.display =
"block";

document.getElementById(
"floatBtn"
).style.display =
"flex";

const hours =
Math.floor(
(
data.expiresAt -
Date.now()
)
/3600000
);

document.getElementById(
"status"
).innerHTML =
"⏳ " +
hours +
" horas restantes";

/* SAVE */

localStorage.setItem(
"savedKey",
key
);

return;

}

}

}

/* NOT FOUND */

if(!found){

document.getElementById(
"error"
).innerHTML =
"❌ Key inválida";

}

}catch(err){

console.log(err);

document.getElementById(
"error"
).innerHTML =
err.message;

}

}

/* BUTTON */

window.onload = () => {

document
.getElementById(
"activateBtn"
)
.addEventListener(
"click",
activateKey
);

};

/* FLOAT MENU */

const floatBtn =
document.getElementById(
"floatBtn"
);

const floatPanel =
document.getElementById(
"floatPanel"
);

floatBtn.addEventListener(
"click",
()=>{

if(
floatPanel.style.display
=== "block"
){

floatPanel.style.display =
"none";

}else{

floatPanel.style.display =
"block";

}

});

</script>

</body>
</html>

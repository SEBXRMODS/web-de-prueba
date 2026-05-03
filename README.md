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
}

.box{
background:#111a2e;
padding:30px;
border-radius:12px;
width:350px;
text-align:center;
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

#error{
color:red;
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
}

</style>

</head>

<body>

<div class="box">

<div id="loginBox">

<h2>Activar Key</h2>

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

</div>

</div>

<div
id="floatBtn"
class="float"
style="display:none;">

⚙️

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

/* FIND KEY */

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

let found = false;

usersSnap.forEach(user=>{

const keys =
user.child("keys");

keys.forEach(async k=>{

const data =
k.val();

if(
data.key === key
){

found = true;

/* DESACTIVADA */

if(!data.active){

document.getElementById(
"error"
).innerHTML =

"❌ Key desactivada";

return;

}

/* EXPIRADA */

if(
Date.now() >
data.expiresAt
){

document.getElementById(
"error"
).innerHTML =

"⌛ Key expirada";

/* AUTO DELETE */

await remove(

ref(
db,
"users/" +
user.key +
"/keys/" +
key
)

);

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
user.key +
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

"🚫 Key compartida detectada";

return;

}

/* ACTIVATE */

await update(

ref(
db,
"users/" +
user.key +
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

}

});

});

setTimeout(()=>{

if(!found){

document.getElementById(
"error"
).innerHTML =

"❌ Key inválida";

}

},1000);

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

/* FLOAT BUTTON */

document
.getElementById(
"floatBtn"
)
.addEventListener(
"click",
()=>{

alert(
"⚙️ Menú flotante abierto"
);

});

</script>

</body>
</html>

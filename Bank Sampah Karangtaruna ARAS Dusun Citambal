<!-- index.html -->
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Dompet Karangtaruna ARAS - Login</title>
  <link rel="stylesheet" href="./style.css" />
</head>
<body>
  <div class="container">
    <h1>Login</h1>
    <input type="text" id="phone" placeholder="Nomor Telepon" />
    <input type="password" id="password" placeholder="Password" />
    <button id="loginBtn">Login</button>
    <p id="error" style="color: red;"></p>
  </div>
  <script type="module" src="./index.js"></script>
</body>
</html>

<!-- dashboard.html -->
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Dashboard</title>
  <link rel="stylesheet" href="./style.css" />
</head>
<body>
  <div class="container">
    <h1>Saldo Anda</h1>
    <p id="saldo"></p>
    <input type="number" id="jumlah" placeholder="Jumlah" />
    <button id="tambah">Tambah</button>
    <button id="tarik">Tarik</button>
    <button id="logout">Logout</button>
    <p id="status" style="color: green;"></p>
  </div>
  <script type="module" src="./dashboard.js"></script>
</body>
</html>

<!-- firebase-config.js -->
import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
import { getAuth } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js";
import { getFirestore } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyDVUHmKIjaHWdoAdB20cmAM06LWdFJBRW8",
  authDomain: "dompet-karangtaruna-aras.firebaseapp.com",
  projectId: "dompet-karangtaruna-aras",
  storageBucket: "dompet-karangtaruna-aras.appspot.com",
  messagingSenderId: "373945571180",
  appId: "1:373945571180:web:28823141c06db4e7d6c549",
  measurementId: "G-PWB8RX7JF0"
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const db = getFirestore(app);

<!-- index.js -->
import { auth, db } from './firebase-config.js';
import { signInWithEmailAndPassword } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js";
import { doc, getDoc } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";

document.getElementById('loginBtn').addEventListener('click', async () => {
  const phone = document.getElementById('phone').value;
  const password = document.getElementById('password').value;

  try {
    const email = `${phone}@dompet-aras.com`; // pola email dari nomor
    const userCredential = await signInWithEmailAndPassword(auth, email, password);
    localStorage.setItem("uid", userCredential.user.uid);
    window.location.href = "dashboard.html";
  } catch (error) {
    document.getElementById("error").textContent = error.message;
  }
});

<!-- dashboard.js -->
import { auth, db } from './firebase-config.js';
import { doc, getDoc, updateDoc } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";
import { signOut } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js";

const uid = localStorage.getItem("uid");
const userRef = doc(db, "users", uid);

async function updateSaldo() {
  const snap = await getDoc(userRef);
  document.getElementById("saldo").textContent = `Rp ${snap.data().saldo}`;
}

updateSaldo();

async function ubahSaldo(jumlah) {
  const snap = await getDoc(userRef);
  const saldoSekarang = snap.data().saldo;
  await updateDoc(userRef, { saldo: saldoSekarang + jumlah });
  updateSaldo();
  document.getElementById("status").textContent = jumlah > 0 ? "Saldo ditambahkan" : "Saldo ditarik";
}

document.getElementById("tambah").onclick = () => {
  const jml = parseInt(document.getElementById("jumlah").value);
  if (jml > 0) ubahSaldo(jml);
};

document.getElementById("tarik").onclick = () => {
  const jml = parseInt(document.getElementById("jumlah").value);
  if (jml > 0) ubahSaldo(-jml);
};

document.getElementById("logout").onclick = async () => {
  await signOut(auth);
  localStorage.clear();
  window.location.href = "index.html";
};

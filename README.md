<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Login Dompet Karangtaruna ARAS</title>
<link rel="stylesheet" href="style.css" />
<script src="firebase-config.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js"></script>
</head>
<body>
<div class="container">
  <h2>Login Dompet Karangtaruna ARAS</h2>
  <form id="loginForm">
    <label for="emailPhone">Email atau No HP:</label>
    <input type="text" id="emailPhone" placeholder="Email atau No HP" required />
    
    <label for="password">Password:</label>
    <input type="password" id="password" placeholder="Password" required />
    
    <button type="submit" id="btnLogin">Login</button>
  </form>
  
  <div id="loading" style="display:none;">Loading...</div>
  <div id="errorMsg" style="color:red; margin-top:10px;"></div>
  
  <button id="btnResetPassword" style="margin-top:15px;">Reset Password</button>
  <p style="margin-top:10px;">Belum punya akun? <a href="#" id="registerLink">Daftar di sini</a></p>
</div>

<script>
const auth = firebase.auth();
const db = firebase.firestore();

function isValidEmail(email) {
  // Simple email regex
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function isValidPhone(phone) {
  // Simple phone regex: angka saja, 6-15 digit
  return /^\d{6,15}$/.test(phone);
}

document.getElementById('loginForm').addEventListener('submit', async e => {
  e.preventDefault();
  const input = document.getElementById('emailPhone').value.trim();
  const password = document.getElementById('password').value.trim();
  const errorMsg = document.getElementById('errorMsg');
  const loading = document.getElementById('loading');
  errorMsg.innerText = '';

  let email = '';
  if (isValidEmail(input)) {
    email = input;
  } else if (isValidPhone(input)) {
    email = input + '@dompet-aras.com'; // dummy domain for phone login
  } else {
    errorMsg.innerText = 'Format Email atau No HP tidak valid.';
    return;
  }

  loading.style.display = 'block';
  document.getElementById('btnLogin').disabled = true;

  try {
    await auth.signInWithEmailAndPassword(email, password);
    // Setelah login, cek role user di Firestore
    const uid = auth.currentUser.uid;
    const userDoc = await db.collection('users').doc(uid).get();
    if (!userDoc.exists) {
      errorMsg.innerText = 'Data user tidak ditemukan.';
      await auth.signOut();
      return;
    }
    const userData = userDoc.data();
    if (userData.role === 'admin') {
      window.location.href = 'admin_firebase.html';
    } else {
      window.location.href = 'dashboard.html';
    }
  } catch (err) {
    errorMsg.innerText = err.message;
  } finally {
    loading.style.display = 'none';
    document.getElementById('btnLogin').disabled = false;
  }
});

// Reset password handler
document.getElementById('btnResetPassword').addEventListener('click', async () => {
  const input = document.getElementById('emailPhone').value.trim();
  const errorMsg = document.getElementById('errorMsg');
  errorMsg.innerText = '';

  let email = '';
  if (isValidEmail(input)) {
    email = input;
  } else if (isValidPhone(input)) {
    email = input + '@dompet-aras.com';
  } else {
    errorMsg.innerText = 'Masukkan Email atau No HP yang valid untuk reset password.';
    return;
  }

  try {
    await auth.sendPasswordResetEmail(email);
    alert('Email reset password telah dikirim.');
  } catch (err) {
    errorMsg.innerText = err.message;
  }
});

// Pendaftaran - link dummy (bisa dikembangkan)
document.getElementById('registerLink').addEventListener('click', (e) => {
  e.preventDefault();
  alert('Fitur pendaftaran belum tersedia.');
});
</script>
</body>
</html>
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Dashboard Dompet Karangtaruna ARAS</title>
<link rel="stylesheet" href="style.css" />
<script src="firebase-config.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js"></script>
<script src="dashboard.js" defer></script>
</head>
<body>
<div class="container">
  <h2>Dashboard Dompet Karangtaruna ARAS</h2>
  <div id="userInfo"></div>

  <div class="balance-section">
    <h3>Saldo Saat Ini: <span id="saldo">-</span></h3>
  </div>

  <form id="addSaldoForm">
    <h4>Tambah Saldo</h4>
    <input type="number" id="addAmount" placeholder="Jumlah (Rp)" min="1" required />
    <button type="submit">Tambah</button>
  </form>

  <form id="withdrawSaldoForm">
    <h4>Tarik Saldo</h4>
    <input type="number" id="withdrawAmount" placeholder="Jumlah (Rp)" min="1" required />
    <button type="submit">Tarik</button>
  </form>

  <button id="btnLogout">Logout</button>

  <div id="message" style="color:green; margin-top:10px;"></div>
  <div id="errorMsg" style="color:red; margin-top:10px;"></div>
</div>
</body>
</html>
const auth = firebase.auth();
const db = firebase.firestore();

const userInfoEl = document.getElementById('userInfo');
const saldoEl = document.getElementById('saldo');
const messageEl = document.getElementById('message');
const errorMsgEl = document.getElementById('errorMsg');

let currentUser = null;

auth.onAuthStateChanged(async user => {
  if (!user) {
    window.location.href = 'index.html';
    return;
  }
  currentUser = user;
  const userDoc = await db.collection('users').doc(user.uid).get();
  if (!userDoc.exists) {
    errorMsgEl.innerText = 'Data user tidak ditemukan.';
    return;
  }
  const userData = userDoc.data();
  userInfoEl.innerHTML = `<p>Selamat datang, <b>${userData.name || 'User'}</b></p>`;
  saldoEl.innerText = formatRupiah(userData.saldo || 0);
});

function formatRupiah(angka) {
  return 'Rp ' + angka.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.');
}

document.getElementById('addSaldoForm').addEventListener('submit', async e => {
  e.preventDefault();
  messageEl.innerText = '';
  errorMsgEl.innerText = '';

  let amount = parseInt(document.getElementById('addAmount').value);
  if (isNaN(amount) || amount <= 0) {
    errorMsgEl.innerText = 'Masukkan jumlah tambah saldo yang valid.';
    return;
  }

  try {
    const userRef = db.collection('users').doc(currentUser.uid);
    await db.runTransaction(async (transaction) => {
      const userDoc = await transaction.get(userRef);
      if (!userDoc.exists) throw 'User tidak ditemukan';

      const newSaldo = (userDoc.data().saldo || 0) + amount;
      transaction.update(userRef, { saldo: newSaldo });
    });
    messageEl.innerText = 'Saldo berhasil ditambahkan.';
    document.getElementById('addAmount').value = '';
    // Update saldo display
    const updatedUserDoc = await db.collection('users').doc(currentUser.uid).get();
    saldoEl.innerText = formatRupiah(updatedUserDoc.data().saldo || 0);
  } catch (err) {
    errorMsgEl.innerText = err.toString();
  }
});

document.getElementById('withdrawSaldoForm').addEventListener('submit', async e => {
  e.preventDefault();
  messageEl.innerText = '';
  errorMsgEl.innerText = '';

  let amount = parseInt(document.getElementById('withdrawAmount').value);
  if (isNaN(amount) || amount <= 0) {
    errorMsgEl.innerText = 'Masukkan jumlah tarik saldo yang valid.';
    return;
  }

  try {
    const userRef = db.collection('users').doc(currentUser.uid);
    await db.runTransaction(async (transaction) => {
      const userDoc = await transaction.get(userRef);
      if (!userDoc.exists) throw 'User tidak ditemukan';

      const currentSaldo = userDoc.data().saldo || 0;
      if (amount > currentSaldo) throw 'Saldo tidak cukup untuk penarikan';

      const newSaldo = currentSaldo - amount;
      transaction.update(userRef, { saldo: newSaldo });
    });
    messageEl.innerText = 'Saldo berhasil ditarik.';
    document.getElementById('withdrawAmount').value = '';
    // Update saldo display
    const updatedUserDoc = await db.collection('users').doc(currentUser.uid).get();
    saldoEl.innerText = formatRupiah(updatedUserDoc.data().saldo || 0);
  } catch (err) {
    errorMsgEl.innerText = err.toString();
  }
});

document.getElementById('btnLogout').addEventListener('click', async () => {
  await auth.signOut();
  window.location.href = 'index.html';
});
// Import dan inisialisasi Firebase
import { initializeApp } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-app.js";
import { getAuth } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-auth.js";
import { getFirestore } from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyDVUHmKIjaHWdoAdB20cmAM06LWdFJBRW8",
  authDomain: "dompet-karangtaruna-aras.firebaseapp.com",
  projectId: "dompet-karangtaruna-aras",
  storageBucket: "dompet-karangtaruna-aras.firebasestorage.app",
  messagingSenderId: "373945571180",
  appId: "1:373945571180:web:28823141c06db4e7d6c549",
  measurementId: "G-PWB8RX7JF0"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

export { auth, db };
/* Styling dasar responsif */

body {
  font-family: Arial, sans-serif;
  margin: 0; padding: 0;
  background: #f7f7f7;
  color: #333;
}

.container {
  max-width: 400px;
  margin: 40px auto;
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
}

h2, h3, h4 {
  margin-bottom: 15px;
}

input[type="text"],
input[type="password"],
input[type="number"] {
  width: 100%;
  padding: 10px;
  margin: 6px 0 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

button {
  width: 100%;
  background-color: #0288d1;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
}

button:disabled {
  background-color: #aaa;
  cursor: not-allowed;
}

button:hover:not(:disabled) {
  background-color: #0277bd;
}

a {
  color: #0288d1;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

.balance-section {
  margin-bottom: 25px;
  font-size: 1.2em;
  font-weight: bold;
  color: #222;
}
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Admin Panel Dompet Karangtaruna ARAS</title>
<link rel="stylesheet" href="style.css" />
<script type="module" src="admin_firebase.js" defer></script>
</head>
<body>
<div class="container">
  <h2>Admin Panel Dompet Karangtaruna ARAS</h2>
  <button id="btnLogout" style="margin-bottom: 20px;">Logout</button>

  <h3>Data User</h3>
  <table border="1" width="100%" cellpadding="8" cellspacing="0">
    <thead>
      <tr>
        <th>Nama</th>
        <th>Email / No HP</th>
        <th>Saldo (Rp)</th>
        <th>Aksi</th>
      </tr>
    </thead>
    <tbody id="userTableBody">
      <tr><td colspan="4">Loading data...</td></tr>
    </tbody>
  </table>

  <div id="errorMsg" style="color:red; margin-top: 10px;"></div>
  <div id="message" style="color:green; margin-top: 10px;"></div>
</div>
</body>
</html>
import { auth, db } from './firebase-config.js';
import {
  collection,
  getDocs,
  doc,
  updateDoc,
} from "https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore.js";

const userTableBody = document.getElementById('userTableBody');
const errorMsg = document.getElementById('errorMsg');
const message = document.getElementById('message');
const btnLogout = document.getElementById('btnLogout');

btnLogout.addEventListener('click', async () => {
  await auth.signOut();
  window.location.href = 'index.html';
});

auth.onAuthStateChanged(async user => {
  if (!user) {
    window.location.href = 'index.html';
    return;
  }
  // Optional: cek role admin dari Firestore user collection
  const userDoc = await db.collection('users').doc(user.uid).get();
  if (!userDoc.exists || userDoc.data().role !== 'admin') {
    alert('Anda bukan admin!');
    await auth.signOut();
    window.location.href = 'index.html';
    return;
  }
  loadUsers();
});

async function loadUsers() {
  errorMsg.innerText = '';
  message.innerText = '';
  userTableBody.innerHTML = '<tr><td colspan="4">Loading data...</td></tr>';
  try {
    const querySnapshot = await getDocs(collection(db, "users"));
    if (querySnapshot.empty) {
      userTableBody.innerHTML = '<tr><td colspan="4">Belum ada data user.</td></tr>';
      return;
    }
    userTableBody.innerHTML = '';
    querySnapshot.forEach(docSnap => {
      const data = docSnap.data();
      const uid = docSnap.id;
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${data.name || '-'}</td>
        <td>${data.email || data.phone || '-'}</td>
        <td>${formatRupiah(data.saldo || 0)}</td>
        <td>
          <button onclick="showUpdateSaldoPrompt('${uid}', ${data.saldo || 0})">Update Saldo</button>
        </td>
      `;
      userTableBody.appendChild(tr);
    });
  } catch (err) {
    errorMsg.innerText = err.message;
  }
}

window.showUpdateSaldoPrompt = async function(uid, currentSaldo) {
  let input = prompt(`Saldo saat ini: ${formatRupiah(currentSaldo)}\nMasukkan saldo baru (Rp):`);
  if (input === null) return; // batal
  input = input.replace(/\D/g, ''); // hanya angka
  const newSaldo = parseInt(input);
  if (isNaN(newSaldo) || newSaldo < 0) {
    alert('Saldo tidak valid!');
    return;
  }
  try {
    await updateDoc(doc(db, 'users', uid), { saldo: newSaldo });
    message.innerText = 'Saldo berhasil diupdate.';
    loadUsers();
  } catch (err) {
    errorMsg.innerText = err.message;
  }
};

function formatRupiah(angka) {
  return 'Rp ' + angka.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.');
}

<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LMS PAI Online</title>
<style>
body{font-family:Arial,sans-serif;margin:0;padding:0;background:#f9f9f9}
header{background:#006400;color:#fff;padding:10px;text-align:center}
nav{background:#228B22;padding:10px;display:flex;flex-wrap:wrap;gap:5px}
nav button{flex:1;min-width:90px;padding:10px;border:none;background:#fff;color:#006400;border-radius:5px;cursor:pointer}
nav button:hover{background:#90EE90}
section{padding:15px;display:none}
.active{display:block}
table{width:100%;border-collapse:collapse;margin-top:10px;font-size:14px}
th,td{border:1px solid #ccc;padding:6px;text-align:center}
th{background:#eee}
input,textarea,select{padding:7px;margin:5px 0;width:100%;max-width:500px;box-sizing:border-box}
.table-container{overflow-x:auto}
@media(max-width:768px){nav{flex-direction:column}nav button{width:100%}}
</style>
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, createUserWithEmailAndPassword, signOut, sendPasswordResetEmail } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-auth.js";
import { getFirestore, collection, addDoc, getDocs, updateDoc, doc, deleteDoc } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-firestore.js";

// === FIREBASE CONFIG ===
const firebaseConfig = {
  apiKey: "API_KEY",
  authDomain: "PROJECT_ID.firebaseapp.com",
  projectId: "PROJECT_ID",
  storageBucket: "PROJECT_ID.appspot.com",
  messagingSenderId: "SENDER_ID",
  appId: "APP_ID"
};
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

let currentUser = null;
let currentRole = null;

// ===== LOGIN =====
window.login = async () => {
  const email = document.getElementById("email").value;
  const pass = document.getElementById("password").value;
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email, pass);
    currentUser = userCredential.user;
    if(email==="admin@lms.com"){currentRole="admin"; showAdminDashboard();} 
    else{currentRole="siswa"; showSiswaDashboard();}
  } catch(err){alert("Login gagal: "+err.message);}
};

// ===== LOGOUT =====
window.logout = async () => {
  await signOut(auth); currentUser=null; currentRole=null;
  document.querySelectorAll("section").forEach(s=>s.style.display="none");
  document.getElementById("loginSection").style.display="block";
};

// ===== ADMIN DASHBOARD =====
async function showAdminDashboard(){
  document.querySelectorAll("section").forEach(s=>s.style.display="none");
  document.getElementById("adminDashboard").style.display="block";
  loadSiswaAdmin(); loadMateriAdmin(); loadSoalAdmin(); loadKuisAdmin(); loadNilaiAdmin();
}

// ===== SISWA DASHBOARD =====
async function showSiswaDashboard(){
  document.querySelectorAll("section").forEach(s=>s.style.display="none");
  document.getElementById("siswaDashboard").style.display="block";
  loadMateriSiswa(); loadSoalSiswa(); loadKuisSiswa(); loadProfilSiswa(); loadNilaiSiswa();
}

// ===== ADMIN: TAMBAH SISWA =====
window.tambahSiswa = async () => {
  const email = document.getElementById("sEmail").value;
  const pass = document.getElementById("sPass").value;
  const nama = document.getElementById("sNama").value;
  const kelas = document.getElementById("sKelas").value;
  const cabor = document.getElementById("sCabor").value;
  try{
    await createUserWithEmailAndPassword(auth,email,pass);
    await addDoc(collection(db,"siswa"), {email,nama,kelas,cabor,nilai:{aktivitas:0,soal:0,kuis:0,akhir:0}});
    alert("Siswa berhasil ditambah!");
    loadSiswaAdmin();
  }catch(err){alert("Gagal tambah siswa: "+err.message);}
};

// ===== ADMIN: RESET PASSWORD SISWA =====
window.resetPasswordSiswa = async (email) => {
  try{ await sendPasswordResetEmail(auth,email); alert("Link reset password terkirim ke "+email);}
  catch(err){alert("Gagal reset password: "+err.message);}
};

// ===== ADMIN: LOAD DATA SISWA =====
async function loadSiswaAdmin(){
  const tbl = document.getElementById("akunTable");
  tbl.innerHTML="<tr><th>Email</th><th>Nama</th><th>Kelas</th><th>Cabor</th><th>Aksi</th></tr>";
  const querySnapshot = await getDocs(collection(db,"siswa"));
  querySnapshot.forEach((docSnap)=>{
    const data = docSnap.data();
    const id = docSnap.id;
    const row = tbl.insertRow();
    row.innerHTML=`<td>${data.email}</td><td>${data.nama}</td><td>${data.kelas}</td><td>${data.cabor}</td>
      <td>
      <button onclick="resetPasswordSiswa('${data.email}')">Reset Password</button>
      <button onclick="hapusSiswa('${id}')">Hapus</button>
      </td>`;
  });
}

// ===== ADMIN: HAPUS SISWA =====
window.hapusSiswa = async (id) => {
  if(confirm("Hapus siswa ini?")){ await deleteDoc(doc(db,"siswa",id)); loadSiswaAdmin();}
};

// ===== EXPORT NILAI =====
window.exportExcel = async () => {
  let csv="Nama,Kelas,Aktivitas,Soal,Kuis,Akhir\n";
  const querySnapshot = await getDocs(collection(db,"siswa"));
  querySnapshot.forEach((docSnap)=>{
    const s=docSnap.data();
    csv+=`${s.nama},${s.kelas},${s.nilai.aktivitas},${s.nilai.soal},${s.nilai.kuis},${s.nilai.akhir}\n`;
  });
  const blob=new Blob([csv],{type:"text/csv"});
  const a=document.createElement("a");
  a.href=URL.createObjectURL(blob);
  a.download="nilai.csv";
  a.click();
};
</script>
</head>
<body>
<header><h2>LMS PAI Online</h2></header>

<section id="loginSection" style="display:block">
  <h3>Login</h3>
  <input id="email" placeholder="Email"><br>
  <input id="password" type="password" placeholder="Password"><br>
  <button onclick="login()">Login</button>
</section>

<section id="adminDashboard">
  <h3>Dashboard Admin</h3>
  <nav>
    <button onclick="loadSiswaAdmin()">Akun Siswa</button>
    <button onclick="logout()">Logout</button>
  </nav>
  <h4>Tambah Siswa</h4>
  <input id="sEmail" placeholder="Email siswa"><br>
  <input id="sPass" type="password" placeholder="Password siswa"><br>
  <input id="sNama" placeholder="Nama"><br>
  <input id="sKelas" placeholder="Kelas"><br>
  <input id="sCabor" placeholder="Cabang Olahraga"><br>
  <button onclick="tambahSiswa()">Tambah</button>
  <div class="table-container">
    <table id="akunTable"></table>
  </div>
  <button onclick="exportExcel()">Export Excel</button>
</section>

<section id="siswaDashboard">
  <h3>Dashboard Siswa</h3>
  <nav>
    <button onclick="loadMateriSiswa()">Materi</button>
    <button onclick="loadSoalSiswa()">Soal</button>
    <button onclick="loadKuisSiswa()">Kuis</button>
    <button onclick="loadNilaiSiswa()">Nilai</button>
    <button onclick="loadProfilSiswa()">Profil</button>
    <button onclick="logout()">Logout</button>
  </nav>
  <div id="materiSiswa"></div>
  <div id="soalSiswa"></div>
  <div id="kuisSiswa"></div>
  <div id="nilaiSiswa"></div>
  <div id="profilSiswa"></div>
</section>
</body>
</html>

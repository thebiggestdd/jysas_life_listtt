<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Vanity Gallery — The Biggest D of All</title>

  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.5.0/firebase-storage.js"></script>

  <style>
    /* Your original styles remain unchanged */
    body {
      margin: 0;
      font-family: Georgia, "Times New Roman", serif;
      background: #fafafa;
      color: #111;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 40px 20px;
    }
    h1 { margin-bottom: 10px; font-size: 2rem; }
    p { color: #555; margin-bottom: 30px; }
    .controls {
      display: none;
      flex-direction: column;
      align-items: center;
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.1);
      margin-bottom: 40px;
      width: min(600px, 90%);
    }
    textarea {
      width: 100%;
      height: 100px;
      resize: vertical;
      padding: 10px;
      font-family: Georgia, "Times New Roman", serif;
      font-size: 1rem;
      border: 1px solid #ccc;
      border-radius: 6px;
      margin-top: 10px;
    }
    input[type="file"] { margin: 10px 0; }
    button {
      background: black;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 6px;
      cursor: pointer;
      font-family: Georgia, "Times New Roman", serif;
      margin: 5px;
    }
    button:hover { background: #333; }
    .gallery {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 20px;
      width: 100%;
      max-width: 1100px;
    }
    .card {
      background: white;
      border-radius: 10px;
      overflow: hidden;
      box-shadow: 0 6px 18px rgba(0,0,0,0.1);
      display: flex;
      flex-direction: column;
    }
    .card img {
      width: 100%;
      height: 200px;
      object-fit: cover;
    }
    .card-content {
      padding: 15px;
      flex: 1;
    }
    .card-content p {
      margin: 0;
      font-size: 0.95rem;
      line-height: 1.5;
    }
    .date {
      color: #777;
      font-size: 0.8rem;
      margin-top: 8px;
    }
    .delete-btn {
      background: #e74c3c;
      color: white;
      border: none;
      padding: 5px 10px;
      border-radius: 4px;
      cursor: pointer;
      font-size: 0.8rem;
      margin-top: 10px;
    }
    .delete-btn:hover { background: #c0392b; }
  </style>
</head>
<body>

<h1>Vanity Gallery</h1>
<p>Where only the biggest D may post their thoughts and pictures.</p>

<div id="login">
  <p>Enter password to unlock editor access:</p>
  <input type="password" id="passwordInput" placeholder="Enter password">
  <button id="loginBtn">Unlock</button>
</div>

<div class="controls" id="controls">
  <input type="file" id="imageInput" accept="image/*">
  <textarea id="textInput" placeholder="Write your vanity thought here..."></textarea>
  <button id="saveBtn">Save Card</button>
  <button id="logoutBtn">Lock / Logout</button>
</div>

<div class="gallery" id="gallery"></div>

<script>
  // Firebase config
  const firebaseConfig = {
    apiKey: "AIzaSyAGD-a1VQWWXPT3OGY3dcnGHgnp2dlPXf8",
    authDomain: "vanitygallery-de239.firebaseapp.com",
    projectId: "vanitygallery-de239",
    storageBucket: "vanitygallery-de239.appspot.com",
    messagingSenderId: "787197040457",
    appId: "1:787197040457:web:f1d31191554d46d642ec98",
    measurementId: "G-3YK000JSK4"
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
  const storage = firebase.storage();

  const ADMIN_PASSWORD = "OnlytheRealDKnows";

  const gallery = document.getElementById("gallery");
  const saveBtn = document.getElementById("saveBtn");
  const imageInput = document.getElementById("imageInput");
  const textInput = document.getElementById("textInput");
  const controls = document.getElementById("controls");
  const loginDiv = document.getElementById("login");
  const loginBtn = document.getElementById("loginBtn");
  const passwordInput = document.getElementById("passwordInput");
  const logoutBtn = document.getElementById("logoutBtn");

  loginBtn.onclick = () => {
    if (passwordInput.value === ADMIN_PASSWORD) {
      localStorage.setItem("vanity_access", "granted");
      loginDiv.style.display = "none";
      controls.style.display = "flex";
      renderCards();
    } else {
      alert("Wrong password. Access denied, pretender of D.");
    }
  };

  logoutBtn.onclick = () => {
    localStorage.removeItem("vanity_access");
    alert("Locked! Only the chosen D can unlock again.");
    location.reload();
  };

  if (localStorage.getItem("vanity_access") === "granted") {
    loginDiv.style.display = "none";
    controls.style.display = "flex";
  }

  saveBtn.onclick = async () => {
    const file = imageInput.files[0];
    const text = textInput.value.trim();
    if (!file && !text) return alert("At least write something or add a photo, the biggest D.");

    let imageUrl = null;

    if (file) {
      const storageRef = storage.ref(`images/${Date.now()}_${file.name}`);
      await storageRef.put(file);
      imageUrl = await storageRef.getDownloadURL();
    }

    await db.collection("cards").add({
      text: text,
      image: imageUrl,
      date: new Date().toISOString()
    });

    imageInput.value = "";
    textInput.value = "";
    renderCards();
  };

  async function renderCards() {
    gallery.innerHTML = "";
    const snapshot = await db.collection("cards").orderBy("date", "desc").get();

    if (snapshot.empty) {
      gallery.innerHTML = "<p style='color:#888;text-align:center;'>No vanity cards yet. Time to add your first brilliance, the biggest D of all.</p>";
      return;
    }

    const isOwner = localStorage.getItem("vanity_access") === "granted";

    snapshot.forEach(doc => {
      const card = doc.data();
      const div = document.createElement("div");
      div.className = "card";
      div.innerHTML = `
        ${card.image ? `<img src="${card.image}" alt="Vanity Image">` : ""}
        <div class="card-content">
          <p>${escapeHtml(card.text)}</p>
          <div class="date">${new Date(card.date).toLocaleString()}</div>
        </div>
      `;
      gallery.appendChild(div);
    });
  }

  function escapeHtml(text) {
    return text.replace(/[&<>"']/g, m => ({
      '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
    }[m]));
  }

  renderCards();
</script>

</body>
</html>

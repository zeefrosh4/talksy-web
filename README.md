<!DOCTYPE html>
<html>
<head>
    <title>Talksy Mini Chat</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(to right, #ffecd2, #fcb69f);
            padding: 20px;
            margin: 0;
        }

        h1 {
            color: #333;
            text-align: center;
        }

        input {
            padding: 12px;
            width: 60%;
            border-radius: 8px;
            border: none;
            outline: none;
            margin-bottom: 10px;
        }

        button {
            padding: 10px 16px;
            border-radius: 8px;
            border: none;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s ease;
        }

        .post {
            padding: 12px 16px;
            margin: 8px 0;
            border-radius: 16px;
            max-width: 70%;
            box-shadow: 0 2px 6px rgba(0,0,0,0.15);
            position: relative;
            animation: popIn 0.3s ease;
            word-wrap: break-word;
        }

        .post.right {
            background: linear-gradient(135deg, #6BCB77, #4D96FF);
            color: white;
            margin-left: 30%;
            text-align: right;
        }

        .post.left {
            background: #fff;
            color: black;
            margin-right: 30%;
            text-align: left;
        }

        @keyframes popIn {
            0% {opacity: 0; transform: scale(0.8);}
            100% {opacity: 1; transform: scale(1);}
        }

        .likeBtn { background: #ff4b5c; color: white; margin-right: 6px; }
        .likeBtn:hover { opacity: 0.8; }
        .likeBtn:active { transform: scale(1.2); transition: transform 0.2s; }

        .deleteBtn { background: #555; color: white; }
        .deleteBtn:hover { opacity: 0.8; }

        .emojiBtn { background: none; border: none; font-size: 18px; cursor: pointer; margin-left: 4px; }
        .emojiBtn:hover { transform: scale(1.2); }

        .likeCount, .reactionCount { font-weight: bold; margin-left: 4px; }
    </style>
</head>
<body>

<h1>Talksy Mini Chat</h1>

<input type="text" id="usernameInput" placeholder="Your name">
<br>
<input type="text" id="messageInput" placeholder="What's on your mind?">
<button onclick="sendMessage()">Post</button>

<div id="posts"></div>

<!-- Firebase SDKs -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<script>
  const firebaseConfig = {
    apiKey: "AIzaSyCK3dYhgPl-CWJ9JuiI2rVuuv00Q7Do9f4",
    authDomain: "talksy-mini.firebaseapp.com",
    projectId: "talksy-mini",
    storageBucket: "talksy-mini.appspot.com",
    messagingSenderId: "1044754891732",
    appId: "1:1044754891732:web:9b933f3d18a17e246a8c41"
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
  const postsDiv = document.getElementById("posts");

  // Generate fixed color for username
  function hashColor(name){
    let colors = ['#FF6B6B','#6BCB77','#4D96FF','#FFD93D','#FF8C42','#9D4EDD'];
    let hash = 0;
    for(let i=0;i<name.length;i++){ hash = name.charCodeAt(i) + ((hash<<5)-hash);}
    return colors[Math.abs(hash) % colors.length];
  }

  function sendMessage() {
    const username = document.getElementById("usernameInput").value.trim() || "Anonymous";
    const message = document.getElementById("messageInput").value.trim();
    if(!message) return alert("Write something first!");

    db.collection("posts").add({
      username: username,
      message: message,
      timestamp: Date.now(),
      likes: 0,
      reactions: { "❤️": 0, "😂": 0, "😮": 0 }
    });

    document.getElementById("messageInput").value = "";
  }

  db.collection("posts").orderBy("timestamp")
    .onSnapshot(snapshot => {
      postsDiv.innerHTML = "";
      const myName = document.getElementById("usernameInput").value.trim() || "Anonymous";

      snapshot.forEach(doc => {
        const post = doc.data();
        const postDiv = document.createElement("div");
        postDiv.className = (post.username === myName) ? "post right" : "post left";

        const time = new Date(post.timestamp);
        const formattedTime = `${time.getHours().toString().padStart(2,'0')}:${time.getMinutes().toString().padStart(2,'0')}`;

        const userColor = hashColor(post.username);

        postDiv.innerHTML = `
          <p><strong style="color:${userColor}">${post.username}:</strong> ${post.message}</p>
          <small style="color: gray;">Posted at ${formattedTime}</small>
          <br>
          <button class="likeBtn">Like ❤️</button>
          <span class="likeCount">${post.likes}</span>
          <button class="deleteBtn">Delete ❌</button>
          <span>Reactions: </span>
          <button class="emojiBtn">❤️ ${post.reactions["❤️"]}</button>
          <button class="emojiBtn">😂 ${post.reactions["😂"]}</button>
          <button class="emojiBtn">😮 ${post.reactions["😮"]}</button>
        `;

        postsDiv.appendChild(postDiv);

        // Like button
        const likeBtn = postDiv.querySelector(".likeBtn");
        likeBtn.addEventListener("click", () => {
          db.collection("posts").doc(doc.id).update({ likes: post.likes + 1 });
        });

        // Delete button
        const deleteBtn = postDiv.querySelector(".deleteBtn");
        deleteBtn.addEventListener("click", () => {
          db.collection("posts").doc(doc.id).delete();
        });

        // Emoji reactions
        const emojiBtns = postDiv.querySelectorAll(".emojiBtn");
        emojiBtns.forEach(btn => {
          btn.addEventListener("click", () => {
            const emoji = btn.textContent.trim().split(" ")[0];
            const current = post.reactions[emoji] || 0;
            const updated = { ...post.reactions };
            updated[emoji] = current + 1;
            db.collection("posts").doc(doc.id).update({ reactions: updated });
          });
        });

      });

      postsDiv.scrollTop = postsDiv.scrollHeight;
    });

  document.getElementById("messageInput").addEventListener("keypress", function(event) {
    if(event.key === "Enter") sendMessage();
  });
</script>
</body>
</html>

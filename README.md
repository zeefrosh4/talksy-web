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
            width: 65%;
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

        .likeBtn {
            background: #ff4b5c;
            color: white;
            margin-right: 6px;
        }

        .likeBtn:hover {
            opacity: 0.8;
        }

        .likeBtn:active {
            transform: scale(1.2);
            transition: transform 0.2s;
        }

        .deleteBtn {
            background: #555;
            color: white;
        }

        .deleteBtn:hover {
            opacity: 0.8;
        }

        .likeCount {
            font-weight: bold;
            margin-left: 4px;
        }
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
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>
  const firebaseConfig = {
    apiKey: "AIzaSyCK3dYhgPl-CWJ9JuiI2rVuuv00Q7Do9f4",
    authDomain: "talksy-mini.firebaseapp.com",
    projectId: "talksy-mini",
    storageBucket: "talksy-mini.firebasestorage.app",
    messagingSenderId: "1044754891732",
    appId: "1:1044754891732:web:9b933f3d18a17e246a8c41",
    databaseURL: "https://talksy-mini-default-rtdb.firebaseio.com/"
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();
  const postsDiv = document.getElementById("posts");

  const colors = ['#FF6B6B','#6BCB77','#4D96FF','#FFD93D','#FF8C42','#9D4EDD'];

  function sendMessage() {
    const username = document.getElementById("usernameInput").value.trim() || "Anonymous";
    const message = document.getElementById("messageInput").value.trim();
    if(!message) return alert("Write something first!");

    const newPostRef = db.ref("posts").push();
    newPostRef.set({
      username: username,
      message: message,
      timestamp: Date.now(),
      likes: 0
    });

    document.getElementById("messageInput").value = "";
  }

  db.ref("posts").on("value", snapshot => {
    postsDiv.innerHTML = "";
    const posts = snapshot.val();
    const myName = document.getElementById("usernameInput").value.trim() || "Anonymous";

    if(posts){
      Object.keys(posts).forEach(key => {
        const post = posts[key];
        const postDiv = document.createElement("div");

        // Align right if it's your post
        postDiv.className = (post.username === myName) ? "post right" : "post left";

        // Format time HH:MM
        const time = new Date(post.timestamp);
        const hours = time.getHours().toString().padStart(2,'0');
        const minutes = time.getMinutes().toString().padStart(2,'0');
        const formattedTime = `${hours}:${minutes}`;

        // Random username color
        const usernameColor = colors[Math.floor(Math.random()*colors.length)];

        postDiv.innerHTML = `
          <p><strong style="color:${usernameColor}">${post.username}:</strong> ${post.message}</p>
          <small style="color: gray;">Posted at ${formattedTime}</small>
          <br>
          <button class="likeBtn">Like ❤️</button>
          <button class="deleteBtn">Delete ❌</button>
          <span class="likeCount">${post.likes}</span> Likes
        `;

        postsDiv.appendChild(postDiv);

        // Like button
        const likeBtn = postDiv.querySelector(".likeBtn");
        const likeCount = postDiv.querySelector(".likeCount");
        likeBtn.addEventListener("click", () => {
          db.ref("posts/" + key + "/likes").set(post.likes + 1);
        });

        // Delete button
        const deleteBtn = postDiv.querySelector(".deleteBtn");
        deleteBtn.addEventListener("click", () => {
          db.ref("posts/" + key).remove();
        });
      });

      // Auto-scroll to newest post
      postsDiv.scrollTop = postsDiv.scrollHeight;
    }
  });

  // Enter key to post
  document.getElementById("messageInput").addEventListener("keypress", function(event) {
    if(event.key === "Enter") sendMessage();
  });
</script>

</body>
</html>

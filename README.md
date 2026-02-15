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
            background: white;
            padding: 12px;
            margin-top: 12px;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            position: relative;
            transition: transform 0.2s ease;
        }

        .post:hover {
            transform: translateY(-2px);
        }

        .likeBtn {
            background: #ff4b5c;
            color: white;
            margin-right: 6px;
        }

        .likeBtn:hover {
            opacity: 0.8;
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

<input type="text" id="usernameInput" placeholder="Your name (optional)">
<br>
<input type="text" id="messageInput" placeholder="What's on your mind?">
<button onclick="sendMessage()">Post</button>

<div id="posts"></div>

<!-- Firebase SDKs -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<script>
  // Your Firebase configuration
  const firebaseConfig = {
    apiKey: "AIzaSyCK3dYhgPl-CWJ9JuiI2rVuuv00Q7Do9f4",
    authDomain: "talksy-mini.firebaseapp.com",
    projectId: "talksy-mini",
    storageBucket: "talksy-mini.firebasestorage.app",
    messagingSenderId: "1044754891732",
    appId: "1:1044754891732:web:9b933f3d18a17e246a8c41",
    databaseURL: "https://talksy-mini-default-rtdb.firebaseio.com/"
  };

  // Initialize Firebase
  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  const postsDiv = document.getElementById("posts");

  function sendMessage() {
    const username = document.getElementById("usernameInput").value || "Anonymous";
    const message = document.getElementById("messageInput").value;

    if(message === "") return alert("Write something first!");

    const newPostRef = db.ref("posts").push();
    newPostRef.set({
      username: username,
      message: message,
      timestamp: Date.now(),
      likes: 0
    });

    document.getElementById("messageInput").value = "";
  }

  // Real-time listener
  db.ref("posts").on("value", snapshot => {
    postsDiv.innerHTML = ""; // clear previous posts
    const posts = snapshot.val();
    if(posts){
      Object.keys(posts).forEach(key => {
        const post = posts[key];
        const postDiv = document.createElement("div");
        postDiv.className = "post";

        const time = new Date(post.timestamp).toLocaleTimeString();

        postDiv.innerHTML = `
          <p><strong>${post.username}:</strong> ${post.message}</p>
          <small style="color: gray;">Posted at ${time}</small>
          <br>
          <button class="likeBtn">Like ❤️</button>
          <button class="deleteBtn">Delete ❌</button>
          <span class="likeCount">${post.likes}</span> Likes
        `;

        // Append post
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
    }
  });

  // Enter key to post
  document.getElementById("messageInput").addEventListener("keypress", function(event) {
    if(event.key === "Enter") sendMessage();
  });
</script>

</body>
</html>

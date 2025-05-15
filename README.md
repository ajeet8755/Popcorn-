<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Admin Video Player</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <style>
    body { font-family: sans-serif; margin: 0; background: #f4f4f4; touch-action: manipulation; }
    header { background: #2196F3; color: white; padding: 15px; text-align: center; font-size: 18px; position: sticky; top: 0; display: flex; justify-content: space-between; align-items: center; }
    .folder, .video-item { background: white; margin: 10px; padding: 15px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; }
    .hidden { display: none; }
    video { width: 100%; border-radius: 10px; margin-top: 10px; }
    .admin-btn, .login-btn, .logout-btn, .share-btn { margin: 10px 0; padding: 12px; width: 100%; background: #4CAF50; color: white; border: none; border-radius: 8px; font-size: 16px; }
    .logout-btn { background: #f44336; }
    .delete-btn, .route-btn { background: #f44336; color: white; border: none; padding: 6px 10px; border-radius: 6px; font-size: 14px; margin-left: 5px; margin-top: 5px; }
    .route-btn { background: #2196F3; }
    button { touch-action: manipulation; }
    .back-btn { margin: 10px; padding: 10px 15px; background: #2196F3; color: white; border: none; border-radius: 8px; font-size: 16px; width: auto; }
  </style>
</head>
<body>

<header>
  <span>Video Player</span>
  <div>
    <button class="login-btn" id="loginBtn" onclick="loginAdmin()">Admin Login</button>
    <button class="logout-btn hidden" id="logoutBtn" onclick="logoutAdmin()">Logout</button>
  </div>
</header>

<div id="folderView">
  <div class="folder" onclick="openFolder('18+')">🔞 18+</div>
  <div class="folder" onclick="openFolder('30+')">🎥 30+</div>
  <div class="folder" onclick="openFolder('Desi')">🇮🇳 Desi</div>
  <div class="folder" onclick="openFolder('Russian')">🇷🇺 Russian</div>
  <div class="folder" onclick="openFolder('Pakistani')">🇵🇰 Pakistani</div>
  <button class="share-btn" onclick="shareApp()">Share App Link</button>
</div>

<div id="videoView" class="hidden">
  <button class="back-btn" onclick="goBack()">← Back</button>
  <div id="videoList"></div>
  <button class="admin-btn hidden" id="uploadBtn" onclick="adminUpload()">+ Upload Video</button>
</div>

<div id="playerView" class="hidden">
  <button class="back-btn" onclick="backToVideos()">← Back to Videos</button>
  <video id="videoPlayer" controls playsinline></video>
</div>

<script>
  const adminPassword = "875549";
  let videoData = JSON.parse(localStorage.getItem('videoData')) || {
    '18+': [], '30+': [], 'Desi': [], 'Russian': [], 'Pakistani': []
  };
  let currentFolder = '';

  function isAdmin() {
    return sessionStorage.getItem('isAdmin') === 'true';
  }

  function showAdminUI() {
    document.getElementById('uploadBtn').classList.toggle('hidden', !isAdmin());
    document.getElementById('logoutBtn').classList.toggle('hidden', !isAdmin());
    document.getElementById('loginBtn').classList.toggle('hidden', isAdmin());
    renderVideoList();
  }

  function loginAdmin() {
    const password = prompt("Enter admin password:");
    if (password === adminPassword) {
      sessionStorage.setItem('isAdmin', 'true');
      alert("Admin login successful.");
      showAdminUI();
    } else {
      alert("Incorrect password.");
    }
  }

  function logoutAdmin() {
    sessionStorage.removeItem('isAdmin');
    alert("Logged out.");
    showAdminUI();
  }

  function openFolder(folder) {
    currentFolder = folder;
    document.getElementById('folderView').classList.add('hidden');
    document.getElementById('videoView').classList.remove('hidden');
    renderVideoList();
    showAdminUI();
  }

  function goBack() {
    document.getElementById('videoView').classList.add('hidden');
    document.getElementById('folderView').classList.remove('hidden');
  }

  function backToVideos() {
    document.getElementById('playerView').classList.add('hidden');
    document.getElementById('videoView').classList.remove('hidden');
  }

  function renderVideoList() {
    const list = document.getElementById('videoList');
    list.innerHTML = '';
    videoData[currentFolder].forEach((video, index) => {
      const div = document.createElement('div');
      div.className = 'video-item';

      const title = document.createElement('span');
      title.textContent = `🎬 ${video.name}`;
      title.style.flex = '1';

      div.appendChild(title);

      if (isAdmin()) {
        const deleteBtn = document.createElement('button');
        deleteBtn.className = 'delete-btn';
        deleteBtn.textContent = 'Delete';
        deleteBtn.onclick = (e) => {
          e.stopPropagation();
          deleteVideo(index);
        };

        const routeBtn = document.createElement('button');
        routeBtn.className = 'route-btn';
        routeBtn.textContent = 'Route';
        routeBtn.onclick = (e) => {
          e.stopPropagation();
          routeVideo(video.url);
        };

        div.appendChild(deleteBtn);
        div.appendChild(routeBtn);
      }

      div.onclick = () => playVideo(index);
      list.appendChild(div);
    });
  }

  function playVideo(index) {
    const video = document.getElementById('videoPlayer');
    video.src = videoData[currentFolder][index].url;
    video.play();
    document.getElementById('videoView').classList.add('hidden');
    document.getElementById('playerView').classList.remove('hidden');
  }

  function deleteVideo(index) {
    if (confirm("Are you sure you want to delete this video?")) {
      videoData[currentFolder].splice(index, 1);
      localStorage.setItem('videoData', JSON.stringify(videoData));
      renderVideoList();
    }
  }

  function routeVideo(url) {
    navigator.clipboard.writeText(url).then(() => {
      alert("Video URL copied to clipboard:\n" + url);
      window.open(url, '_blank');
    });
  }

  function adminUpload() {
    if (!isAdmin()) {
      alert("You are not logged in as admin.");
      return;
    }
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = 'video/*';
    input.onchange = function(e) {
      const file = e.target.files[0];
      const url = URL.createObjectURL(file);
      videoData[currentFolder].push({ name: file.name, url: url });
      localStorage.setItem('videoData', JSON.stringify(videoData));
      renderVideoList();
    };
    input.click();
  }

  function shareApp() {
    const appLink = window.location.href;
    if (navigator.share) {
      navigator.share({
        title: 'Video Player App',
        text: 'Check out this Video Player App!',
        url: appLink
      }).catch(console.error);
    } else {
      navigator.clipboard.writeText(appLink).then(() => {
        alert("App link copied to clipboard:\n" + appLink);
      });
    }
  }

  window.onload = showAdminUI;
</script>

</body>
</html>

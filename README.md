<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Android TV Style IPTV Player</title>
    <link href="https://vjs.zencdn.net/7.20.3/video-js.css" rel="stylesheet" />
    <style>
        body {
            margin: 0;
            font-family: 'Roboto', sans-serif;
            background-color: #1a1a2e;
            color: #fff;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            height: 100vh;
            overflow: hidden;
            padding: 20px;
        }
        #container {
            width: 100%;
            max-width: 1280px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        #video-section {
            width: 100%;
            aspect-ratio: 16 / 9;
            position: relative;
            border-radius: 5px;
            overflow: hidden;
            background: #000;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
        }
        #video {
            width: 100%;
            height: 100%;
        }
        #controls {
            width: 100%;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        #category-select {
            padding: 10px;
            border-radius: 8px;
            border: none;
            outline: none;
            color: #fff;
            font-size: 16px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
            background: #53354a;
        }
        #channel-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            height: 430px;
            overflow-y: auto;
            padding: 10px;
            background: #16213e;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
            scroll-behavior: smooth;
        }
        .channel-card {
            background: #0f3460;
            border-radius: 12px;
            padding: 10px;
            text-align: center;
            transition: transform 0.3s, background 0.3s, opacity 0.3s;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
            opacity: 0;
            transform: translateY(20px);
        }
        .channel-card img {
            width: 100%;
            height: 80px;
            border-radius: 8px;
            object-fit: cover;
        }
        .channel-card:hover {
            background: #1c3a72;
            transform: translateY(-5px);
        }
        .playing {
            background: #e94560;
            border: 2px solid #ffffff;
            opacity: 1;
        }
        .playing img {
            opacity: 0.8;
        }
        .footer {
            margin-top: auto;
            padding: 20px;
            text-align: center;
        }
        .floating-button {
            background-color: #e94560;
            color: #fff;
            border: none;
            border-radius: 50px;
            padding: 15px 25px;
            font-size: 16px;
            cursor: pointer;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
            transition: background-color 0.3s, transform 0.3s;
        }
        .floating-button:hover {
            background-color: #f05945;
            transform: translateY(-5px) scale(1.05);
        }
        .error-message {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #ff4b5c;
            font-size: 18px;
            text-align: center;
        }
    </style>
</head>
<body>

<div id="container">
    <div id="video-section">
        <video id="video" class="video-js vjs-default-skin" controls preload="auto">
            <source id="video-source" src="" type="application/x-mpegURL">
            Your browser does not support the video tag.
        </video>
        <div id="error-message" class="error-message" style="display: none;">
            The channel is currently unavailable. Please contact the developer.
        </div>
    </div>
    <div id="controls">
        <select id="category-select">
            <option value="bangladesh">Bangladesh</option>
            <option value="islamic">Islamic</option>
            <option value="news">News</option>
            <option value="sports">Sports</option>
            <option value="movies">Kids</option>
        </select>
    </div>
    <div id="channel-container"></div>
</div>

<div class="footer">
    <button class="floating-button" onclick="window.location.href='https://yourwebsite.com'">Visit our apps store</button>
</div>

<script>
  const video = document.getElementById('video');
  const videoSource = document.getElementById('video-source');
  const errorMessage = document.getElementById('error-message');
  const categorySelect = document.getElementById('category-select');
  const channelContainer = document.getElementById('channel-container');

  const m3uUrls = {
    bangladesh: "https://raw.githubusercontent.com/sultanarabi161/bd-playlist/main/all-category.m3u",
    islamic: "https://raw.githubusercontent.com/sultanarabi161/Youtube-playlist/main/bd.m3u",
    news: "https://raw.githubusercontent.com/sultanarabi161/bd-playlist/main/all-category.m3u",
    sports: "https://example.com/sports.m3u",
    kids: "https://raw.githubusercontent.com/sultanarabi161/New-playlist-demo/main/demo.m3u"
  };

  async function fetchChannels(url) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error("Network response was not ok");
      const text = await response.text();
      parseM3U(text);
    } catch (error) {
      errorMessage.style.display = 'block';
      videoSource.src = '';
      video.load();
    }
  }

  function parseM3U(data) {
    const lines = data.split("\n");
    const channels = [];
    let currentChannel = {};

    lines.forEach(line => {
      line = line.trim();
      if (line.startsWith("#EXTINF")) {
        const info = line.match(/#EXTINF:-1.*,(.*)/);
        if (info) {
          currentChannel.name = info[1];
        }
        const logo = line.match(/tvg-logo="(.*?)"/);
        if (logo) {
          currentChannel.logo = logo[1];
        }
      } else if (line && !line.startsWith("#")) {
        currentChannel.url = line;
        channels.push(currentChannel);
        currentChannel = {};
      }
    });

    renderChannels(channels);
  }

  function renderChannels(channels) {
    channelContainer.innerHTML = '';
    errorMessage.style.display = 'none'; // Hide error message when channels are rendered

    channels.forEach((channel, index) => {
      const card = document.createElement('div');
      card.className = 'channel-card';
      card.dataset.url = channel.url;
      card.innerHTML = `<img src="${channel.logo || 'default-logo.png'}" alt="${channel.name}">`;
      card.style.transitionDelay = `${index * 0.1}s`; // Animation delay
      card.style.opacity = '1';
      card.style.transform = 'translateY(0)';
      card.addEventListener('click', () => {
        videoSource.src = channel.url;
        video.load();
        video.play();

        // সব চ্যানেল কার্ড থেকে 'playing' ক্লাস সরিয়ে দিন
        const allCards = document.querySelectorAll('.channel-card');
        allCards.forEach(card => card.classList.remove('playing'));

        // ক্লিককৃত চ্যানেল কার্ডে 'playing' ক্লাস যোগ করুন
        card.classList.add('playing');
      });
      channelContainer.appendChild(card);
    });
  }

  categorySelect.addEventListener('change', () => {
    const selectedCategory = categorySelect.value;
    const urlToFetch = m3uUrls[selectedCategory] || m3uUrls.bangladesh;
    fetchChannels(urlToFetch);
  });

  // Initial load
  fetchChannels(m3uUrls.bangladesh);
</script>

</body>
</html>

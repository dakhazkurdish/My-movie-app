<!doctype html>
<html lang="ku" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CineKurd - فلیمێن کامل</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-950 text-white min-h-screen font-sans">

  <header class="bg-slate-900 border-b border-slate-800 px-4 py-3 flex items-center justify-between sticky top-0 z-50">
    <div onclick="showhome()" class="text-2xl font-black text-blue-500 cursor-pointer">
      CINEKURD
    </div>
    <div class="flex items-center gap-2">
      <input 
        type="text" 
        id="searchInput" 
        placeholder="ناڤێ فلیمی ب ئینگلیزی..." 
        class="bg-slate-800 border border-slate-700 text-xs text-white rounded-lg px-3 py-1.5 focus:outline-none"
        onkeyup="if(event.key === 'Enter') searchMovies()"
      />
      <button onclick="searchMovies()" class="bg-blue-600 text-xs px-3 py-1.5 rounded-lg">گەڕان</button>
    </div>
  </header>

  <main class="p-4 max-w-6xl mx-auto w-full">

    <!-- بەشێ پلەیەر -->
    <div id="playersection" class="hidden space-y-4">
      <div class="flex items-center justify-between">
        <button onclick="showhome()" class="bg-slate-800 hover:bg-slate-700 text-xs px-3 py-2 rounded-lg">
          ← گەڕانەوە
        </button>
        <span id="serverstatus" class="text-xs text-green-400">سێرڤەری 1</span>
      </div>

      <!-- Iframe Player -->
      <div class="aspect-video w-full bg-black rounded-xl overflow-hidden border border-slate-800 shadow-2xl">
        <iframe 
          id="videoplayer" 
          class="w-full h-full border-0" 
          allowfullscreen 
          allow="autoplay; encrypted-media">
        </iframe>
      </div>

      <div class="flex items-center justify-between flex-wrap gap-2">
        <h2 id="detailtitle" class="text-lg font-bold text-blue-400"></h2>
        
        <!-- گۆڕینا سێرڤەران -->
        <div class="flex gap-2">
          <button onclick="switchServer(1)" class="bg-blue-600 text-xs px-3 py-1.5 rounded">سێرڤەری ١ (VidSrc)</button>
          <button onclick="switchServer(2)" class="bg-slate-800 text-xs px-3 py-1.5 rounded">سێرڤەری ٢ (2Embed)</button>
          <button onclick="switchServer(3)" class="bg-slate-800 text-xs px-3 py-1.5 rounded">سێرڤەری ٣ (Smashy)</button>
        </div>
      </div>
    </div>

    <!-- بەشێ سەرەکی -->
    <div id="homesection" class="space-y-4">
      <h2 class="text-md font-bold text-slate-300 border-b border-slate-800 pb-2">فلیمێن دیارکری</h2>
      <div id="moviesgrid" class="grid grid-cols-2 sm:grid-cols-4 gap-4"></div>
    </div>

  </main>

  <script>
    const movies = [
      { id: 27205, title: "Inception", img: "https://image.tmdb.org/t/p/w500/oYuLEt3zVCKq57qu2F8dT7NIa6f.jpg", rate: "8.4" },
      { id: 157336, title: "Interstellar", img: "https://image.tmdb.org/t/p/w500/gEU2QniE6E77NI6lCU6MxlNBvIx.jpg", rate: "8.4" },
      { id: 299536, title: "Avengers: Infinity War", img: "https://image.tmdb.org/t/p/w500/7WsyChQLEftFiDOVTGkv3hFpyyt.jpg", rate: "8.3" },
      { id: 155, title: "The Dark Knight", img: "https://image.tmdb.org/t/p/w500/qJ2tW6WMUDux911r6m7haRef0WH.jpg", rate: "8.5" },
      { id: 550, title: "Fight Club", img: "https://image.tmdb.org/t/p/w500/pB8BM7pdSp6B6Ih7QZ4DrQ3PmJK.jpg", rate: "8.4" },
      { id: 603, title: "The Matrix", img: "https://image.tmdb.org/t/p/w500/f89U3ADr1oiB1s9GkdPOEpXUk5H.jpg", rate: "8.2" }
    ];

    let currentId = null;

    window.onload = function() {
      displayMovies(movies);
    };

    function displayMovies(list) {
      const grid = document.getElementById("moviesgrid");
      grid.innerHTML = "";
      list.forEach(m => {
        grid.innerHTML += `
          <div onclick="playmovie(${m.id}, '${m.title}')" class="bg-slate-900 border border-slate-800 rounded-lg p-2 cursor-pointer hover:border-blue-500 transition">
            <img src="${m.img}" class="w-full aspect-[2/3] object-cover rounded mb-2" />
            <div class="text-xs font-bold truncate">${m.title}</div>
            <div class="text-xs text-yellow-400 mt-1">★ ${m.rate}</div>
          </div>
        `;
      });
    }

    function playmovie(id, title) {
      currentId = id;
      document.getElementById("homesection").classList.add("hidden");
      document.getElementById("playersection").classList.remove("hidden");
      document.getElementById("detailtitle").innerText = title;
      switchServer(1);
    }

    function switchServer(num) {
      if (!currentId) return;
      const player = document.getElementById("videoplayer");
      const status = document.getElementById("serverstatus");

      if (num === 1) {
        player.src = `https://vidsrc.cc/v2/embed/movie/${currentId}`;
        status.innerText = "سێرڤەری ١ (VidSrc.cc)";
      } else if (num === 2) {
        player.src = `https://www.2embed.cc/embed/${currentId}`;
        status.innerText = "سێرڤەری ٢ (2Embed)";
      } else {
        player.src = `https://embed.smashystream.com/playere.php?tmdb=${currentId}`;
        status.innerText = "سێرڤەری ٣ (Smashy)";
      }
    }

    function searchMovies() {
      const q = document.getElementById("searchInput").value.trim();
      if (!q) return;
      fetch(`https://api.themoviedb.org/3/search/movie?api_key=3fd2be6f0c70a2a598f084dd27548780&query=${encodeURIComponent(q)}`)
        .then(r => r.json())
        .then(d => {
          if (d.results && d.results.length) {
            const mapped = d.results.map(m => ({
              id: m.id,
              title: m.title,
              img: m.poster_path ? `https://image.tmdb.org/t/p/w500${m.poster_path}` : 'https://via.placeholder.com/500x750',
              rate: m.vote_average ? m.vote_average.toFixed(1) : "N/A"
            }));
            displayMovies(mapped);
          }
        });
    }

    function showhome() {
      document.getElementById("videoplayer").src = "";
      document.getElementById("playersection").classList.add("hidden");
      document.getElementById("homesection").classList.remove("hidden");
    }
  </script>
</body>
</html>

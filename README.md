<!doctype html>
<html lang="ku" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CineKurd - فلیمێن کامل</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-950 text-white min-h-screen font-sans">

  <!-- Header -->
  <header class="bg-slate-900 border-b border-slate-800 px-4 py-3 flex flex-wrap items-center justify-between sticky top-0 z-50 gap-3">
    <div onclick="showhome()" class="text-2xl font-black text-blue-500 cursor-pointer tracking-wider">
      CINEKURD
    </div>

    <!-- گەڕان -->
    <div class="flex items-center bg-slate-800 rounded-lg px-3 py-1.5 w-full sm:w-auto flex-1 max-w-md border border-slate-700">
      <input 
        type="text" 
        id="searchInput" 
        placeholder="ناڤێ فلیمەکێ ب نڤێسە (ب ئینگلیزی)..." 
        class="bg-transparent text-sm text-white focus:outline-none w-full px-2"
        onkeyup="if(event.key === 'Enter') searchMovies()"
      />
      <button onclick="searchMovies()" class="text-slate-400 hover:text-white text-sm font-bold px-2">
        🔍
      </button>
    </div>
  </header>

  <main class="p-4 max-w-7xl mx-auto w-full">

    <!-- بەشی پلەیەر -->
    <div id="playersection" class="hidden space-y-4">
      <div class="flex items-center justify-between">
        <button onclick="showhome()" class="bg-slate-800 hover:bg-slate-700 text-sm px-4 py-2 rounded-lg transition border border-slate-700">
          ← گەڕانەوە
        </button>
        <span id="serverstatus" class="text-xs text-green-400 bg-green-950/50 border border-green-800 px-3 py-1 rounded-full">سێرڤەری ١ (VidSrc)</span>
      </div>

      <!-- Player Frame -->
      <div class="aspect-video w-full bg-black rounded-xl overflow-hidden border border-slate-800 shadow-2xl">
        <iframe 
          id="videoplayer" 
          class="w-full h-full border-0" 
          allowfullscreen 
          referrerpolicy="origin"
          allow="autoplay; encrypted-media; picture-in-picture">
        </iframe>
      </div>

      <!-- Detail & Server Switch -->
      <div class="flex items-center justify-between flex-wrap gap-3 bg-slate-900 p-4 rounded-xl border border-slate-800">
        <h2 id="detailtitle" class="text-xl font-bold text-blue-400"></h2>
        
        <div class="flex items-center gap-2">
          <span class="text-xs text-slate-400">سێرڤەر:</span>
          <button onclick="switchServer(1)" class="bg-blue-600 text-xs px-3 py-1.5 rounded text-white font-medium">سێرڤەری ١</button>
          <button onclick="switchServer(2)" class="bg-slate-800 hover:bg-slate-700 text-xs px-3 py-1.5 rounded text-white font-medium">سێرڤەری ٢</button>
          <button onclick="switchServer(3)" class="bg-slate-800 hover:bg-slate-700 text-xs px-3 py-1.5 rounded text-white font-medium">سێرڤەری ٣</button>
        </div>
      </div>
    </div>

    <!-- بەشی سەرەکی -->
    <div id="homesection" class="space-y-6">
      <h2 id="sectionTitle" class="text-lg font-bold border-b border-slate-800 pb-2 text-blue-400">
        فلیمێن دەستنیشانکری (ب لێدانەکێ دێ ڤەبەن)
      </h2>
      
      <div id="moviesgrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4">
        <!-- لستا فلیمان -->
      </div>
    </div>

  </main>

  <script>
    // لستا فلیمێن سەرەکی دگەل IDێن فلیمان ل TMDB
    const initialMovies = [
      { id: 27205, title: "Inception", img: "https://image.tmdb.org/t/p/w500/oYuLEt3zVCKq57qu2F8dT7NIa6f.jpg", rate: "8.4" },
      { id: 157336, title: "Interstellar", img: "https://image.tmdb.org/t/p/w500/gEU2QniE6E77NI6lCU6MxlNBvIx.jpg", rate: "8.4" },
      { id: 299536, title: "Avengers: Infinity War", img: "https://image.tmdb.org/t/p/w500/7WsyChQLEftFiDOVTGkv3hFpyyt.jpg", rate: "8.3" },
      { id: 155, title: "The Dark Knight", img: "https://image.tmdb.org/t/p/w500/qJ2tW6WMUDux911r6m7haRef0WH.jpg", rate: "8.5" },
      { id: 550, title: "Fight Club", img: "https://image.tmdb.org/t/p/w500/pB8BM7pdSp6B6Ih7QZ4DrQ3PmJK.jpg", rate: "8.4" },
      { id: 603, title: "The Matrix", img: "https://image.tmdb.org/t/p/w500/f89U3ADr1oiB1s9GkdPOEpXUk5H.jpg", rate: "8.2" },
      { id: 671, title: "Harry Potter", img: "https://image.tmdb.org/t/p/w500/wuMc08IPKEatf9MStM12S3A3M2d.jpg", rate: "7.9" },
      { id: 98, title: "Gladiator", img: "https://image.tmdb.org/t/p/w500/ty8T31fs31411393.jpg", rate: "8.5" }
    ];

    let currentMovieId = null;

    window.onload = function() {
      displayMovies(initialMovies);
    };

    function displayMovies(list) {
      const grid = document.getElementById("moviesgrid");
      grid.innerHTML = "";

      list.forEach(m => {
        const poster = m.img || "https://via.placeholder.com/500x750?text=No+Cover";
        grid.innerHTML += `
          <div onclick="playmovie(${m.id}, '${m.title.replace(/'/g, "\\'")}')" class="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden cursor-pointer hover:border-blue-500 hover:scale-[1.02] transition">
            <img src="${poster}" class="w-full aspect-[2/3] object-cover" />
            <div class="p-3">
              <div class="text-sm font-bold truncate text-slate-100">${m.title}</div>
              <div class="text-xs text-yellow-400 mt-1">★ ${m.rate}</div>
            </div>
          </div>
        `;
      });
    }

    function playmovie(id, title) {
      currentMovieId = id;
      document.getElementById("homesection").classList.add("hidden");
      document.getElementById("playersection").classList.remove("hidden");
      document.getElementById("detailtitle").innerText = title;
      
      switchServer(1);
      window.scrollTo({ top: 0, behavior: "smooth" });
    }

    function switchServer(serverNum) {
      if (!currentMovieId) return;
      const player = document.getElementById("videoplayer");
      const status = document.getElementById("serverstatus");

      if (serverNum === 1) {
        // VidSrc Embed Player
        player.src = `https://vidsrc.to/embed/movie/${currentMovieId}`;
        status.innerText = "سێرڤەری ١ (VidSrc)";
      } else if (serverNum === 2) {
        // VidLink Player
        player.src = `https://vidlink.pro/movie/${currentMovieId}`;
        status.innerText = "سێرڤەری ٢ (VidLink)";
      } else {
        // Embed.su Player
        player.src = `https://embed.su/embed/movie/${currentMovieId}`;
        status.innerText = "سێرڤەری ٣ (EmbedSU)";
      }
    }

    // گەڕان ڕاستەوخۆ ب ڕێکا API
    function searchMovies() {
      const query = document.getElementById("searchInput").value.trim();
      if (!query) return;

      fetch(`https://api.themoviedb.org/3/search/movie?api_key=3fd2be6f0c70a2a598f084dd27548780&query=${encodeURIComponent(query)}`)
        .then(res => res.json())
        .then(data => {
          if (data.results && data.results.length > 0) {
            const mapped = data.results.map(m => ({
              id: m.id,
              title: m.title,
              img: m.poster_path ? `https://image.tmdb.org/t/p/w500${m.poster_path}` : null,
              rate: m.vote_average ? m.vote_average.toFixed(1) : "N/A"
            }));
            displayMovies(mapped);
          } else {
            alert("هیچ فلیمەک ب ڤی ناڤی نەهاتە پەیداکرن!");
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

<!DOCTYPE html>
<html lang="ku" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CineKurd</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-950 text-white min-h-screen font-sans">

  <!-- سەرپەڕ و سێرچ بار -->
  <header class="bg-slate-900 border-b border-slate-800 px-4 py-3 flex flex-wrap items-center justify-between gap-4 sticky top-0 z-50">
    <div onclick="showHome()" class="text-2xl font-black text-blue-500 cursor-pointer">
      CINEKURD
    </div>

    <div class="relative flex-1 max-w-md">
      <input
        type="text"
        id="searchInput"
        oninput="handleSearch(this.value)"
        placeholder="ل ناڤێ فلیم یان زنجیرەکێ بگەڕە..."
        class="w-full bg-slate-950 border border-slate-700 rounded-xl px-4 py-2 text-sm text-white placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <div id="searchResults" class="hidden absolute left-0 right-0 mt-2 bg-slate-900 border border-slate-800 rounded-xl shadow-2xl max-h-80 overflow-y-auto z-50 divide-y divide-slate-800"></div>
    </div>
  </header>

  <!-- ناڤەڕۆکا سەرەکی -->
  <main class="p-4 md:p-8 max-w-7xl mx-auto w-full">

    <!-- پشکا پلەیەرێ (ل دەستپێکێ یا ڤەشارتییە) -->
    <div id="playerSection" class="hidden space-y-6">
      <button onclick="showHome()" class="bg-slate-900 border border-slate-800 hover:bg-slate-800 text-sm px-4 py-2 rounded-xl transition">
        ← زڤرین بۆ لاپەڕێ سەرەکی
      </button>

      <div class="aspect-video w-full bg-black rounded-2xl overflow-hidden shadow-2xl border border-slate-800">
        <iframe id="videoPlayer" class="w-full h-full border-0" allowfullscreen></iframe>
      </div>

      <div id="tvControls" class="hidden bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-wrap items-center gap-4">
        <div class="flex items-center gap-2">
          <span class="text-sm text-slate-400">وەرز:</span>
          <select id="seasonSelect" onchange="changeEpisode()" class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-1.5 text-sm"></select>
        </div>
        <div class="flex items-center gap-2">
          <span class="text-sm text-slate-400">خەلەک:</span>
          <select id="episodeSelect" onchange="changeEpisode()" class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-1.5 text-sm"></select>
        </div>
      </div>

      <div class="bg-slate-900 border border-slate-800 rounded-2xl p-6 flex flex-col md:flex-row gap-6 items-start">
        <img id="detailPoster" class="w-36 h-52 object-cover rounded-xl shadow-lg border border-slate-800" src="" alt="" />
        <div class="space-y-3 flex-1 text-right">
          <h1 id="detailTitle" class="text-2xl font-bold"></h1>
          <div id="detailRating" class="text-sm text-yellow-400"></div>
          <p id="detailOverview" class="text-slate-300 text-sm leading-relaxed"></p>
        </div>
      </div>
    </div>

    <!-- لاپەڕێ سەرەکی -->
    <div id="homeSection" class="space-y-10 text-right">
      <section class="space-y-4">
        <div class="flex items-center gap-2 border-b border-slate-800 pb-2">
          <span class="w-2.5 h-6 bg-red-500 rounded-full inline-block"></span>
          <h2 class="text-xl font-bold">بەرهەمێن ترێند (Trending)</h2>
        </div>
        <div id="trendingGrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
          <p class="text-slate-500 text-sm">چاوەڕێ بە، دهێنە بارکرن...</p>
        </div>
      </section>

      <section class="space-y-4">
        <div class="flex items-center gap-2 border-b border-slate-800 pb-2">
          <span class="w-2.5 h-6 bg-blue-500 rounded-full inline-block"></span>
          <h2 class="text-xl font-bold">فلیمێن بەناوبانگ (Popular)</h2>
        </div>
        <div id="popularGrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
          <p class="text-slate-500 text-sm">چاوەڕێ بە، دهێنە بارکرن...</p>
        </div>
      </section>
    </div>

  </main>

  <script>
    const TMDB_KEY = "3fd2be8f0c70a2a598f084ddfb75487c";
    let currentId = null;
    let currentType = "movie";

    window.onload = function() {
      // ئامادەکرنا ژمارا وەرز و خەلەکان
      const sSelect = document.getElementById("seasonSelect");
      for(let i=1; i<=10; i++) sSelect.innerHTML += `<option value="${i}">وەرزێ ${i}</option>`;
      const epSelect = document.getElementById("episodeSelect");
      for(let i=1; i<=30; i++) epSelect.innerHTML += `<option value="${i}">خەلەکا ${i}</option>`;

      loadMovies("trending/movie/week", "trendingGrid");
      loadMovies("movie/popular", "popularGrid");
    };

    function loadMovies(endpoint, gridId) {
      fetch(`https://api.themoviedb.org/3/${endpoint}?api_key=${TMDB_KEY}`)
        .then(res => res.json())
        .then(data => {
          const grid = document.getElementById(gridId);
          grid.innerHTML = "";
          (data.results || []).slice(0, 12).forEach(item => {
            const poster = item.poster_path 
              ? `https://image.tmdb.org/t/p/w500${item.poster_path}` 
              : 'https://via.placeholder.com/500x750?text=No+Image';
            
            grid.innerHTML += `
              <div onclick='openMedia(${JSON.stringify(item).replace(/'/g, "&apos;")})' class="bg-slate-900 border border-slate-800 hover:border-blue-500 rounded-xl overflow-hidden cursor-pointer transition shadow-lg">
                <div class="relative aspect-[2/3] bg-slate-950">
                  <img src="${poster}" class="w-full h-full object-cover" />
                  <div class="absolute top-2 right-2 bg-slate-950/80 px-2 py-0.5 rounded text-[11px] font-bold text-yellow-400">
                    ★ ${(item.vote_average || 0).toFixed(1)}
                  </div>
                </div>
                <div class="p-2.5">
                  <h4 class="text-xs font-semibold truncate text-white">${item.title || item.name}</h4>
                </div>
              </div>
            `;
          });
        })
        .catch(err => {
          document.getElementById(gridId).innerHTML = "<p class='text-red-400 text-xs'>خەلەتیەک د بارکرنێ دا چێبوو!</p>";
        });
    }

    function handleSearch(query) {
      const resultsDiv = document.getElementById("searchResults");
      if (query.trim().length < 2) {
        resultsDiv.classList.add("hidden");
        return;
      }

      fetch(`https://api.themoviedb.org/3/search/multi?api_key=${TMDB_KEY}&query=${encodeURIComponent(query)}`)
        .then(res => res.json())
        .then(data => {
          resultsDiv.innerHTML = "";
          const items = (data.results || []).filter(x => x.poster_path).slice(0, 6);
          if (items.length > 0) {
            resultsDiv.classList.remove("hidden");
            items.forEach(item => {
              resultsDiv.innerHTML += `
                <div onclick='openMedia(${JSON.stringify(item).replace(/'/g, "&apos;")}); document.getElementById("searchResults").classList.add("hidden");' class="p-3 flex items-center gap-3 hover:bg-slate-800 cursor-pointer text-right">
                  <img src="https://image.tmdb.org/t/p/w92${item.poster_path}" class="w-10 h-14 object-cover rounded bg-slate-950" />
                  <div class="flex-1 min-w-0">
                    <div class="text-sm font-semibold truncate">${item.title || item.name}</div>
                    <div class="text-xs text-slate-400 mt-1">★ ${(item.vote_average || 0).toFixed(1)}</div>
                  </div>
                </div>
              `;
            });
          } else {
            resultsDiv.classList.add("hidden");
          }
        });
    }

    function openMedia(item) {
      currentId = item.id;
      currentType = item.media_type || (item.title ? "movie" : "tv");

      document.getElementById("homeSection").classList.add("hidden");
      document.getElementById("playerSection").classList.remove("hidden");

      document.getElementById("detailTitle").innerText = item.title || item.name;
      document.getElementById("detailRating").innerText = `★ ${(item.vote_average || 0).toFixed(1)}`;
      document.getElementById("detailOverview").innerText = item.overview || "چ زانیاری بەردەست نینن.";
      document.getElementById("detailPoster").src = item.poster_path 
        ? `https://image.tmdb.org/t/p/w500${item.poster_path}` 
        : '';

      const tvControls = document.getElementById("tvControls");
      if (currentType === "tv") {
        tvControls.classList.remove("hidden");
        changeEpisode();
      } else {
        tvControls.classList.add("hidden");
        document.getElementById("videoPlayer").src = `https://vidsrc.to/embed/movie/${currentId}`;
      }

      window.scrollTo({ top: 0, behavior: "smooth" });
    }

    function changeEpisode() {
      const season = document.getElementById("seasonSelect").value;
      const episode = document.getElementById("episodeSelect").value;
      document.getElementById("videoPlayer").src = `https://vidsrc.to/embed/tv/${currentId}/${season}/${episode}`;
    }

    function showHome() {
      document.getElementById("videoPlayer").src = "";
      document.getElementById("playerSection").classList.add("hidden");
      document.getElementById("homeSection").classList.remove("hidden");
    }
  </script>
</body>
</html>

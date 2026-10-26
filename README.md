<!doctype html>
<html lang="ku" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CineKurd - سەرجەم فلیمێن جیهانێ</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-950 text-white min-h-screen font-sans">

  <!-- سەرپەڕی (Header) -->
  <header class="bg-slate-900 border-b border-slate-800 px-4 py-3 flex flex-wrap items-center justify-between sticky top-0 z-50 gap-3">
    <div onclick="showhome()" class="text-2xl font-black text-blue-500 cursor-pointer tracking-wider">
      CINEKURD
    </div>

    <!-- بەشێ گەڕانێ -->
    <div class="flex items-center bg-slate-800 rounded-lg px-3 py-1.5 w-full sm:w-auto flex-1 max-w-md border border-slate-700 focus-within:border-blue-500">
      <input 
        type="text" 
        id="searchInput" 
        placeholder="ناڤێ فلیمەکێ ب نڤێسە..." 
        class="bg-transparent text-sm text-white focus:outline-none w-full px-2"
        onkeyup="if(event.key === 'Enter') searchMovies()"
      />
      <button onclick="searchMovies()" class="text-slate-400 hover:text-white text-sm font-bold px-2">
        🔍
      </button>
    </div>

    <div class="text-xs text-slate-400 hidden md:block">فلیم و زنجیرەیێن جیهانی</div>
  </header>

  <main class="p-4 max-w-7xl mx-auto w-full">

    <!-- بەشی پلەیەر -->
    <div id="playersection" class="hidden space-y-4">
      <div class="flex items-center justify-between">
        <button onclick="showhome()" class="bg-slate-800 hover:bg-slate-700 text-sm px-4 py-2 rounded-lg transition border border-slate-700 flex items-center gap-2">
          <span>←</span> گەڕانەوە بۆ سەرپەڕە
        </button>
        <span id="serverstatus" class="text-xs text-green-400 bg-green-950/50 border border-green-800 px-3 py-1 rounded-full">سێرڤەری ١ (VidLink)</span>
      </div>

      <div class="aspect-video w-full bg-black rounded-xl overflow-hidden border border-slate-800 shadow-2xl">
        <iframe 
          id="videoplayer" 
          class="w-full h-full border-0" 
          allowfullscreen 
          allow="autoplay; encrypted-media; picture-in-picture">
        </iframe>
      </div>

      <div class="flex items-center justify-between flex-wrap gap-3 bg-slate-900 p-4 rounded-xl border border-slate-800">
        <h2 id="detailtitle" class="text-xl font-bold text-blue-400"></h2>
        
        <!-- گۆڕینی سێرڤەر -->
        <div class="flex items-center gap-2">
          <span class="text-xs text-slate-400 hidden sm:inline">گۆڕینا سێرڤەری:</span>
          <button id="btn-srv1" onclick="switchServer(1)" class="bg-blue-600 text-xs px-3 py-1.5 rounded-lg text-white font-medium transition">سێرڤەری ١ (VidLink)</button>
          <button id="btn-srv2" onclick="switchServer(2)" class="bg-slate-800 hover:bg-slate-700 text-xs px-3 py-1.5 rounded-lg text-white font-medium transition">سێرڤەری ٢ (EmbedSU)</button>
        </div>
      </div>
    </div>

    <!-- بەشی سەرەکی -->
    <div id="homesection" class="space-y-6">
      
      <!-- بەشێ فلترکرنێ ب پێی جۆری (Categories) -->
      <div class="flex items-center gap-2 overflow-x-auto pb-2 scrollbar-none text-xs">
        <button onclick="filterCategory('popular', this)" class="category-btn bg-blue-600 text-white px-4 py-2 rounded-full whitespace-nowrap font-medium">بەربەلاڤترین (Popular)</button>
        <button onclick="filterCategory('top_rated', this)" class="category-btn bg-slate-800 hover:bg-slate-700 text-slate-300 px-4 py-2 rounded-full whitespace-nowrap font-medium">پلا بەرزترین (Top Rated)</button>
        <button onclick="filterGenre(28, this)" class="category-btn bg-slate-800 hover:bg-slate-700 text-slate-300 px-4 py-2 rounded-full whitespace-nowrap font-medium">ئەکشن (Action)</button>
        <button onclick="filterGenre(35, this)" class="category-btn bg-slate-800 hover:bg-slate-700 text-slate-300 px-4 py-2 rounded-full whitespace-nowrap font-medium">کۆمیدی (Comedy)</button>
        <button onclick="filterGenre(27, this)" class="category-btn bg-slate-800 hover:bg-slate-700 text-slate-300 px-4 py-2 rounded-full whitespace-nowrap font-medium">ترسناک (Horror)</button>
        <button onclick="filterGenre(878, this)" class="category-btn bg-slate-800 hover:bg-slate-700 text-slate-300 px-4 py-2 rounded-full whitespace-nowrap font-medium">زانسستی-خەیاڵی (Sci-Fi)</button>
      </div>

      <div>
        <h2 id="sectionTitle" class="text-lg font-bold border-b border-slate-800 pb-2 mb-4 text-blue-400 flex items-center justify-between">
          <span>فلیمێن دیارکری</span>
          <span id="pageIndicator" class="text-xs text-slate-500 font-normal">لاپەڕە 1</span>
        </h2>
        
        <div id="moviesgrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4">
          <!-- فلیم ل ڤێرە ب شێوەکێ ئۆتۆماتیکی دهێنە بارکرن -->
        </div>
      </div>

      <!-- دوگمێن لاپەڕان (Pagination) -->
      <div class="flex justify-center items-center gap-4 pt-4 border-t border-slate-800">
        <button id="prevBtn" onclick="changePage(-1)" class="bg-slate-800 hover:bg-slate-700 text-xs px-4 py-2 rounded-lg transition disabled:opacity-50 disabled:cursor-not-allowed">
          ← لاپەڕێ پێشتر
        </button>
        <button id="nextBtn" onclick="changePage(1)" class="bg-slate-800 hover:bg-slate-700 text-xs px-4 py-2 rounded-lg transition">
          لاپەڕێ پاشتر →
        </button>
      </div>

    </div>

  </main>

  <script>
    const API_KEY = "3fd2be6f0c70a2a598f084dd27548780"; // TMDB Key
    const BASE_URL = "https://api.themoviedb.org/3";
    const IMG_URL = "https://image.tmdb.org/t/p/w500";

    let currentMovieId = null;
    let currentPage = 1;
    let currentEndpoint = "/movie/popular";
    let isSearchMode = false;
    let currentSearchQuery = "";

    window.onload = function() {
      fetchMovies(currentEndpoint);
    };

    // بارکرنا فلیمان ژ API
    function fetchMovies(endpoint, isGenre = false, genreId = null) {
      let url = "";
      if (isSearchMode) {
        url = `${BASE_URL}/search/movie?api_key=${API_KEY}&query=${encodeURIComponent(currentSearchQuery)}&page=${currentPage}&language=en-US`;
      } else if (isGenre) {
        url = `${BASE_URL}/discover/movie?api_key=${API_KEY}&with_genres=${genreId}&page=${currentPage}&language=en-US`;
      } else {
        url = `${BASE_URL}${endpoint}?api_key=${API_KEY}&page=${currentPage}&language=en-US`;
      }

      document.getElementById("moviesgrid").innerHTML = `
        <div class="col-span-full text-center py-12 text-slate-500">
          د ئامادەکرنێ دایە...
        </div>
      `;

      fetch(url)
        .then(res => res.json())
        .then(data => {
          displayMovies(data.results);
          document.getElementById("pageIndicator").innerText = `لاپەڕە ${currentPage}`;
          document.getElementById("prevBtn").disabled = currentPage === 1;
        })
        .catch(err => {
          document.getElementById("moviesgrid").innerHTML = `
            <div class="col-span-full text-center py-12 text-red-500">
              خەلەتییەک هەیە د بارکرنا داتایان دا!
            </div>
          `;
        });
    }

    // نیشاندانا فلیمان د لاپەڕەی دا
    function displayMovies(movies) {
      const grid = document.getElementById("moviesgrid");
      grid.innerHTML = "";

      if (!movies || movies.length === 0) {
        grid.innerHTML = '<div class="col-span-full text-center py-12 text-slate-400">هیچ فلیمەک نەهاتە پەیداکرن.</div>';
        return;
      }

      movies.forEach(m => {
        const poster = m.poster_path ? (IMG_URL + m.poster_path) : "https://via.placeholder.com/500x750?text=No+Cover";
        const rating = m.vote_average ? m.vote_average.toFixed(1) : "N/A";
        const title = m.title.replace(/'/g, "\\'");

        grid.innerHTML += `
          <div onclick="playmovie(${m.id}, '${title}')" class="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden cursor-pointer hover:border-blue-500 hover:scale-[1.02] transition-all duration-200 group flex flex-col justify-between">
            <div>
              <div class="relative overflow-hidden">
                <img src="${poster}" alt="${m.title}" class="w-full aspect-[2/3] object-cover group-hover:opacity-90 transition" loading="lazy" />
                <span class="absolute top-2 right-2 bg-slate-950/80 text-yellow-400 text-xs font-bold px-2 py-1 rounded-md border border-slate-700 backdrop-blur-sm">
                  ★ ${rating}
                </span>
              </div>
              <div class="p-3">
                <div class="text-sm font-bold truncate text-slate-100 group-hover:text-blue-400 transition">${m.title}</div>
                <div class="text-xs text-slate-500 mt-1">${m.release_date ? m.release_date.split('-')[0] : 'N/A'}</div>
              </div>
            </div>
          </div>
        `;
      });
    }

    // لێدانا فلیمی
    function playmovie(id, title) {
      currentMovieId = id;
      document.getElementById("homesection").classList.add("hidden");
      document.getElementById("playersection").classList.remove("hidden");
      document.getElementById("detailtitle").innerText = title;
      
      switchServer(1);
      window.scrollTo({ top: 0, behavior: "smooth" });
    }

    // گۆڕینا سێرڤەری
    function switchServer(serverNum) {
      if (!currentMovieId) return;
      const player = document.getElementById("videoplayer");
      const btn1 = document.getElementById("btn-srv1");
      const btn2 = document.getElementById("btn-srv2");
      const status = document.getElementById("serverstatus");

      if (serverNum === 1) {
        player.src = `https://vidlink.pro/movie/${currentMovieId}`;
        btn1.className = "bg-blue-600 text-xs px-3 py-1.5 rounded-lg text-white font-medium transition";
        btn2.className = "bg-slate-800 hover:bg-slate-700 text-xs px-3 py-1.5 rounded-lg text-white font-medium transition";
        status.innerText = "سێرڤەری ١ (VidLink)";
      } else {
        player.src = `https://embed.su/embed/movie/${currentMovieId}`;
        btn2.className = "bg-blue-600 text-xs px-3 py-1.5 rounded-lg text-white font-medium transition";
        btn1.className = "bg-slate-800 hover:bg-slate-700 text-xs px-3 py-1.5 rounded-lg text-white font-medium transition";
        status.innerText = "سێرڤەری ٢ (EmbedSU)";
      }
    }

    // گەڕان
    function searchMovies() {
      const query = document.getElementById("searchInput").value.trim();
      if (!query) return;

      isSearchMode = true;
      currentSearchQuery = query;
      currentPage = 1;
      document.getElementById("sectionTitle").children[0].innerText = `ئەنجامێن گەڕانێ بۆ: "${query}"`;
      fetchMovies();
    }

    // گۆڕینا بەشان (Categories)
    function filterCategory(type, btn) {
      setActiveButton(btn);
      isSearchMode = false;
      currentPage = 1;
      currentEndpoint = `/movie/${type}`;
      document.getElementById("sectionTitle").children[0].innerText = btn.innerText;
      fetchMovies(currentEndpoint);
    }

    // گۆڕینا جۆران (Genres)
    function filterGenre(genreId, btn) {
      setActiveButton(btn);
      isSearchMode = false;
      currentPage = 1;
      document.getElementById("sectionTitle").children[0].innerText = btn.innerText;
      fetchMovies('', true, genreId);
    }

    function setActiveButton(activeBtn) {
      document.querySelectorAll('.category-btn').forEach(btn => {
        btn.className = "category-btn bg-slate-800 hover:bg-slate-700 text-slate-300 px-4 py-2 rounded-full whitespace-nowrap font-medium";
      });
      activeBtn.className = "category-btn bg-blue-600 text-white px-4 py-2 rounded-full whitespace-nowrap font-medium";
    }

    // گۆڕینا لاپەڕەی
    function changePage(direction) {
      currentPage += direction;
      if (currentPage < 1) currentPage = 1;
      fetchMovies(currentEndpoint);
      window.scrollTo({ top: 0, behavior: "smooth" });
    }

    // گەڕانەوە بۆ لاپەڕێ سەرەکی
    function showhome() {
      document.getElementById("videoplayer").src = "";
      document.getElementById("playersection").classList.add("hidden");
      document.getElementById("homesection").classList.remove("hidden");
    }
  </script>
</body>
</html>

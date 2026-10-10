<!doctype html>
<html lang="ku" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CineKurd</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-950 text-white min-h-screen font-sans">

  <header class="bg-slate-900 border-b border-slate-800 px-4 py-3 flex items-center justify-between sticky top-0 z-50">
    <div onclick="showhome()" class="text-2xl font-black text-blue-500 cursor-pointer">
      CINEKURD
    </div>
    <div class="text-xs text-slate-400">فلیم و زنجیرە</div>
  </header>

  <main class="p-4 max-w-6xl mx-auto w-full">

    <!-- بەشی پلەیەر -->
    <div id="playersection" class="hidden space-y-4">
      <div class="flex items-center justify-between">
        <button onclick="showhome()" class="bg-slate-800 hover:bg-slate-700 text-sm px-4 py-2 rounded-lg transition">
          ← گەڕانەوە بۆ سەرپەڕە
        </button>
        <span class="text-xs text-green-400">سێرڤەری بێ ڕیکلام (VidLink)</span>
      </div>

      <div class="aspect-video w-full bg-black rounded-xl overflow-hidden border border-slate-800 shadow-2xl">
        <iframe 
          id="videoplayer" 
          class="w-full h-full border-0" 
          allowfullscreen 
          allow="autoplay; encrypted-media; picture-in-picture">
        </iframe>
      </div>

      <div class="flex items-center justify-between flex-wrap gap-2">
        <h2 id="detailtitle" class="text-xl font-bold text-blue-400"></h2>
        
        <!-- گۆڕینی سێرڤەر -->
        <div class="flex gap-2">
          <button onclick="switchServer(1)" class="bg-blue-600 hover:bg-blue-700 text-xs px-3 py-1.5 rounded text-white">سێرڤەری ١ (VidLink)</button>
          <button onclick="switchServer(2)" class="bg-slate-800 hover:bg-slate-700 text-xs px-3 py-1.5 rounded text-white">سێرڤەری ٢ (EmbedSU)</button>
        </div>
      </div>
    </div>

    <!-- بەشی سەرەکی -->
    <div id="homesection" class="space-y-8 text-right">
      <div>
        <h2 class="text-lg font-bold border-b border-slate-800 pb-2 mb-4 text-red-400">بەرهەمەکان (Trending)</h2>
        <div id="moviesgrid" class="grid grid-cols-2 sm:grid-cols-4 gap-4"></div>
      </div>
    </div>

  </main>

  <script>
    var currentMovieId = null;

    var movies = [
      { id: 27205, title: "Inception", img: "https://image.tmdb.org/t/p/w500/oYuLEt3zVCKq57qu2F8dT7NIa6f.jpg", rate: "8.4" },
      { id: 157336, title: "Interstellar", img: "https://image.tmdb.org/t/p/w500/gEU2QniE6E77NI6lCU6MxlNBvIx.jpg", rate: "8.4" },
      { id: 299536, title: "Avengers: Infinity War", img: "https://image.tmdb.org/t/p/w500/7WsyChQLEftFiDOVTGkv3hFpyyt.jpg", rate: "8.3" },
      { id: 155, title: "The Dark Knight", img: "https://image.tmdb.org/t/p/w500/qJ2tW6WMUDux911r6m7haRef0WH.jpg", rate: "8.5" },
      { id: 550, title: "Fight Club", img: "https://image.tmdb.org/t/p/w500/pB8BM7pdSp6B6Ih7QZ4DrQ3PmJK.jpg", rate: "8.4" },
      { id: 603, title: "The Matrix", img: "https://image.tmdb.org/t/p/w500/f89U3ADr1oiB1s9GkdPOEpXUk5H.jpg", rate: "8.2" }
    ];

    window.onload = function() {
      var grid = document.getElementById("moviesgrid");
      grid.innerHTML = "";
      for (var i = 0; i < movies.length; i++) {
        var m = movies[i];
        grid.innerHTML += 
          '<div onclick="playmovie(' + m.id + ', \'' + m.title + '\')" class="bg-slate-900 border border-slate-800 rounded-lg overflow-hidden cursor-pointer p-2 hover:border-blue-500 transition">' +
            '<img src="' + m.img + '" class="w-full aspect-[2/3] object-cover rounded mb-2" />' +
            '<div class="text-xs font-bold truncate">' + m.title + '</div>' +
            '<div class="text-xs text-yellow-400 mt-1">★ ' + m.rate + '</div>' +
          '</div>';
      }
    };

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
      var player = document.getElementById("videoplayer");
      if (serverNum === 1) {
        // VidLink سێرڤەرێکی زۆر خاوێنە و پەنجەرەی زیادە ناکاتەوە
        player.src = "https://vidlink.pro/movie/" + currentMovieId;
      } else {
        // Embed.su جێگرەوەی دووەم
        player.src = "https://embed.su/embed/movie/" + currentMovieId;
      }
    }

    function showhome() {
      document.getElementById("videoplayer").src = "";
      document.getElementById("playersection").classList.add("hidden");
      document.getElementById("homesection").classList.remove("hidden");
    }
  </script>
</body>
</html>

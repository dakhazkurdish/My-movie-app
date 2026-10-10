<!doctype html>
<html lang="ku" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>cinekurd</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-slate-950 text-white min-h-screen font-sans">

  <!-- سەرپەڕ و سێرچ بار -->
  <header class="bg-slate-900 border-b border-slate-800 px-4 py-3 flex flex-wrap items-center justify-between gap-4 sticky top-0 z-50">
    <div onclick="showhome()" class="text-2xl font-black text-blue-500 cursor-pointer">
      cinekurd
    </div>

    <div class="relative flex-1 max-w-md">
      <input
        type="text"
        id="searchinput"
        oninput="handlesearch(this.value)"
        placeholder="ل ناڤێ فلیم یان زنجیرەکێ بگەڕە..."
        class="w-full bg-slate-950 border border-slate-700 rounded-xl px-4 py-2 text-sm text-white placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <div id="searchresults" class="hidden absolute left-0 right-0 mt-2 bg-slate-900 border border-slate-800 rounded-xl shadow-2xl max-h-80 overflow-y-auto z-50 divide-y divide-slate-800"></div>
    </div>
  </header>

  <!-- ناڤەڕۆک -->
  <main class="p-4 md:p-8 max-w-7xl mx-auto w-full">

    <!-- پشکا پلەیەرێ -->
    <div id="playersection" class="hidden space-y-6">
      <button onclick="showhome()" class="bg-slate-900 border border-slate-800 hover:bg-slate-800 text-sm px-4 py-2 rounded-xl transition">
        ← زڤرین بۆ سەرپەڕی
      </button>

      <div class="aspect-video w-full bg-black rounded-2xl overflow-hidden shadow-2xl border border-slate-800">
        <iframe id="videoplayer" class="w-full h-full border-0" allowfullscreen></iframe>
      </div>

      <div id="tvcontrols" class="hidden bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-wrap items-center gap-4">
        <div class="flex items-center gap-2">
          <span class="text-sm text-slate-400">وەرز:</span>
          <select id="seasonselect" onchange="changeepisode()" class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-1.5 text-sm"></select>
        </div>
        <div class="flex items-center gap-2">
          <span class="text-sm text-slate-400">خەلەک:</span>
          <select id="episodeselect" onchange="changeepisode()" class="bg-slate-950 border border-slate-700 rounded-lg px-3 py-1.5 text-sm"></select>
        </div>
      </div>

      <div class="bg-slate-900 border border-slate-800 rounded-2xl p-6 flex flex-col md:flex-row gap-6 items-start">
        <img id="detailposter" class="w-36 h-52 object-cover rounded-xl shadow-lg border border-slate-800" src="" alt="" />
        <div class="space-y-3 flex-1 text-right">
          <h1 id="detailtitle" class="text-2xl font-bold"></h1>
          <div id="detailrating" class="text-sm text-yellow-400"></div>
          <p id="detailoverview" class="text-slate-300 text-sm leading-relaxed"></p>
        </div>
      </div>
    </div>

    <!-- لاپەڕێ سەرەکی -->
    <div id="homesection" class="space-y-10 text-right">
      <section class="space-y-4">
        <div class="flex items-center gap-2 border-b border-slate-800 pb-2">
          <span class="w-2.5 h-6 bg-red-500 rounded-full inline-block"></span>
          <h2 class="text-xl font-bold">بەرهەمێن ترێند (trending)</h2>
        </div>
        <div id="trendinggrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
          <p class="text-slate-500 text-sm">چاوەڕێ بە...</p>
        </div>
      </section>

      <section class="space-y-4">
        <div class="flex items-center gap-2 border-b border-slate-800 pb-2">
          <span class="w-2.5 h-6 bg-blue-500 rounded-full inline-block"></span>
          <h2 class="text-xl font-bold">فلیمێن بەناوبانگ (popular)</h2>
        </div>
        <div id="populargrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
          <p class="text-slate-500 text-sm">چاوەڕێ بە...</p>
        </div>
      </section>
    </div>

  </main>

  <script>
    var currentid = null;
    var currenttype = "movie";

    window.onload = function() {
      var sselect = document.getElementById("seasonselect");
      for (var i = 1; i <= 10; i++) {
        sselect.innerHTML += '<option value="' + i + '">وەرزێ ' + i + '</option>';
      }
      var epselect = document.getElementById("episodeselect");
      for (var j = 1; j <= 30; j++) {
        epselect.innerHTML += '<option value="' + j + '">خەلەکا ' + j + '</option>';
      }

      loadmovies("trending/movie/week", "trendinggrid");
      loadmovies("movie/popular", "populargrid");
    };

    function loadmovies(endpoint, gridid) {
      var url = "https://api.themoviedb.org/3/" + endpoint + "?api_key=3fd2be8f0c70a2a598f084ddfb75487c";
      fetch(url)
        .then(function(res) { return res.json(); })
        .then(function(data) {
          var grid = document.getElementById(gridid);
          grid.innerHTML = "";
          var items = data.results || [];
          for (var i = 0; i < Math.min(items.length, 12); i++) {
            var item = items[i];
            var poster = item.poster_path 
              ? "https://image.tmdb.org/t/p/w500" + item.poster_path 
              : "https://via.placeholder.com/500x750?text=no+image";
            var rating = item.vote_average ? item.vote_average.toFixed(1) : "0.0";
            var title = item.title || item.name || "";
            var safeitem = JSON.stringify(item).replace(/'/g, "&apos;");

            grid.innerHTML += 
              '<div onclick=\'openmedia(' + safeitem + ')\' class="bg-slate-900 border border-slate-800 hover:border-blue-500 rounded-xl overflow-hidden cursor-pointer transition shadow-lg">' +
                '<div class="relative aspect-[2/3] bg-slate-950">' +
                  '<img src="' + poster + '" class="w-full h-full object-cover" />' +
                  '<div class="absolute top-2 right-2 bg-slate-950/80 px-2 py-0.5 rounded text-[11px] font-bold text-yellow-400">★ ' + rating + '</div>' +
                '</div>' +
                '<div class="p-2.5">' +
                  '<h4 class="text-xs font-semibold truncate text-white">' + title + '</h4>' +
                '</div>' +
              '</div>';
          }
        })
        .catch(function(err) {
          document.getElementById(gridid).innerHTML = "<p class='text-red-400 text-xs'>خەلەتەک چێبوو د بارکرنێ دا!</p>";
        });
    }

    function handlesearch(query) {
      var resultsdiv = document.getElementById("searchresults");
      if (query.trim().length < 2) {
        resultsdiv.classList.add("hidden");
        return;
      }

      var url = "https://api.themoviedb.org/3/search/multi?api_key=3fd2be8f0c70a2a598f084ddfb75487c&query=" + encodeURIComponent(query);
      fetch(url)
        .then(function(res) { return res.json(); })
        .then(function(data) {
          resultsdiv.innerHTML = "";
          var items = (data.results || []).filter(function(x) { return x.poster_path; }).slice(0, 6);
          if (items.length > 0) {
            resultsdiv.classList.remove("hidden");
            for (var i = 0; i < items.length; i++) {
              var item = items[i];
              var safeitem = JSON.stringify(item).replace(/'/g, "&apos;");
              var title = item.title || item.name;
              var rating = item.vote_average ? item.vote_average.toFixed(1) : "0.0";
              resultsdiv.innerHTML += 
                '<div onclick=\'openmedia(' + safeitem + '); document.getElementById("searchresults").classList.add("hidden");\' class="p-3 flex items-center gap-3 hover:bg-slate-800 cursor-pointer text-right">' +
                  '<img src="https://image.tmdb.org/t/p/w92' + item.poster_path + '" class="w-10 h-14 object-cover rounded bg-slate-950" />' +
                  '<div class="flex-1 min-w-0">' +
                    '<div class="text-sm font-semibold truncate">' + title + '</div>' +
                    '<div class="text-xs text-slate-400 mt-1">★ ' + rating + '</div>' +
                  '</div>' +
                '</div>';
            }
          } else {
            resultsdiv.classList.add("hidden");
          }
        });
    }

    function openmedia(item) {
      currentid = item.id;
      currenttype = item.media_type || (item.title ? "movie" : "tv");

      document.getElementById("homesection").classList.add("hidden");
      document.getElementById("playersection").classList.remove("hidden");

      document.getElementById("detailtitle").innerText = item.title || item.name;
      document.getElementById("detailrating").innerText = "★ " + (item.vote_average ? item.vote_average.toFixed(1) : "0.0");
      document.getElementById("detailoverview").innerText = item.overview || "چ زانیاری بەردەست نینن.";
      document.getElementById("detailposter").src = item.poster_path 
        ? "https://image.tmdb.org/t/p/w500" + item.poster_path 
        : "";

      var tvcontrols = document.getElementById("tvcontrols");
      if (currenttype === "tv") {
        tvcontrols.classList.remove("hidden");
        changeepisode();
      } else {
        tvcontrols.classList.add("hidden");
        document.getElementById("videoplayer").src = "https://vidsrc.to/embed/movie/" + currentid;
      }

      window.scrollTo({ top: 0, behavior: "smooth" });
    }

    function changeepisode() {
      var season = document.getElementById("seasonselect").value;
      var episode = document.getElementById("episodeselect").value;
      document.getElementById("videoplayer").src = "https://vidsrc.to/embed/tv/" + currentid + "/" + season + "/" + episode;
    }

    function showhome() {
      document.getElementById("videoplayer").src = "";
      document.getElementById("playersection").classList.add("hidden");
      document.getElementById("homesection").classList.remove("hidden");
    }
  </script>
</body>
</html>

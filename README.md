<!DOCTYPE html>
<html lang="ku" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ماڵپەڕێ فلیم و زنجیرەیان</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- React & ReactDOM CDN -->
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <!-- Babel CDN -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body class="bg-slate-950 text-white min-h-screen">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect } = React;

    // کلیلا فەرمی یا TMDB
    const TMDB_API_KEY = "3fd2be8f0c70a2a598f084ddfb75487c";

    function App() {
      const [view, setView] = useState("home"); // "home" | "watch"
      const [mediaType, setMediaType] = useState("movie"); // "movie" | "tv"
      const [selectedItem, setSelectedItem] = useState(null);
      const [season, setSeason] = useState(1);
      const [episode, setEpisode] = useState(1);
      
      const [trending, setTrending] = useState([]);
      const [popular, setPopular] = useState([]);
      const [searchQuery, setSearchQuery] = useState("");
      const [searchResults, setSearchResults] = useState([]);
      const [loading, setLoading] = useState(true);

      // بارکرنا داتایا دەستپێکێ ژ TMDB
      useEffect(() => {
        async function fetchInitialData() {
          try {
            setLoading(true);
            const [trendRes, popRes] = await Promise.all([
              fetch(`https://api.themoviedb.org/3/trending/all/week?api_key=${TMDB_API_KEY}&language=en-US`),
              fetch(`https://api.themoviedb.org/3/movie/popular?api_key=${TMDB_API_KEY}&language=en-US`)
            ]);
            const trendData = await trendRes.json();
            const popData = await popRes.json();
            setTrending(trendData.results || []);
            setPopular(popData.results || []);
          } catch (err) {
            console.error("Error loading movies:", err);
          } finally {
            setLoading(false);
          }
        }
        fetchInitialData();
      }, []);

      // گەڕیان ل ناڤ داتابەیسێ TMDB
      const handleSearch = async (e) => {
        const q = e.target.value;
        setSearchQuery(q);
        if (q.trim().length > 1) {
          try {
            const res = await fetch(`https://api.themoviedb.org/3/search/multi?api_key=${TMDB_API_KEY}&query=${encodeURIComponent(q)}&language=en-US`);
            const data = await res.json();
            setSearchResults(data.results ? data.results.filter(x => x.poster_path) : []);
          } catch (err) {
            console.error(err);
          }
        } else {
          setSearchResults([]);
        }
      };

      const openPlayer = (item) => {
        const type = item.media_type || (item.title ? "movie" : "tv");
        setMediaType(type);
        setSelectedItem(item);
        setSeason(1);
        setEpisode(1);
        setView("watch");
        window.scrollTo({ top: 0, behavior: "smooth" });
      };

      return (
        <div className="min-h-screen flex flex-col font-sans">
          {/* سەرپەڕ (Header & Navbar) */}
          <header className="bg-slate-900/90 backdrop-blur-md sticky top-0 z-50 border-b border-slate-800 px-4 py-3 flex flex-wrap items-center justify-between gap-4">
            <div 
              onClick={() => { setView("home"); setSelectedItem(null); setSearchQuery(""); }} 
              className="text-2xl font-black bg-gradient-to-r from-blue-400 to-indigo-500 bg-clip-text text-transparent cursor-pointer"
            >
              CINEKURD
            </div>

            {/* سێرچ بار */}
            <div className="relative flex-1 max-w-md">
              <input
                type="text"
                value={searchQuery}
                onChange={handleSearch}
                placeholder="ل ناڤێ فلیم یان زنجیرەکێ بگەڕە..."
                className="w-full bg-slate-950 border border-slate-700 rounded-xl px-4 py-2 text-sm text-white placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
              {searchResults.length > 0 && (
                <div className="absolute left-0 right-0 mt-2 bg-slate-900 border border-slate-800 rounded-xl shadow-2xl max-h-80 overflow-y-auto z-50 divide-y divide-slate-800">
                  {searchResults.slice(0, 6).map((item) => (
                    <div
                      key={item.id}
                      onClick={() => { openPlayer(item); setSearchResults([]); setSearchQuery(""); }}
                      className="p-3 flex items-center gap-3 hover:bg-slate-800 cursor-pointer transition text-right"
                    >
                      <img
                        src={`https://image.tmdb.org/t/p/w92${item.poster_path}`}
                        alt={item.title || item.name}
                        className="w-10 h-14 object-cover rounded bg-slate-950"
                      />
                      <div className="flex-1 min-w-0">
                        <div className="text-sm font-semibold truncate">{item.title || item.name}</div>
                        <div className="text-xs text-slate-400 flex items-center gap-2 mt-1">
                          <span>{item.media_type === "tv" ? "زنجیرە" : "فلیم"}</span>
                          <span>•</span>
                          <span className="text-yellow-400">★ {item.vote_average?.toFixed(1)}</span>
                        </div>
                      </div>
                    </div>
                  ))}
                </div>
              )}
            </div>
          </header>

          {/* ناڤەڕۆک */}
          <main className="flex-1 p-4 md:p-8 max-w-7xl mx-auto w-full">
            {view === "watch" && selectedItem ? (
              <div className="space-y-6">
                <button
                  onClick={() => setView("home")}
                  className="bg-slate-900 border border-slate-800 hover:bg-slate-800 text-sm px-4 py-2 rounded-xl transition flex items-center gap-2"
                >
                  ← زڤرین بۆ سەرپەڕی
                </button>

                {/* پلەیەرێ ئامادە */}
                <div className="aspect-video w-full bg-black rounded-2xl overflow-hidden shadow-2xl border border-slate-800">
                  <iframe
                    key={`${selectedItem.id}-${season}-${episode}`}
                    src={
                      mediaType === "tv"
                        ? `https://vidsrc.to/embed/tv/${selectedItem.id}/${season}/${episode}`
                        : `https://vidsrc.to/embed/movie/${selectedItem.id}`
                    }
                    className="w-full h-full border-0"
                    allowFullScreen
                    allow="autoplay; encrypted-media; picture-in-picture"
                    title={selectedItem.title || selectedItem.name}
                  />
                </div>

                {/* کۆنترۆڵا وەرز و خەلەکان بۆ زنجیرەیان */}
                {mediaType === "tv" && (
                  <div className="bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-wrap items-center gap-4">
                    <div className="flex items-center gap-2">
                      <span className="text-sm text-slate-400">وەرز (Season):</span>
                      <select
                        value={season}
                        onChange={(e) => { setSeason(Number(e.target.value)); setEpisode(1); }}
                        className="bg-slate-950 border border-slate-700 rounded-lg px-3 py-1.5 text-sm"
                      >
                        {[1, 2, 3, 4, 5, 6, 7, 8, 9, 10].map(s => (
                          <option key={s} value={s}>وەرزێ {s}</option>
                        ))}
                      </select>
                    </div>

                    <div className="flex items-center gap-2">
                      <span className="text-sm text-slate-400">خەلەک (Episode):</span>
                      <select
                        value={episode}
                        onChange={(e) => setEpisode(Number(e.target.value))}
                        className="bg-slate-950 border border-slate-700 rounded-lg px-3 py-1.5 text-sm"
                      >
                        {Array.from({ length: 30 }, (_, i) => i + 1).map(ep => (
                          <option key={ep} value={ep}>خەلەکا {ep}</option>
                        ))}
                      </select>
                    </div>
                  </div>
                )}

                {/* پێزانینێن فلیم و زنجیرێ */}
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-6 flex flex-col md:flex-row gap-6 items-start">
                  {selectedItem.poster_path && (
                    <img
                      src={`https://image.tmdb.org/t/p/w500${selectedItem.poster_path}`}
                      alt={selectedItem.title || selectedItem.name}
                      className="w-36 h-52 object-cover rounded-xl shadow-lg border border-slate-800"
                    />
                  )}
                  <div className="space-y-3 flex-1 text-right">
                    <h1 className="text-2xl font-bold">{selectedItem.title || selectedItem.name}</h1>
                    <div className="flex items-center gap-3 text-sm text-slate-400">
                      <span className="bg-yellow-500/20 text-yellow-400 px-2.5 py-0.5 rounded-full border border-yellow-500/30">
                        ★ {selectedItem.vote_average?.toFixed(1)}
                      </span>
                      <span>{selectedItem.release_date || selectedItem.first_air_date}</span>
                    </div>
                    <p className="text-slate-300 text-sm leading-relaxed">{selectedItem.overview || "چ زانیاری بەردەست نینن."}</p>
                  </div>
                </div>
              </div>
            ) : (
              /* لاپەڕێ سەرەکی */
              <div className="space-y-10 text-right">
                {/* بەشێ ترێند */}
                <section className="space-y-4">
                  <div className="flex items-center gap-2 border-b border-slate-800 pb-2">
                    <span className="w-2.5 h-6 bg-red-500 rounded-full inline-block"></span>
                    <h2 className="text-xl font-bold">بەرهەمێن ترێند (Trending)</h2>
                  </div>
                  <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
                    {trending.slice(0, 12).map((item) => (
                      <div
                        key={item.id}
                        onClick={() => openPlayer(item)}
                        className="group bg-slate-900 border border-slate-800 hover:border-blue-500/60 rounded-xl overflow-hidden cursor-pointer transition transform hover:-translate-y-1 shadow-lg"
                      >
                        <div className="relative aspect-[2/3] bg-slate-950">
                          {item.poster_path ? (
                            <img
                              src={`https://image.tmdb.org/t/p/w500${item.poster_path}`}
                              alt={item.title || item.name}
                              className="w-full h-full object-cover"
                            />
                          ) : (
                            <div className="flex h-full items-center justify-center text-xs text-slate-500">وێنە نینە</div>
                          )}
                          <div className="absolute top-2 right-2 bg-slate-950/80 backdrop-blur px-2 py-0.5 rounded text-[11px] font-bold text-yellow-400">
                            ★ {item.vote_average?.toFixed(1)}
                          </div>
                        </div>
                        <div className="p-2.5">
                          <h4 className="text-xs font-semibold truncate group-hover:text-blue-400 transition">
                            {item.title || item.name}
                          </h4>
                          <span className="text-[10px] text-slate-400">
                            {(item.release_date || item.first_air_date || "").split("-")[0]}
                          </span>
                        </div>
                      </div>
                    ))}
                  </div>
                </section>

                {/* بەشێ بەناوبانگ */}
                <section className="space-y-4">
                  <div className="flex items-center gap-2 border-b border-slate-800 pb-2">
                    <span className="w-2.5 h-6 bg-blue-500 rounded-full inline-block"></span>
                    <h2 className="text-xl font-bold">فلیمێن بەناوبانگ (Popular)</h2>
                  </div>
                  <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
                    {popular.slice(0, 12).map((item) => (
                      <div
                        key={item.id}
                        onClick={() => openPlayer(item)}
                        className="group bg-slate-900 border border-slate-800 hover:border-blue-500/60 rounded-xl overflow-hidden cursor-pointer transition transform hover:-translate-y-1 shadow-lg"
                      >
                        <div className="relative aspect-[2/3] bg-slate-950">
                          {item.poster_path ? (
                            <img
                              src={`https://image.tmdb.org/t/p/w500${item.poster_path}`}
                              alt={item.title || item.name}
                              className="w-full h-full object-cover"
                            />
                          ) : (
                            <div className="flex h-full items-center justify-center text-xs text-slate-500">وێنە نینە</div>
                          )}
                          <div className="absolute top-2 right-2 bg-slate-950/80 backdrop-blur px-2 py-0.5 rounded text-[11px] font-bold text-yellow-400">
                            ★ {item.vote_average?.toFixed(1)}
                          </div>
                        </div>
                        <div className="p-2.5">
                          <h4 className="text-xs font-semibold truncate group-hover:text-blue-400 transition">
                            {item.title || item.name}
                          </h4>
                          <span className="text-[10px] text-slate-400">
                            {(item.release_date || "").split("-")[0]}
                          </span>
                        </div>
                      </div>
                    ))}
                  </div>
                </section>
              </div>
            )}
          </main>

          <footer className="bg-slate-950 border-t border-slate-800 py-6 text-center text-xs text-slate-500">
            CineKurd • پلاتفۆرمێ تاقیکردنێ ب زاراڤێ بادینی
          </footer>
        </div>
      );
    }

    ReactDOM.createRoot(document.getElementById("root")).render(<App />);
  </script>
</body>
</html>

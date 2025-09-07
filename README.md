<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>FlashWire – Real-time Market Events (Enhanced)</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"/>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            'dark-bg': '#0f172a',
            'card-bg': '#1e293b',
            'border-color': '#334155',
            'success': '#22c55e',
            'warning': '#f59e0b',
            'danger': '#ef4444'
          }
        }
      }
    }
  </script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
    body { font-family: 'Inter', sans-serif; background: linear-gradient(135deg,#0f172a 0%,#0b1223 100%); color: #e5e7eb; }
    .event-card { transition: all .18s ease; }
    .event-card:hover { border-color: #22d3ee; transform: translateY(-1px); }
    .ticker-badge { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono', 'Courier New', monospace; }
    .pulse { animation: pulse 2s infinite; }
    @keyframes pulse { 0%{opacity:1} 50%{opacity:.5} 100%{opacity:1} }
    .glass { background: rgba(30,41,59,.55); backdrop-filter: blur(6px); }
    .scroll-smooth { scroll-behavior: smooth; }
    .btn { @apply px-3 py-2 rounded-lg border border-slate-700 bg-slate-800/60 hover:bg-slate-700 text-sm; }
  </style>
</head>
<body class="scroll-smooth">
  <div id="app" class="min-h-screen p-4 md:p-6">
    <div class="max-w-7xl mx-auto">
      <!-- Header -->
      <header class="mb-6">
        <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
          <div>
            <h1 class="text-2xl md:text-3xl font-bold flex items-center">
              <span class="bg-gradient-to-r from-cyan-400 to-blue-500 bg-clip-text text-transparent">FlashWire</span>
              <span id="liveBadge" class="ml-3 bg-cyan-900 text-cyan-100 text-xs px-2 py-1 rounded-full flex items-center gap-1">
                <span class="h-2 w-2 rounded-full bg-green-400 pulse"></span> LIVE
              </span>
            </h1>
            <p class="text-gray-400 text-sm mt-1">Institutional-grade market events • Real-time alerts • Multi-asset (Equities, Crypto, ETFs, FX)</p>
          </div>
          <div class="flex flex-wrap items-center gap-3">
            <div class="relative">
              <i class="fas fa-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400"></i>
              <input id="searchInput" type="text" placeholder="Search tickers, text… ( / to focus )" class="pl-10 pr-10 py-2 bg-slate-800 border border-slate-700 rounded-lg text-sm w-full md:w-72 focus:outline-none focus:ring-2 focus:ring-cyan-500" />
              <button id="clearSearch" class="hidden absolute right-2 top-1/2 -translate-y-1/2 text-gray-400 hover:text-white"><i class="fa fa-times"></i></button>
            </div>
            <div class="flex gap-2">
              <button id="pauseBtn" class="btn">Pause</button>
              <button id="connectBtn" class="btn hidden">Connect</button>
              <button id="settingsBtn" class="btn" title="Settings"><i class="fas fa-sliders-h"></i></button>
            </div>
          </div>
        </div>
      </header>

      <!-- Filters / Controls -->
      <div class="glass border border-slate-700 rounded-xl p-4 mb-6">
        <div class="grid grid-cols-1 md:grid-cols-5 gap-4">
          <div>
            <label class="block text-xs text-gray-400 mb-1">Asset Class</label>
            <select id="assetFilter" class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm">
              <option value="all">All</option>
              <option value="equity">Equities</option>
              <option value="crypto">Crypto</option>
              <option value="etf">ETFs</option>
              <option value="forex">FX</option>
            </select>
          </div>
          <div>
            <label class="block text-xs text-gray-400 mb-1">Event Types</label>
            <select id="typeFilter" class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm" multiple>
              <option>Guidance Update</option>
              <option>FDA Approval</option>
              <option>Merger Announcement</option>
              <option>Earnings Beat</option>
              <option>Regulatory Action</option>
              <option>Product Recall</option>
              <option>SEC Filing</option>
              <option>Clinical Trial Results</option>
              <option>Crypto Listing</option>
              <option>Network Outage</option>
              <option>Macro Print</option>
            </select>
            <p class="text-xs text-gray-500 mt-1">Hold Ctrl/Cmd to multi-select</p>
          </div>
          <div>
            <label class="block text-xs text-gray-400 mb-1">Min Rank</label>
            <input id="rankRange" type="range" min="0" max="100" value="60" class="w-full" />
            <div class="flex justify-between text-xs text-gray-400"><span>0</span><span id="rankVal">60</span></div>
          </div>
          <div>
            <label class="block text-xs text-gray-400 mb-1">Sort</label>
            <select id="sortSelect" class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-sm">
              <option value="time">Newest</option>
              <option value="rank">Rank (desc)</option>
              <option value="latency">Latency (asc)</option>
            </select>
          </div>
          <div class="flex items-end gap-2">
            <label class="inline-flex items-center gap-2 text-sm">
              <input id="showMuted" type="checkbox" class="accent-cyan-500" /> Show muted
            </label>
            <label class="inline-flex items-center gap-2 text-sm">
              <input id="onlyWatchlist" type="checkbox" class="accent-cyan-500" /> Watchlist only
            </label>
          </div>
        </div>
      </div>

      <!-- Stats Bar -->
      <div class="grid grid-cols-2 md:grid-cols-5 gap-4 mb-6">
        <div class="glass border border-slate-700 rounded-lg">
          <div class="p-4 flex items-center">
            <div class="rounded-full bg-cyan-900/30 p-2 mr-3"><i class="fas fa-bell text-cyan-400"></i></div>
            <div><p class="text-gray-400 text-sm">Events/hr</p><p id="statEventsHr" class="text-xl font-bold">0</p></div>
          </div>
        </div>
        <div class="glass border border-slate-700 rounded-lg">
          <div class="p-4 flex items-center">
            <div class="rounded-full bg-green-900/30 p-2 mr-3"><i class="fas fa-chart-line text-green-400"></i></div>
            <div><p class="text-gray-400 text-sm">Precision (mock)</p><p id="statPrecision" class="text-xl font-bold">92%</p></div>
          </div>
        </div>
        <div class="glass border border-slate-700 rounded-lg">
          <div class="p-4 flex items-center">
            <div class="rounded-full bg-blue-900/30 p-2 mr-3"><i class="fas fa-clock text-blue-400"></i></div>
            <div><p class="text-gray-400 text-sm">Avg Latency</p><p id="statLatency" class="text-xl font-bold">–</p></div>
          </div>
        </div>
        <div class="glass border border-slate-700 rounded-lg">
          <div class="p-4 flex items-center">
            <div class="rounded-full bg-purple-900/30 p-2 mr-3"><i class="fas fa-eye-slash text-purple-400"></i></div>
            <div><p class="text-gray-400 text-sm">Muted Tickers</p><p id="statMuted" class="text-xl font-bold">0</p></div>
          </div>
        </div>
        <div class="glass border border-slate-700 rounded-lg">
          <div class="p-4 flex items-center">
            <div class="rounded-full bg-green-900/30 p-2 mr-3"><span id="statusDot" class="h-2 w-2 rounded-full bg-green-400 inline-block"></span></div>
            <div><p class="text-gray-400 text-sm">Status</p><p id="statStatus" class="text-xl font-bold">Live</p></div>
          </div>
        </div>
      </div>

      <!-- Events Feed -->
      <div class="space-y-4">
        <div class="flex items-center justify-between">
          <h2 class="text-lg font-semibold">Real-time Market Events</h2>
          <div class="flex items-center gap-2">
            <span class="bg-slate-800 text-gray-300 text-sm px-2 py-1 rounded"><span id="eventCount">0</span> events</span>
            <button id="exportBtn" class="btn" title="Export visible events JSON"><i class="fa fa-file-export mr-1"></i>Export</button>
          </div>
        </div>
        <div id="eventsContainer" class="space-y-4"></div>
      </div>

      <!-- Footer -->
      <footer class="mt-10 pt-6 border-t border-slate-800 text-center text-sm text-gray-500">
        <p>FlashWire • Institutional Market Intelligence • Updated in real-time</p>
        <p class="mt-1">Latency p95 (mock): 1.2s • Coverage: 99% • Keyboard: J/K navigate, Enter open, M mute, F follow</p>
      </footer>
    </div>
  </div>

  <!-- Detail Drawer -->
  <div id="detailDrawer" class="fixed inset-y-0 right-0 w-full sm:w-[520px] translate-x-full transition-transform duration-300 z-40">
    <div class="h-full glass border-l border-slate-700 p-0 flex flex-col">
      <div class="p-4 border-b border-slate-700 flex items-start justify-between">
        <div>
          <div class="flex items-center gap-2">
            <span id="ddAssetBadge" class="text-xs px-2 py-0.5 rounded bg-slate-900 border border-slate-700"></span>
            <span id="ddTicker" class="ticker-badge text-xl font-bold"></span>
            <span id="ddCompany" class="text-gray-400 text-sm"></span>
          </div>
          <div class="mt-1 flex items-center gap-2 text-xs text-gray-400">
            <span id="ddExchange"></span>
            <span>•</span>
            <span id="ddCurrency"></span>
          </div>
        </div>
        <div class="flex items-center gap-2">
          <button id="ddFollowBtn" class="btn text-xs"><i class="fa fa-star mr-1"></i>Follow</button>
          <button id="ddMuteBtn" class="btn text-xs"><i class="fa fa-bell-slash mr-1"></i>Mute</button>
          <button id="ddCloseBtn" class="btn" title="Close"><i class="fa fa-times"></i></button>
        </div>
      </div>
      <div class="p-4 space-y-4 overflow-y-auto">
        <div>
          <h3 class="font-semibold mb-2">Headline</h3>
          <p id="ddHeadline" class="text-lg"></p>
          <div class="mt-2 text-sm text-gray-400 flex items-center gap-3">
            <span id="ddEventType" class="px-2 py-0.5 rounded border border-slate-600"></span>
            <span id="ddConfidence" class="px-2 py-0.5 rounded"></span>
            <span id="ddRank" class="px-2 py-0.5 rounded border border-slate-600"></span>
            <span id="ddLatency" class="font-mono text-cyan-400"></span>
          </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div class="bg-slate-900/50 border border-slate-700 rounded-lg p-3">
            <p class="text-xs text-gray-400 mb-1">Impact</p>
            <p id="ddImpact"></p>
          </div>
          <div class="bg-slate-900/50 border border-slate-700 rounded-lg p-3">
            <p class="text-xs text-gray-400 mb-1">Mechanism</p>
            <p id="ddMechanism"></p>
          </div>
          <div class="bg-slate-900/50 border border-slate-700 rounded-lg p-3">
            <p class="text-xs text-gray-400 mb-1">Watch</p>
            <p id="ddWatch"></p>
          </div>
        </div>

        <div class="bg-slate-900/50 border border-slate-700 rounded-lg p-3">
          <div class="flex items-center justify-between">
            <h3 class="font-semibold">Price</h3>
            <div class="flex items-center gap-2 text-xs">
              <button data-tf="1m" class="tf-btn btn">1m</button>
              <button data-tf="5m" class="tf-btn btn">5m</button>
              <button data-tf="1d" class="tf-btn btn">1d</button>
            </div>
          </div>
          <div class="mt-3 grid grid-cols-3 gap-2 text-sm">
            <div><span class="text-gray-400">Last</span><div id="ddLast" class="font-semibold"></div></div>
            <div><span class="text-gray-400">1m</span><div id="ddCh1m" class="font-semibold"></div></div>
            <div><span class="text-gray-400">5m</span><div id="ddCh5m" class="font-semibold"></div></div>
          </div>
          <div class="mt-3 h-28"><svg id="ddSpark" viewBox="0 0 300 100" class="w-full h-full"></svg></div>
        </div>

        <div class="bg-slate-900/50 border border-slate-700 rounded-lg p-3">
          <h3 class="font-semibold mb-2">Timeline</h3>
          <div id="ddTimeline" class="space-y-2 text-sm"></div>
        </div>

        <div class="bg-slate-900/50 border border-slate-700 rounded-lg p-3">
          <h3 class="font-semibold mb-2">Source</h3>
          <div class="text-sm text-gray-300">Source: <a id="ddSource" href="#" target="_blank" class="text-cyan-400 underline">Open</a></div>
          <div id="ddSourceMeta" class="text-xs text-gray-400 mt-1"></div>
          <div class="mt-3 flex gap-2">
            <button id="ddCopyBtn" class="btn text-xs"><i class="fa fa-copy mr-1"></i>Copy headline</button>
            <button id="ddShareBtn" class="btn text-xs"><i class="fa fa-share mr-1"></i>Share</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Settings Modal -->
  <div id="settingsModal" class="fixed inset-0 bg-black/50 hidden z-50">
    <div class="max-w-xl mx-auto mt-20 glass border border-slate-700 rounded-xl p-6">
      <div class="flex items-center justify-between mb-3">
        <h3 class="text-lg font-semibold">Settings</h3>
        <button id="settingsClose" class="btn"><i class="fa fa-times"></i></button>
      </div>
      <div class="space-y-4 text-sm">
        <div>
          <label class="block text-xs text-gray-400 mb-1">WebSocket URL</label>
          <input id="wsUrlInput" type="text" class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2" placeholder="wss://your-server/ws" />
          <p class="text-xs text-gray-500 mt-1">Leave empty to use mock generator.</p>
        </div>
        <div class="flex items-center gap-2">
          <button id="saveSettings" class="btn"><i class="fa fa-save mr-1"></i>Save</button>
          <button id="testConnect" class="btn"><i class="fa fa-plug mr-1"></i>Test connect</button>
        </div>
      </div>
    </div>
  </div>

  <script>
  /**********************
   * FlashWire Frontend *
   **********************/
  const cfg = {
    wsUrl: localStorage.getItem('fw.wsUrl') || '',
    maxEvents: 400,
    notifyOnFollow: true,
  };

  // State
  const State = {
    events: [],             // newest first
    byTicker: new Map(),    // ticker -> events[]
    muted: new Set(JSON.parse(localStorage.getItem('fw.muted')||'[]')),
    watchlist: new Set(JSON.parse(localStorage.getItem('fw.watch')||'[]')),
    paused: false,
    connected: false,
    lastHourTimestamps: [],
    filters: {
      asset: 'all',
      types: new Set(),
      minRank: 60,
      showMuted: false,
      onlyWatchlist: false,
      sort: 'time',
      search: ''
    },
    selectedEventId: null
  };

  // Elements
  const el = (id)=>document.getElementById(id);
  const eventsContainer = el('eventsContainer');

  // Utilities
  const fmtTime = (d)=> new Date(d).toLocaleTimeString([], {hour:'2-digit',minute:'2-digit',second:'2-digit'});
  const confCls = (c)=> c==='high'?'bg-green-900/30 text-green-400':(c==='med'?'bg-yellow-900/30 text-yellow-400':'bg-red-900/30 text-red-400');
  const dirIcon = (d)=> d==='up'?'<i class="fas fa-arrow-up text-green-500"></i>':(d==='down'?'<i class="fas fa-arrow-down text-red-500"></i>':'<i class="fas fa-minus text-gray-500"></i>');
  const assetBadge = (a)=> ({equity:'Equity', crypto:'Crypto', etf:'ETF', forex:'FX'}[a]||'Other');
  const clsAsset = (a)=> a==='crypto'? 'bg-indigo-900/40 text-indigo-300' : a==='etf'? 'bg-emerald-900/40 text-emerald-300' : a==='forex'? 'bg-amber-900/40 text-amber-300' : 'bg-slate-900/40 text-slate-300';

  // Desktop notifications
  if (('Notification' in window) && Notification.permission==='default') {
    Notification.requestPermission().catch(()=>{});
  }
  function notifyFollow(e){
    if (!('Notification' in window)) return;
    if (Notification.permission!=='granted') return;
    if (!cfg.notifyOnFollow) return;
    new Notification(`${e.ticker}: ${e.eventType}`, { body: e.headline, tag: e.ticker, silent: true });
  }

  // WebSocket (pluggable)
  let ws = null, reconnectTimer = null;
  function connectWS(){
    if (!cfg.wsUrl){ setConnected(false); return; }
    try{
      ws = new WebSocket(cfg.wsUrl);
      ws.onopen = ()=>{ setConnected(true); el('connectBtn').classList.add('hidden'); };
      ws.onmessage = (msg)=>{
        try { const e = JSON.parse(msg.data); addEvent(e); }
        catch(err){ console.warn('WS parse error', err); }
      };
      ws.onclose = ()=>{ setConnected(false); scheduleReconnect(); };
      ws.onerror = ()=>{ setConnected(false); };
    }catch(err){ console.error(err); setConnected(false); scheduleReconnect(); }
  }
  function scheduleReconnect(){
    clearTimeout(reconnectTimer);
    reconnectTimer = setTimeout(connectWS, 2000);
    el('connectBtn').classList.remove('hidden');
  }
  function setConnected(v){
    State.connected = v;
    el('statusDot').className = `h-2 w-2 rounded-full ${v? 'bg-green-400':'bg-red-400'}`;
    el('statStatus').textContent = v? 'Live' : 'Offline';
  }

  // Mock generator (fallback)
  const MOCK_TICKERS = [
    {t:'AAPL', n:'Apple Inc.', a:'equity', ex:'NASDAQ', ccy:'USD'},
    {t:'TSLA', n:'Tesla, Inc.', a:'equity', ex:'NASDAQ', ccy:'USD'},
    {t:'NVDA', n:'NVIDIA', a:'equity', ex:'NASDAQ', ccy:'USD'},
    {t:'BTC-USD', n:'Bitcoin', a:'crypto', ex:'Coinbase', ccy:'USD'},
    {t:'ETH-USD', n:'Ethereum', a:'crypto', ex:'Coinbase', ccy:'USD'},
    {t:'SPY', n:'SPDR S&P 500', a:'etf', ex:'NYSE Arca', ccy:'USD'},
    {t:'EURUSD', n:'Euro / US Dollar', a:'forex', ex:'FX', ccy:'USD'}
  ];
  const EVENT_TYPES = [
    'Guidance Update','FDA Approval','Merger Announcement','Earnings Beat','Regulatory Action','Product Recall','SEC Filing','Clinical Trial Results','Crypto Listing','Network Outage','Macro Print'
  ];
  const IMPACTS = [
    'Revenue guidance raised 15% above consensus','EPS beat by $0.12, revenue up 12% YoY','Acquisition offer at 25% premium to market','8-K filing indicates material contract win','Product recall affects 500K units','Phase 3 trial shows 80% efficacy','Network congestion resolved; TPS +40%','CPI prints 0.2% m/m vs 0.3% exp.'
  ];
  const MECHS = [
    'Investor confidence boosted by strong outlook','Strong fundamentals drive institutional buying','Takeover premium creates acquisition buzz','Material event triggers re-rating potential','Liability concerns impact brand perception','Positive trial results accelerate launch timeline','Macro surprise shifts rate-path odds'
  ];
  const WATCHES = [
    'Next earnings in 3 weeks','Shareholder vote next month','Hearing on Oct 21','PDUFA in 30 days','FOMC next Wed','Mainnet patch ETA 24h','ECB presser 14:30 CET'
  ];

  function rnd(min,max){ return Math.floor(Math.random()*(max-min+1))+min; }
  function pick(arr){ return arr[Math.floor(Math.random()*arr.length)]; }
  function genSpark(){ const n=40; let v=100+Math.random()*10, pts=[v]; for(let i=1;i<n;i++){ v += (Math.random()-.5)*2; pts.push(v);} return pts; }

  function generateMockEvent(){
    const s = pick(MOCK_TICKERS);
    const et = pick(EVENT_TYPES);
    const dir = pick(['up','down','mixed']);
    const conf = pick(['low','med','high']);
    const latency = rnd(120, 1800);
    const now = Date.now();
    const ev = {
      id: Math.random().toString(36).slice(2,10),
      ts: now,
      ticker: s.t,
      company: s.n,
      asset: s.a,
      exchange: s.ex,
      currency: s.ccy,
      eventType: et,
      headline: `${s.t} ${et}: ${pick(IMPACTS)}`,
      impact: pick(IMPACTS),
      mechanism: pick(MECHS),
      watch: pick(WATCHES),
      source: pick(['SEC EDGAR 8-K','Company PR','Exchange Notice','Regulator','FDA Press Release','Stats Office']),
      url: '#',
      confidence: conf,
      direction: dir,
      rank: rnd(60, 98),
      latency,
      price: +(90+Math.random()*100).toFixed(2),
      ch1m: +(Math.random()*4-2).toFixed(2),
      ch5m: +(Math.random()*6-3).toFixed(2),
      spark: genSpark()
    };
    return ev;
  }

  // Core – add event
  function addEvent(ev){
    // normalize fields if coming from WS: expected keys align with above structure
    State.events.unshift(ev);
    if (State.events.length>cfg.maxEvents) State.events.pop();
    const arr = State.byTicker.get(ev.ticker) || [];
    arr.unshift(ev); State.byTicker.set(ev.ticker, arr);
    State.lastHourTimestamps.push(ev.ts);
    const cutoff = Date.now()-3600_000;
    State.lastHourTimestamps = State.lastHourTimestamps.filter(t=>t>=cutoff);
    // Notifications
    if (State.watchlist.has(ev.ticker)) notifyFollow(ev);
    if (!State.paused) render();
  }

  // Rendering – stats
  function renderStats(){
    el('statEventsHr').textContent = State.lastHourTimestamps.length.toString();
    const lat = State.events.length? Math.round(State.events.slice(0,20).reduce((a,b)=>a+(b.latency||0),0)/Math.min(20,State.events.length))+'ms' : '–';
    el('statLatency').textContent = lat;
    el('statMuted').textContent = State.muted.size.toString();
  }

  // Filters
  function passFilters(ev){
    const f=State.filters;
    if (f.asset!=='all' && ev.asset!==f.asset) return false;
    if (f.types.size>0 && !f.types.has(ev.eventType)) return false;
    if (!f.showMuted && State.muted.has(ev.ticker)) return false;
    if (f.onlyWatchlist && !State.watchlist.has(ev.ticker)) return false;
    if (ev.rank < f.minRank) return false;
    const s=f.search.trim().toLowerCase();
    if (s){
      const hay = `${ev.ticker} ${ev.company||''} ${ev.headline||''} ${ev.eventType||''}`.toLowerCase();
      if (!hay.includes(s)) return false;
    }
    return true;
  }

  // Sorting
  function sortEvents(list){
    const s = State.filters.sort;
    if (s==='rank') return list.sort((a,b)=> b.rank - a.rank);
    if (s==='latency') return list.sort((a,b)=> (a.latency||9999) - (b.latency||9999));
    return list.sort((a,b)=> b.ts - a.ts);
  }

  // Event card HTML
  function card(ev){
    return `
      <div class="event-card bg-slate-800/50 border border-slate-700 rounded-lg" data-id="${ev.id}" data-ticker="${ev.ticker}">
        <div class="p-4">
          <div class="flex flex-col md:flex-row md:items-center justify-between gap-2 mb-3">
            <div class="flex flex-wrap items-center gap-2 md:gap-3">
              <span class="ticker-badge font-mono font-bold text-lg bg-slate-900 px-2 py-1 rounded">${ev.ticker}</span>
              <span class="hidden md:inline text-gray-400">${ev.company||''}</span>
              <span class="text-xs px-2 py-0.5 rounded border border-slate-600">${ev.eventType}</span>
              <span class="text-xs px-2 py-0.5 rounded ${confCls(ev.confidence)}">${ev.confidence} confidence</span>
              <span class="text-xs px-2 py-0.5 rounded ${clsAsset(ev.asset)}">${assetBadge(ev.asset)}</span>
            </div>
            <div class="flex items-center gap-3 text-sm text-gray-400">
              <div class="flex items-center"><i class="far fa-clock mr-1"></i><span>${fmtTime(ev.ts)}</span></div>
              <div class="flex items-center bg-slate-900/50 px-2 py-1 rounded"><span class="font-mono text-cyan-400">T+${ev.latency||'–'}ms</span></div>
            </div>
          </div>
          <h3 class="font-semibold text-lg mb-3">${ev.headline}</h3>
          <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-4">
            <div class="md:col-span-3 grid grid-cols-1 md:grid-cols-3 gap-4">
              <div><p class="text-sm text-gray-400 mb-1">Impact</p><p>${ev.impact||''}</p></div>
              <div><p class="text-sm text-gray-400 mb-1">Mechanism</p><p>${ev.mechanism||''}</p></div>
              <div><p class="text-sm text-gray-400 mb-1">Watch</p><p>${ev.watch||''}</p></div>
            </div>
            <div class="bg-slate-900/40 border border-slate-700 rounded p-2">
              <div class="flex items-center justify-between text-sm">
                <div class="flex items-center gap-2"><span>${dirIcon(ev.direction)}</span><span class="capitalize">${ev.direction||''}</span></div>
                <span class="text-xs border border-slate-600 px-1.5 py-0.5 rounded">Rank ${Math.round(ev.rank)}</span>
              </div>
              <div class="mt-2 grid grid-cols-3 gap-2 text-xs">
                <div><span class="text-gray-400">Last</span><div class="font-semibold">${ev.price??'–'} ${ev.currency||''}</div></div>
                <div><span class="text-gray-400">1m</span><div class="font-semibold ${ev.ch1m>0?'text-green-400':(ev.ch1m<0?'text-red-400':'')}">${ev.ch1m??'–'}%</div></div>
                <div><span class="text-gray-400">5m</span><div class="font-semibold ${ev.ch5m>0?'text-green-400':(ev.ch5m<0?'text-red-400':'')}">${ev.ch5m??'–'}%</div></div>
              </div>
              <div class="mt-2 h-12"><svg viewBox="0 0 200 48" class="w-full h-full">${sparkPath(ev.spark||[])}</svg></div>
            </div>
          </div>
          <div class="flex flex-col md:flex-row md:items-center justify-between gap-3 pt-3 border-t border-slate-700">
            <div class="text-sm text-gray-400 flex items-center gap-2">Source: <a class="text-cyan-400 underline" href="${ev.url||'#'}" target="_blank">${ev.source||'Unknown'}</a></div>
            <div class="flex items-center gap-2">
              <button class="btn text-sm view-btn" data-id="${ev.id}"><i class="fa fa-eye mr-1"></i>View Details</button>
              <button class="btn text-sm mute-btn" data-ticker="${ev.ticker}"><i class="fa fa-bell-slash mr-1"></i>Mute</button>
              <button class="btn text-sm follow-btn" data-ticker="${ev.ticker}"><i class="fa fa-star mr-1"></i>${State.watchlist.has(ev.ticker)?'Unfollow':'Follow'}</button>
            </div>
          </div>
        </div>
      </div>`;
  }

  function sparkPath(arr){
    if (!arr || arr.length<2) return '';
    const w=200, h=48, min=Math.min(...arr), max=Math.max(...arr), span=max-min||1;
    const pts = arr.map((v,i)=>{
      const x = i*(w/(arr.length-1));
      const y = h - ((v-min)/span)*h;
      return `${x.toFixed(2)},${y.toFixed(2)}`;
    }).join(' ');
    return `<polyline points="${pts}" fill="none" stroke="currentColor" stroke-width="2" class="text-cyan-400" />`;
  }

  // Render list
  function renderList(){
    const filtered = sortEvents(State.events.filter(passFilters));
    el('eventCount').textContent = filtered.length.toString();
    if (!filtered.length){
      eventsContainer.innerHTML = `<div class="bg-slate-800/50 border border-slate-700 rounded-lg"><div class="py-12 text-center text-gray-400">No events match your filters.</div></div>`;
      return;
    }
    eventsContainer.innerHTML = filtered.map(card).join('');
    // bind buttons
    document.querySelectorAll('.mute-btn').forEach(b=> b.addEventListener('click', (e)=> toggleMute(e.target.closest('button').dataset.ticker)));
    document.querySelectorAll('.follow-btn').forEach(b=> b.addEventListener('click', (e)=> toggleFollow(e.target.closest('button').dataset.ticker)));
    document.querySelectorAll('.view-btn').forEach(b=> b.addEventListener('click', (e)=> openDetail(e.target.closest('button').dataset.id)));
  }

  function render(){ renderStats(); renderList(); }

  // Mute / Follow
  function toggleMute(t){
    if (State.muted.has(t)) State.muted.delete(t); else State.muted.add(t);
    localStorage.setItem('fw.muted', JSON.stringify([...State.muted]));
    render();
  }
  function toggleFollow(t){
    if (State.watchlist.has(t)) State.watchlist.delete(t); else State.watchlist.add(t);
    localStorage.setItem('fw.watch', JSON.stringify([...State.watchlist]));
    render();
  }

  // Detail Drawer
  function openDetail(id){
    const e = State.events.find(x=>x.id===id);
    if (!e) return;
    State.selectedEventId=id;
    // header
    el('ddAssetBadge').textContent = assetBadge(e.asset);
    el('ddAssetBadge').className = `text-xs px-2 py-0.5 rounded border ${clsAsset(e.asset)} border-slate-700`;
    el('ddTicker').textContent = e.ticker;
    el('ddCompany').textContent = e.company||'';
    el('ddExchange').textContent = e.exchange||'';
    el('ddCurrency').textContent = e.currency||'';
    // headline
    el('ddHeadline').textContent = e.headline||'';
    el('ddEventType').textContent = e.eventType||'';
    el('ddConfidence').textContent = (e.confidence||'').toUpperCase();
    el('ddConfidence').className = `px-2 py-0.5 rounded ${confCls(e.confidence)}`;
    el('ddRank').textContent = `Rank ${Math.round(e.rank||0)}`;
    el('ddLatency').textContent = `T+${e.latency||'–'}ms`;
    el('ddImpact').textContent = e.impact||'';
    el('ddMechanism').textContent = e.mechanism||'';
    el('ddWatch').textContent = e.watch||'';
    el('ddLast').textContent = `${e.price??'–'} ${e.currency||''}`;
    el('ddCh1m').textContent = `${e.ch1m>0?'+':''}${e.ch1m??'–'}%`;
    el('ddCh5m').textContent = `${e.ch5m>0?'+':''}${e.ch5m??'–'}%`;
    // spark
    const svg = el('ddSpark'); svg.innerHTML = sparkPath(e.spark||[]);
    // timeline
    const tl = State.byTicker.get(e.ticker)||[];
    el('ddTimeline').innerHTML = tl.slice(0,10).map(x=> `
      <div class="flex items-start gap-2">
        <div class="w-16 text-xs text-gray-400">${fmtTime(x.ts)}</div>
        <div class="flex-1">
          <div class="text-sm">${x.headline}</div>
          <div class="text-xs text-gray-500">${x.eventType} • Rank ${Math.round(x.rank)}</div>
        </div>
      </div>`).join('');
    // source
    el('ddSource').href = e.url || '#';
    el('ddSource').textContent = e.source || 'Open';
    el('ddSourceMeta').textContent = `First seen: ${fmtTime(e.ts)} • Exchange: ${e.exchange||''}`;
    // controls
    el('ddFollowBtn').innerHTML = `<i class="fa fa-star mr-1"></i>${State.watchlist.has(e.ticker)?'Unfollow':'Follow'}`;
    el('ddMuteBtn').innerHTML = `<i class="fa fa-bell-slash mr-1"></i>${State.muted.has(e.ticker)?'Unmute':'Mute'}`;
    el('ddFollowBtn').onclick = ()=> { toggleFollow(e.ticker); openDetail(id); };
    el('ddMuteBtn').onclick = ()=> { toggleMute(e.ticker); openDetail(id); };

    // open drawer
    const dr = el('detailDrawer');
    dr.classList.remove('translate-x-full');
  }
  function closeDetail(){ el('detailDrawer').classList.add('translate-x-full'); }

  // Export visible
  function exportVisible(){
    const filtered = sortEvents(State.events.filter(passFilters));
    const blob = new Blob([JSON.stringify(filtered, null, 2)], {type:'application/json'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href = url; a.download = `flashwire_export_${Date.now()}.json`; a.click(); URL.revokeObjectURL(url);
  }

  // Keyboard shortcuts (J/K navigate, Enter open, M mute, F follow)
  let cursor = -1;
  function moveCursor(delta){
    const nodes = [...document.querySelectorAll('[data-id]')];
    if (!nodes.length) return;
    cursor = Math.max(0, Math.min(nodes.length-1, cursor+delta));
    nodes.forEach(n=>n.classList.remove('ring-2','ring-cyan-500'));
    const node = nodes[cursor]; node.scrollIntoView({block:'nearest'}); node.classList.add('ring-2','ring-cyan-500');
  }
  document.addEventListener('keydown', (e)=>{
    if (e.key==='/' && document.activeElement!==el('searchInput')){ e.preventDefault(); el('searchInput').focus(); return; }
    if (e.key==='j' || e.key==='J'){ moveCursor(1); }
    if (e.key==='k' || e.key==='K'){ moveCursor(-1); }
    if (e.key==='Enter'){ const node=[...document.querySelectorAll('[data-id]')][cursor]; if(node){ openDetail(node.dataset.id); }}
    if (e.key==='m' || e.key==='M'){ const node=[...document.querySelectorAll('[data-id]')][cursor]; if(node){ toggleMute(node.dataset.ticker); }}
    if (e.key==='f' || e.key==='F'){ const node=[...document.querySelectorAll('[data-id]')][cursor]; if(node){ toggleFollow(node.dataset.ticker); }}
  });

  // UI bindings
  el('pauseBtn').onclick = ()=> { State.paused = !State.paused; el('pauseBtn').textContent = State.paused? 'Resume':'Pause'; el('liveBadge').classList.toggle('opacity-50', State.paused); };
  el('connectBtn').onclick = connectWS;
  el('ddCloseBtn').onclick = closeDetail;
  el('exportBtn').onclick = exportVisible;

  // Filters change
  el('assetFilter').onchange = (e)=> { State.filters.asset = e.target.value; render(); };
  el('typeFilter').onchange = (e)=>{
    const opts=[...e.target.selectedOptions].map(o=>o.value); State.filters.types = new Set(opts); render();
  };
  el('rankRange').oninput = (e)=> { State.filters.minRank = +e.target.value; el('rankVal').textContent = e.target.value; render(); };
  el('sortSelect').onchange = (e)=> { State.filters.sort = e.target.value; render(); };
  el('showMuted').onchange = (e)=> { State.filters.showMuted = e.target.checked; render(); };
  el('onlyWatchlist').onchange = (e)=> { State.filters.onlyWatchlist = e.target.checked; render(); };
  el('searchInput').addEventListener('input', (e)=> { State.filters.search = e.target.value; el('clearSearch').classList.toggle('hidden', !e.target.value); render(); });
  el('clearSearch').onclick = ()=> { el('searchInput').value=''; State.filters.search=''; el('clearSearch').classList.add('hidden'); render(); };

  // Settings
  el('settingsBtn').onclick = ()=> { el('settingsModal').classList.remove('hidden'); el('wsUrlInput').value = cfg.wsUrl; };
  el('settingsClose').onclick = ()=> el('settingsModal').classList.add('hidden');
  el('saveSettings').onclick = ()=> { cfg.wsUrl = el('wsUrlInput').value.trim(); localStorage.setItem('fw.wsUrl', cfg.wsUrl); el('settingsModal').classList.add('hidden'); connectWS(); };
  el('testConnect').onclick = ()=> { cfg.wsUrl = el('wsUrlInput').value.trim(); connectWS(); };

  // Init
  function init(){
    // prefill type multiselect with none (all) – or pick defaults
    // Start WS if configured
    connectWS();
    // Fallback mock loop
    setInterval(()=>{ if (!State.paused && !State.connected) addEvent(generateMockEvent()); }, 1300);
    // seed a few
    for(let i=0;i<6;i++) addEvent(generateMockEvent());
    render();
  }
  document.addEventListener('DOMContentLoaded', init);
  </script>
</body>
</html>

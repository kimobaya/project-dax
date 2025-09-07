# project-dax
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>FlashWire - Real-time Market Events</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            'dark-bg': '#0f172a',
            'card-bg': '#1e293b',
            'border-color': '#334155'
          }
        }
      }
    }
  </script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
    body {
      font-family: 'Inter', sans-serif;
      background: linear-gradient(135deg, #0f172a 0%, #0c1221 100%);
      color: #f1f5f9;
      min-height: 100vh;
    }
    .event-card {
      transition: all 0.2s ease;
    }
    .event-card:hover {
      border-color: #22d3ee;
    }
    .ticker-badge {
      font-family: monospace;
    }
    .pulse {
      animation: pulse 2s infinite;
    }
    @keyframes pulse {
      0% { opacity: 1; }
      50% { opacity: 0.5; }
      100% { opacity: 1; }
    }
  </style>
</head>
<body>
  <div class="min-h-screen p-4 md:p-6">
    <div class="max-w-7xl mx-auto">
      <!-- Header -->
      <header class="mb-6">
        <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
          <div>
            <h1 class="text-2xl md:text-3xl font-bold flex items-center">
              <span class="bg-gradient-to-r from-cyan-400 to-blue-500 bg-clip-text text-transparent">
                FlashWire
              </span>
              <span class="ml-3 bg-cyan-900 text-cyan-100 text-xs px-2 py-1 rounded-full">LIVE</span>
            </h1>
            <p class="text-gray-400 text-sm mt-1">
              Institutional-grade market events • Real-time alerts
            </p>
          </div>
          
          <div class="flex items-center gap-3">
            <div class="relative">
              <i class="fas fa-search absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
              <input
                type="text"
                id="searchInput"
                placeholder="Search tickers or events..."
                class="pl-10 pr-4 py-2 bg-slate-800 border border-slate-700 rounded-lg text-sm w-full md:w-64 focus:outline-none focus:ring-2 focus:ring-cyan-500"
              />
            </div>
            
            <button 
              id="pauseBtn"
              class="px-3 py-2 bg-slate-800 border border-slate-700 rounded-lg text-sm text-gray-300 hover:bg-slate-700 transition-colors"
            >
              Pause
            </button>
            
            <button class="p-2 text-gray-400 hover:text-white rounded-lg hover:bg-slate-800">
              <i class="fas fa-cog"></i>
            </button>
          </div>
        </div>
      </header>

      <!-- Stats Bar -->
      <div class="grid grid-cols-2 md:grid-cols-5 gap-4 mb-6">
        <div class="bg-slate-800/50 border border-slate-700 rounded-lg">
          <div class="p-4">
            <div class="flex items-center">
              <div class="rounded-full bg-cyan-900/30 p-2 mr-3">
                <i class="fas fa-bell text-cyan-400"></i>
              </div>
              <div>
                <p class="text-gray-400 text-sm">Events/hr</p>
                <p class="text-xl font-bold">1,247</p>
              </div>
            </div>
          </div>
        </div>
        
        <div class="bg-slate-800/50 border border-slate-700 rounded-lg">
          <div class="p-4">
            <div class="flex items-center">
              <div class="rounded-full bg-green-900/30 p-2 mr-3">
                <i class="fas fa-chart-line text-green-400"></i>
              </div>
              <div>
                <p class="text-gray-400 text-sm">Precision</p>
                <p class="text-xl font-bold">92%</p>
              </div>
            </div>
          </div>
        </div>
        
        <div class="bg-slate-800/50 border border-slate-700 rounded-lg">
          <div class="p-4">
            <div class="flex items-center">
              <div class="rounded-full bg-blue-900/30 p-2 mr-3">
                <i class="fas fa-clock text-blue-400"></i>
              </div>
              <div>
                <p class="text-gray-400 text-sm">Avg Latency</p>
                <p class="text-xl font-bold">842ms</p>
              </div>
            </div>
          </div>
        </div>
        
        <div class="bg-slate-800/50 border border-slate-700 rounded-lg">
          <div class="p-4">
            <div class="flex items-center">
              <div class="rounded-full bg-purple-900/30 p-2 mr-3">
                <i class="fas fa-eye-slash text-purple-400"></i>
              </div>
              <div>
                <p class="text-gray-400 text-sm">Muted Tickers</p>
                <p class="text-xl font-bold">0</p>
              </div>
            </div>
          </div>
        </div>
        
        <div class="bg-slate-800/50 border border-slate-700 rounded-lg">
          <div class="p-4">
            <div class="flex items-center">
              <div class="rounded-full bg-green-900/30 p-2 mr-3">
                <div class="h-2 w-2 rounded-full bg-green-400 pulse"></div>
              </div>
              <div>
                <p class="text-gray-400 text-sm">Status</p>
                <p class="text-xl font-bold">Live</p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Events Feed -->
      <div class="space-y-4">
        <div class="flex items-center justify-between">
          <h2 class="text-lg font-semibold">Real-time Market Events</h2>
          <span class="bg-slate-800 text-gray-300 text-sm px-2 py-1 rounded">
            <span id="eventCount">0</span> events
          </span>
        </div>
        
        <div id="eventsContainer" class="space-y-4">
          <!-- Events will be inserted here -->
        </div>
      </div>
      
      <!-- Footer -->
      <footer class="mt-8 pt-6 border-t border-slate-800 text-center text-sm text-gray-500">
        <p>FlashWire • Institutional Market Intelligence • Updated in real-time</p>
        <p class="mt-1">Latency p95: 1.2s • Precision: 92% • Coverage: 99%</p>
      </footer>
    </div>
  </div>

  <script>
    // Event data structure
    const events = [];
    let isPaused = false;
    const mutedTickers = new Set();
    
    // DOM elements
    const eventsContainer = document.getElementById('eventsContainer');
    const eventCountElement = document.getElementById('eventCount');
    const searchInput = document.getElementById('searchInput');
    const pauseBtn = document.getElementById('pauseBtn');
    
    // Event templates
    const eventTypes = [
      "Guidance Update", 
      "FDA Approval", 
      "Merger Announcement", 
      "Earnings Beat", 
      "Regulatory Action",
      "Product Recall",
      "SEC Filing",
      "Clinical Trial Results"
    ];
    
    const impacts = [
      "Revenue guidance raised 15% above consensus",
      "FDA approves breakthrough therapy designation",
      "Acquisition offer at 25% premium to market",
      "EPS beat by $0.12, revenue up 12% YoY",
      "DOJ antitrust investigation initiated",
      "Product recall affects 500K units",
      "8-K filing indicates material contract win",
      "Phase 3 trial shows 80% efficacy"
    ];
    
    const mechanisms = [
      "Investor confidence boosted by strong outlook",
      "Market validation of drug development pipeline",
      "Takeover premium creates acquisition buzz",
      "Strong fundamentals drive institutional buying",
      "Regulatory uncertainty creates selling pressure",
      "Liability concerns impact brand perception",
      "Material event triggers re-rating potential",
      "Positive trial results accelerate drug launch timeline"
    ];
    
    const watches = [
      "Next earnings: Q3 guidance update",
      "PDUFA date: 2023-11-15",
      "Shareholder approval by 2023-12-01",
      "Analyst upgrades expected next week",
      "Court ruling expected in Q2 2024",
      "Recall completion by 2023-10-31",
      "Contract implementation begins Q4",
      "Commercial launch Q1 2024"
    ];
    
    const sources = [
      "SEC EDGAR 8-K",
      "FDA Press Release",
      "Company PR",
      "Regulatory Notice",
      "Exchange Filing",
      "Clinical Trial Database"
    ];
    
    const tickers = ["AAPL", "TSLA", "MSFT", "GOOGL", "AMZN", "META", "NVDA", "AMD", "INTC", "ORCL"];
    const directions = ["up", "down", "mixed"];
    const confidences = ["low", "med", "high"];
    
    // Generate mock event
    function generateMockEvent() {
      const ticker = tickers[Math.floor(Math.random() * tickers.length)];
      const eventType = eventTypes[Math.floor(Math.random() * eventTypes.length)];
      const impact = impacts[Math.floor(Math.random() * impacts.length)];
      const mechanism = mechanisms[Math.floor(Math.random() * mechanisms.length)];
      const watch = watches[Math.floor(Math.random() * watches.length)];
      const source = sources[Math.floor(Math.random() * sources.length)];
      const direction = directions[Math.floor(Math.random() * directions.length)];
      const confidence = confidences[Math.floor(Math.random() * confidences.length)];
      
      return {
        id: Math.random().toString(36).substring(2, 9),
        timestamp: new Date(Date.now() - Math.random() * 300000),
        ticker,
        eventType,
        headline: `${ticker} ${eventType}: ${impact.split(" ").slice(0, 5).join(" ")}...`,
        impact,
        mechanism,
        watch,
        source,
        confidence,
        direction,
        rank: Math.floor(Math.random() * 35) + 65,
        latency: Math.floor(Math.random() * 2000)
      };
    }
    
    // Format time
    function formatTime(date) {
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }
    
    // Get confidence color
    function getConfidenceColor(confidence) {
      switch(confidence) {
        case "high": return "bg-green-900/30 text-green-400";
        case "med": return "bg-yellow-900/30 text-yellow-400";
        case "low": return "bg-red-900/30 text-red-400";
        default: return "bg-gray-700 text-gray-300";
      }
    }
    
    // Get direction icon
    function getDirectionIcon(direction) {
      switch(direction) {
        case "up": return '<i class="fas fa-arrow-up text-green-500"></i>';
        case "down": return '<i class="fas fa-arrow-down text-red-500"></i>';
        case "mixed": return '<i class="fas fa-minus text-gray-500"></i>';
        default: return '';
      }
    }
    
    // Render event card
    function renderEvent(event) {
      return `
        <div class="event-card bg-slate-800/50 border border-slate-700 rounded-lg hover:border-cyan-500/50 transition-colors" data-ticker="${event.ticker}">
          <div class="p-4">
            <div class="flex flex-col md:flex-row md:items-center justify-between gap-2 mb-3">
              <div class="flex items-center gap-3">
                <div class="flex items-center">
                  <span class="ticker-badge font-mono font-bold text-lg bg-slate-900 px-2 py-1 rounded">
                    ${event.ticker}
                  </span>
                  <div class="ml-2 flex items-center">
                    ${getDirectionIcon(event.direction)}
                    <span class="ml-1 text-xs capitalize">${event.direction}</span>
                  </div>
                </div>
                
                <span class="text-xs px-2 py-1 rounded ${getConfidenceColor(event.confidence)}">
                  ${event.confidence} confidence
                </span>
                
                <span class="text-xs border border-slate-600 text-gray-300 px-2 py-1 rounded">
                  ${event.eventType}
                </span>
              </div>
              
              <div class="flex items-center gap-3 text-sm text-gray-400">
                <div class="flex items-center">
                  <i class="far fa-clock mr-1"></i>
                  <span>${formatTime(event.timestamp)}</span>
                </div>
                <div class="flex items-center bg-slate-900/50 px-2 py-1 rounded">
                  <span class="font-mono text-cyan-400">T+${event.latency}ms</span>
                </div>
              </div>
            </div>
            
            <h3 class="font-semibold text-lg mb-3">${event.headline}</h3>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-4">
              <div>
                <p class="text-sm text-gray-400 mb-1">Impact</p>
                <p>${event.impact}</p>
              </div>
              <div>
                <p class="text-sm text-gray-400 mb-1">Mechanism</p>
                <p>${event.mechanism}</p>
              </div>
              <div>
                <p class="text-sm text-gray-400 mb-1">Watch</p>
                <p>${event.watch}</p>
              </div>
            </div>
            
            <div class="flex flex-col md:flex-row md:items-center justify-between gap-3 pt-3 border-t border-slate-700">
              <div class="text-sm text-gray-400">
                Source: <span class="text-cyan-400">${event.source}</span>
              </div>
              
              <div class="flex items-center gap-2">
                <button class="text-sm px-3 py-1 text-gray-400 hover:text-white hover:bg-slate-700 rounded transition-colors mute-btn" data-ticker="${event.ticker}">
                  Mute ${event.ticker}
                </button>
                <button class="text-sm px-3 py-1 border border-slate-600 text-gray-300 hover:bg-slate-700 rounded transition-colors">
                  View Details
                </button>
              </div>
            </div>
          </div>
        </div>
      `;
    }
    
    // Add event to feed
    function addEvent(event) {
      events.unshift(event);
      if (events.length > 50) {
        events.pop();
      }
      updateEventFeed();
    }
    
    // Update event feed display
    function updateEventFeed() {
      const searchTerm = searchInput.value.toLowerCase();
      const filteredEvents = events.filter(event => 
        !mutedTickers.has(event.ticker) &&
        (event.ticker.includes(searchTerm.toUpperCase()) || 
         event.headline.toLowerCase().includes(searchTerm))
      );
      
      eventCountElement.textContent = filteredEvents.length;
      
      if (filteredEvents.length === 0) {
        eventsContainer.innerHTML = `
          <div class="bg-slate-800/50 border border-slate-700 rounded-lg">
            <div class="py-12 text-center">
              <i class="fas fa-bell text-4xl text-gray-600 mb-4"></i>
              <h3 class="text-lg font-medium mb-1">No events to display</h3>
              <p class="text-gray-400">
                ${searchTerm 
                  ? `No events match "${searchTerm}"` 
                  : mutedTickers.size > 0 
                    ? "All events are muted. Adjust your filters to see alerts."
                    : "Waiting for new market events..."}
              </p>
            </div>
          </div>
        `;
        return;
      }
      
      eventsContainer.innerHTML = filteredEvents.map(event => renderEvent(event)).join('');
      
      // Add event listeners to mute buttons
      document.querySelectorAll('.mute-btn').forEach(button => {
        button.addEventListener('click', (e) => {
          const ticker = e.target.getAttribute('data-ticker');
          toggleMuteTicker(ticker);
        });
      });
    }
    
    // Toggle mute for ticker
    function toggleMuteTicker(ticker) {
      if (mutedTickers.has(ticker)) {
        mutedTickers.delete(ticker);
      } else {
        mutedTickers.add(ticker);
      }
      updateEventFeed();
    }
    
    // Toggle pause
    function togglePause() {
      isPaused = !isPaused;
      pauseBtn.textContent = isPaused ? 'Resume' : 'Pause';
      document.querySelector('.pulse').classList.toggle('pulse', !isPaused);
    }
    
    // Initialize
    function init() {
      // Add initial events
      for (let i = 0; i < 5; i++) {
        addEvent(generateMockEvent());
      }
      
      // Set up event generation
      setInterval(() => {
        if (!isPaused) {
          addEvent(generateMockEvent());
        }
      }, 1500);
      
      // Set up search
      searchInput.addEventListener('input', updateEventFeed);
      
      // Set up pause button
      pauseBtn.addEventListener('click', togglePause);
      
      // Add event listeners to mute buttons
      document.querySelectorAll('.mute-btn').forEach(button => {
        button.addEventListener('click', (e) => {
          const ticker = e.target.getAttribute('data-ticker');
          toggleMuteTicker(ticker);
        });
      });
    }
    
    // Start when DOM is loaded
    document.addEventListener('DOMContentLoaded', init);
  </script>
</body>
</html>
Share
Refresh
Copy

Version 3 of 3

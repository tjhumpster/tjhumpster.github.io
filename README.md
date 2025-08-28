<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>OVO Alps Coverage — Heatmap + Targets</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    html, body { height: 100%; margin: 0; font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; background:#f6f5ef; }
    #app { display: grid; grid-template-columns: 360px 1fr; height: 100%; }
    #sidebar { padding: 18px 18px 0; border-right: 1px solid #e6e2d6; background: #f9f7f1; overflow:auto; }
    h1 { font-size: 24px; margin: 0 0 8px; }
    .sub { color:#6b665b; font-size: 13px; line-height: 1.35; }
    #map { height: 100vh; width: 100%; }
    .control { margin: 14px 0; padding: 10px 12px; background: #fff; border:1px solid #e6e2d6; border-radius: 14px; box-shadow: 0 1px 0 rgba(0,0,0,0.03); }
    .legend { position:absolute; bottom:16px; right:16px; background:#fff; padding:10px 12px; border:1px solid #e6e2d6; border-radius: 12px; font-size: 12px; line-height:1.2; box-shadow:0 2px 12px rgba(0,0,0,.06); }
    .legend .bar { width: 180px; height: 12px; background: linear-gradient(90deg, #1e3a8a, #3b82f6, #facc15, #ef4444, #b91c1c); border-radius: 6px; margin:6px 0; }
    .legend .bar2 { width: 180px; height: 12px; background: linear-gradient(90deg, #1e3a8a, #3b82f6, #facc15, #ef4444, #b91c1c); border-radius: 6px; margin:6px 0; }
    .kpi { display:flex; gap:10px; flex-wrap:wrap; }
    .pill { background:#fff; border:1px solid #e6e2d6; border-radius:999px; padding:6px 10px; font-size:12px; }
    .footer { color:#7b766b; font-size:12px; margin-top:8px; }
    .marker-label{ font-size:11px; color:#3b372f; font-weight:600; }
  </style>
</head>
<body>
  <div id="app">
    <aside id="sidebar">
      <h1>OVO Alps Coverage</h1>
      <p class="sub">Heatmap showing chalet coverage (blue → red) and recommended growth areas (blue → red). Toggle layers below.</p>
      <div class="control">
        <label><input type="checkbox" id="toggleHeat" checked/> Coverage heat</label><br/>
        <label><input type="checkbox" id="toggleTargets" checked/> Target areas heat</label><br/>
        <label style="display:block;margin-top:8px;">Intensity scale <span id="scaleOut">/40</span>
          <input type="range" id="scale" min="10" max="60" value="40" step="5" style="width:100%"/>
        </label>
      </div>
      <div class="kpi">
        <span class="pill" id="totalPill">Total chalets: …</span>
        <span class="pill" id="placesPill">Places: …</span>
      </div>
      <p class="footer">Blue → Red = coverage and target intensities (vibrant scale).</p>
    </aside>
    <div id="map"></div>
  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script src="https://unpkg.com/leaflet.heat/dist/leaflet-heat.js"></script>
  <script>
    // --- Data: current coverage ---
    const data = [
      ["La Clusaz",45.904,6.423,37],["Le Grand-Bornand",45.941,6.427,19],["Les Clefs",45.850,6.400,2],["Manigod",45.860,6.390,27],["Saint-Jean-de-Sixt",45.920,6.410,10],["Thônes",45.880,6.320,5],["Villard-sur-Thônes",45.900,6.320,3],
      ["Chamonix",45.923,6.869,1],["Les Houches",45.889,6.798,1],["Combloux",45.900,6.640,1],["Domancy",45.930,6.640,1],["Cordon",45.920,6.590,3],["La Giettaz",45.820,6.510,8],["Les Contamines-Montjoie",45.820,6.730,1],["Megève",45.850,6.620,2],["Saint-Gervais",45.890,6.710,10],["Bionnassay",45.860,6.790,3],["Saint-Nicolas-de-Véroce",45.840,6.680,2],
      ["Morillon",46.080,6.680,3],["La Rivière-Enverse",46.070,6.650,2],["Samoëns",46.080,6.740,13],["Sixt-Fer-à-Cheval",46.100,6.770,1],
      ["Alex",45.940,6.160,1],["Annecy",45.900,6.120,1],["Doussard",45.770,6.230,1],["Menthon-Saint-Bernard",45.860,6.180,1],["Saint-Jorioz",45.830,6.150,1],["Talloires",45.840,6.210,2],
      ["Châtel",46.270,6.840,13],["La Chapelle-d'Abondance",46.290,6.790,2],["Les Gets",46.160,6.670,8],["Mieussy",46.140,6.530,1],["Montriond",46.200,6.700,2],["Morzine",46.180,6.770,6],["Essert-Romand",46.189,6.695,1],["La Côte-d'Arbroz",46.200,6.660,1],["Seytroux",46.250,6.590,1],["Saint-Jean-d'Aulps",46.240,6.650,3],
      ["Saint-Martin-de-Belleville",45.380,6.500,1],
      ["Crest-Voland",45.780,6.510,3],["Flumet",45.820,6.520,2],["Saint-Nicolas-la-Chapelle",45.800,6.530,2],["Notre-Dame-de-Bellecombe",45.820,6.530,3],
      ["Doucy",45.460,6.430,1],["Valmorel",45.450,6.450,1],
      ["Chartreuse (St-Pierre)",45.340,5.820,1],["Saint-Pierre-d'Entremont",45.370,5.870,1],
      ["La Toussuire",45.250,6.280,1],["Le Corbier",45.230,6.270,1],["Villarembert",45.230,6.270,1],
      ["Les Arcs",45.580,6.780,2],["Séez",45.620,6.800,2],
      ["Villars-sur-Ollon",46.300,7.060,1],["Praz-de-Fort",45.990,7.140,1]
    ];

    // --- Data: target future areas (strategic expansion) ---
    const targets = [
      ["Arêches-Beaufort",45.720,6.570,8],["Hauteluce",45.760,6.570,6],
      ["Aillons-Margériaz",45.620,6.050,5],["Lescheraines",45.730,6.080,4],
      ["Bernex",46.360,6.700,4],["Thollon-les-Mémises",46.380,6.650,3],["Abondance",46.280,6.730,3],
      ["Albiez-Montrond",45.250,6.340,5],["Aussois",45.240,6.740,4],["Val Cenis (Lanslebourg)",45.290,6.880,6],
      ["Le Sappey-en-Chartreuse",45.270,5.770,3],["St-Pierre-de-Chartreuse",45.330,5.820,4],
      ["Peisey-Nancroix",45.540,6.750,4],["Villaroger",45.570,6.840,3],["Montchavin-Les Coches",45.560,6.740,4]
    ];

    const scaleEl = document.getElementById('scale');
    const scaleOut = document.getElementById('scaleOut');

    const map = L.map('map', { zoomSnap: 0.25 }).setView([46.0, 6.55], 8.25);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 18, attribution: '&copy; OpenStreetMap' }).addTo(map);

    // Compute totals
    const total = data.reduce((s, d) => s + d[3], 0);
    document.getElementById('totalPill').textContent = `Total chalets: ${total}`;
    document.getElementById('placesPill').textContent = `Places: ${data.length}`;

    // Coverage heat layer
    let heatLayer;
    function buildHeat() {
      const denom = Number(scaleEl.value);
      const pts = data.map(([name, lat, lon, cnt]) => [lat, lon, Math.max(0.05, cnt/denom)]);
      if (heatLayer) heatLayer.remove();
      heatLayer = L.heatLayer(pts, { 
        radius: 28, blur: 22, maxZoom: 13, minOpacity: 0.25,
        gradient:{0.1:'#1e3a8a',0.3:'#3b82f6',0.6:'#facc15',0.8:'#ef4444',1:'#b91c1c'}
      });
      if (document.getElementById('toggleHeat').checked) heatLayer.addTo(map);
      scaleOut.textContent = `/${denom}`;
    }
    buildHeat();

    // Target heat layer
    let targetLayer;
    function buildTargetHeat() {
      const pts = targets.map(([name, lat, lon, cnt]) => [lat, lon, cnt/20]);
      if (targetLayer) targetLayer.remove();
      targetLayer = L.heatLayer(pts, { 
        radius: 28, blur: 22, maxZoom: 13, minOpacity:0.3,
        gradient:{0.1:'#1e3a8a',0.3:'#3b82f6',0.6:'#facc15',0.8:'#ef4444',1:'#b91c1c'}
      });
      if (document.getElementById('toggleTargets').checked) targetLayer.addTo(map);
    }
    buildTargetHeat();

    // Toggles
    document.getElementById('toggleHeat').addEventListener('change', (e) => {
      if (e.target.checked) buildHeat(); else if (heatLayer) heatLayer.remove();
    });
    document.getElementById('toggleTargets').addEventListener('change', (e) => {
      if (e.target.checked) buildTargetHeat(); else if (targetLayer) targetLayer.remove();
    });
    scaleEl.addEventListener('input', buildHeat);

    // Legend
    const legend = L.control({position:'bottomright'});
    legend.onAdd = function(){
      const div = L.DomUtil.create('div','legend');
      div.innerHTML = `<strong>Coverage density</strong><div class='bar'></div><div style='display:flex;justify-content:space-between;'><span>Low</span><span>High</span></div><br/><strong>Target areas</strong><div class='bar2'></div><div style='display:flex;justify-content:space-between;'><span>Low</span><span>High</span></div>`;
      return div;
    };
    legend.addTo(map);
  </script>
</body>
</html>

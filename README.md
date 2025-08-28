<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>OVO Alps Coverage Heatmap</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <style>
    #map { height: 100vh; width: 100%; }
  </style>
</head>
<body>
  <div id="map"></div>

  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script src="https://unpkg.com/leaflet.heat/dist/leaflet-heat.js"></script>
  <script>
    // Chalet coverage data (mock sample)
    const chalets = [
      [45.8625, 6.4278, 12], // La Clusaz
      [45.9050, 6.4270, 8],  // Le Grand-Bornand
      [46.1890, 6.7680, 10], // Morzine
      [46.3500, 6.7667, 6],  // Châtel
      [46.0833, 6.7250, 7],  // Samoëns
      [45.7100, 6.5670, 5],  // Les Arcs area
      [45.5730, 6.8070, 4],  // Valmorel
      [45.2930, 6.5800, 3],  // St François Longchamp
      [45.1410, 6.3830, 2],  // Valloire
      [45.2160, 6.4310, 2]   // Valmeinier
    ];

    // Target expansion areas (recommended)
    const targets = [
      [45.7214, 6.5726, 6], // Arêches-Beaufort
      [45.6500, 6.5500, 5], // Hauteluce
      [45.6500, 6.1000, 4], // Aillons-Margériaz (Bauges)
      [45.7300, 6.1300, 3], // Lescheraines (Bauges)
      [46.3830, 6.7100, 5], // Bernex
      [46.3830, 6.6500, 4], // Thollon-les-Mémises
      [46.2830, 6.7160, 4], // Abondance
      [45.2420, 6.3500, 4], // Albiez-Montrond
      [45.2330, 6.7400, 4], // Aussois
      [45.2900, 6.9200, 3], // Val Cenis villages
      [45.3000, 5.8160, 4], // St-Pierre-de-Chartreuse
      [45.2700, 5.7900, 3], // Le Sappey-en-Chartreuse
      [45.5500, 6.7300, 4], // Peisey-Nancroix
      [45.5700, 6.8500, 3], // Villaroger
      [45.5670, 6.7330, 3]  // Montchavin-Les Coches
    ];

    // Init map
    const map = L.map('map').setView([45.9, 6.6], 9);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap'
    }).addTo(map);

    // Current coverage heatmap
    const currentHeat = L.heatLayer(chalets, {
      radius: 40,
      blur: 25,
      maxZoom: 12,
      gradient: {
        0.2: "lime",
        0.4: "yellow",
        0.6: "orange",
        0.8: "red"
      }
    });

    // Target coverage heatmap (Blue → Red)
    const targetHeat = L.heatLayer(targets, {
      radius: 40,
      blur: 25,
      maxZoom: 12,
      gradient: {
        0.2: "blue",
        0.4: "deepskyblue",
        0.6: "magenta",
        0.8: "red"
      }
    });

    // Bubble markers for current chalets
    const bubbles = L.layerGroup(
      chalets.map(c => L.circleMarker([c[0], c[1]], {
        radius: c[2],
        color: 'green',
        fillOpacity: 0.6
      }))
    );

    // Add to map
    currentHeat.addTo(map);
    bubbles.addTo(map);

    // Layer controls
    L.control.layers(null, {
      "Current Heat (chalets)": currentHeat,
      "Target Heat (expansion)": targetHeat,
      "Bubbles": bubbles
    }).addTo(map);
  </script>
</body>
</html>

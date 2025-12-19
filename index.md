---
layout: home
title: Nation‑Empires
hero:
  title: Nation‑Empires
  image: /assets/images/hero.jpg
  tagline: "A grand‑strategy micro‑sim — create, compete, and share"
  actions:
    - label: "Play the demo"
      url: /demo
      style: primary
    - label: "Join Discord"
      url: https://discord.gg/your-invite
features:
  - title: "Quick nation generator"
    description: "Build a country in 60 seconds and run a short simulation. Share a stylized nation card."
  - title: "Alliances & Leagues"
    description: "Invite friends, form alliances, and fight for the leaderboard."
  - title: "Creator scenarios"
    description: "Create scenarios and export replays perfect for streamers."
---

<!-- MapLibre CSS (move this into your site head if you prefer) -->
<link href="https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.css" rel="stylesheet" />

<section class="home-section">
  <div class="grid">
    <div class="grid-item hoi-panel">
      <h3>Interactive Map (GeoJSON subdivisions demo)</h3>

      <!-- Map container (fills panel width) -->
      <div id="map" style="width:100%; height:520px; border-radius:4px; overflow:hidden;"></div>

      <!-- MapLibre JS (move to footer include for production) -->
      <script src="https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.js"></script>

      <script>
        // Small helper: compute bbox of a GeoJSON feature (works for Polygon/MultiPolygon)
        function computeFeatureBBox(feature) {
          const coords = (feature.geometry.type === 'Polygon')
            ? feature.geometry.coordinates
            : feature.geometry.type === 'MultiPolygon'
              ? feature.geometry.coordinates.flat()
              : [];
          let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
          coords.forEach(ring => {
            ring.forEach(pt => {
              const [x, y] = pt;
              if (x < minX) minX = x;
              if (y < minY) minY = y;
              if (x > maxX) maxX = x;
              if (y > maxY) maxY = y;
            });
          });
          return [[minX, minY], [maxX, maxY]];
        }

        document.addEventListener("DOMContentLoaded", function () {
          try {
            const map = new maplibregl.Map({
              container: 'map',
              style: 'https://demotiles.maplibre.org/style.json',
              center: [0, 20],
              zoom: 2
            });

            map.addControl(new maplibregl.NavigationControl({ showCompass: true }), 'top-right');

            map.on('load', async () => {
              // Load a small demo GeoJSON from /data/sample-country.geojson
              // Replace this file with your real subdivision GeoJSON files later.
              const geojsonUrl = '/data/sample-country.geojson';

              // Add GeoJSON source (generateId: true makes feature.id available)
              map.addSource('subdivisions', {
                type: 'geojson',
                data: geojsonUrl,
                generateId: true
              });

              // Fill layer for subdivisions
              map.addLayer({
                id: 'subdivisions-fill',
                type: 'fill',
                source: 'subdivisions',
                paint: {
                  'fill-color': ['case',
                    ['boolean', ['feature-state', 'selected'], false],
                    '#ffd166', // selected color
                    '#dbeafe'  // default color
                  ],
                  'fill-opacity': 0.85
                }
              });

              // Border lines
              map.addLayer({
                id: 'subdivisions-line',
                type: 'line',
                source: 'subdivisions',
                paint: {
                  'line-color': '#94a3b8',
                  'line-width': 0.8
                }
              });

              // Hover highlight (uses feature state)
              let hoveredId = null;
              map.on('mousemove', 'subdivisions-fill', (e) => {
                map.getCanvas().style.cursor = 'pointer';
                if (e.features.length > 0) {
                  if (hoveredId !== null && hoveredId !== e.features[0].id) {
                    map.setFeatureState({ source: 'subdivisions', id: hoveredId }, { hover: false });
                  }
                  hoveredId = e.features[0].id;
                  map.setFeatureState({ source: 'subdivisions', id: hoveredId }, { hover: true });
                }
              });
              map.on('mouseleave', 'subdivisions-fill', () => {
                map.getCanvas().style.cursor = '';
                if (hoveredId !== null) {
                  map.setFeatureState({ source: 'subdivisions', id: hoveredId }, { hover: false });
                }
                hoveredId = null;
              });

              // Click handler: popup, highlight, and fit to bounds
              let selectedId = null;
              map.on('click', 'subdivisions-fill', (e) => {
                const feat = e.features && e.features[0];
                if (!feat) return;

                // Show popup with properties
                const props = feat.properties || {};
                const name = props.name || props.NAME || props.NAME_1 || 'Unknown';
                new maplibregl.Popup({ offset: 6 })
                  .setLngLat(e.lngLat)
                  .setHTML(`<strong>${name}</strong><br/>id: ${feat.id}`)
                  .addTo(map);

                // Clear previous selected state
                if (selectedId !== null) {
                  map.setFeatureState({ source: 'subdivisions', id: selectedId }, { selected: false });
                }

                // Set selected state (used by fill-color expression)
                selectedId = feat.id;
                map.setFeatureState({ source: 'subdivisions', id: selectedId }, { selected: true });

                // Fit map to the clicked feature bounds
                const bbox = computeFeatureBBox(feat);
                map.fitBounds(bbox, { padding: 20, maxZoom: 9, duration: 600 });
              });

              // Optional: build a simple sidebar list of features (client-side)
              // We'll fetch the GeoJSON and render a tiny index below the map.
              try {
                const resp = await fetch(geojsonUrl);
                const gj = await resp.json();
                const listEl = document.createElement('div');
                listEl.style.marginTop = '0.75rem';
                listEl.innerHTML = '<strong>Subdivisions (click to zoom):</strong>';
                const ul = document.createElement('ul');
                ul.style.paddingLeft = '1rem';
                (gj.features || []).forEach(f => {
                  const li = document.createElement('li');
                  const display = f.properties.name || f.properties.NAME || f.properties.NAME_1 || `id:${f.id}`;
                  const a = document.createElement('a');
                  a.href = '#';
                  a.textContent = display;
                  a.style.cursor = 'pointer';
                  a.onclick = (ev) => {
                    ev.preventDefault();
                    // programmatically highlight and fit
                    if (selectedId !== null) {
                      map.setFeatureState({ source: 'subdivisions', id: selectedId }, { selected: false });
                    }
                    selectedId = f.id;
                    map.setFeatureState({ source: 'subdivisions', id: selectedId }, { selected: true });
                    const bbox = computeFeatureBBox(f);
                    map.fitBounds(bbox, { padding: 20, maxZoom: 9, duration: 600 });
                  };
                  li.appendChild(a);
                  ul.appendChild(li);
                });
                listEl.appendChild(ul);
                document.getElementById('map').parentNode.appendChild(listEl);
              } catch (err) {
                console.warn('Could not build subdivisions index:', err);
              }
            });

            // Slight resize fix for some Jekyll themes
            setTimeout(() => { map.resize(); }, 250);
          } catch (e) {
            const el = document.getElementById('map');
            if (el) el.innerHTML = '<div style="padding:1rem; color:#cbd5e1">Map failed to load.</div>';
            console.error('Map init error:', e);
          }
        });
      </script>
    </div>

    <aside class="grid-item hoi-panel">
      <h4>Selected Nation</h4>
      <p><strong>Kingdom of Aster</strong></p>
      <ul>
        <li>Population: 12.4M</li>
        <li>GDP: $148B</li>
        <li>Stability: 62%</li>
      </ul>
      <p><a class="button" href="/nations/aster">View profile</a></p>
    </aside>
  </div>
</section>

<section class="home-section">
  <h3>Devlog</h3>
  <ul>
    <li>Initial prototype: nation generator finished.</li>
    <li>Interactive subdivisions demo loaded from <code>/data/sample-country.geojson</code>.</li>
    <li>Discord seeded with 120 alpha testers.</li>
  </ul>
</section>

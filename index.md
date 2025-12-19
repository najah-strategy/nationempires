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

<!-- OpenLayers CSS (move to site head if you prefer) -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/ol@7.4.0/ol.css">

<section class="home-section">
  <div class="grid">
    <div class="grid-item hoi-panel">
      <h3>Interactive Map (OpenLayers subdivisions demo)</h3>

      <!-- Map container -->
      <div id="olmap" style="width:100%; height:520px; border-radius:4px; overflow:hidden;"></div>

      <!-- Popup overlay (OpenLayers overlay element) -->
      <div id="popup" class="ol-popup" style="display:block; position:absolute; background:rgba(0,0,0,0.75); color:#fff; padding:6px 8px; border-radius:4px; font-size:13px; transform:translate(-50%, -100%); pointer-events:none;"></div>

      <!-- Subdivisions list (click to zoom/select) -->
      <div id="subdivisions-index" style="margin-top:0.8rem;">
        <strong>Subdivisions (click to zoom):</strong>
        <ul id="subdiv-list" style="padding-left:1rem; margin-top:0.25rem;"></ul>
      </div>

      <!-- OpenLayers JS (move to footer include for production) -->
      <script src="https://cdn.jsdelivr.net/npm/ol@7.4.0/dist/ol.js"></script>
      <script>
        document.addEventListener('DOMContentLoaded', function () {
          try {
            // Base OSM raster layer
            const raster = new ol.layer.Tile({
              source: new ol.source.OSM()
            });

            // Map view centered globally
            const view = new ol.View({
              center: ol.proj.fromLonLat([0, 20]),
              zoom: 2
            });

            const map = new ol.Map({
              target: 'olmap',
              layers: [raster],
              view: view,
              controls: ol.control.defaults({ attribution: true, rotate: false })
            });

            // Popup overlay
            const popupEl = document.getElementById('popup');
            const popupOverlay = new ol.Overlay({
              element: popupEl,
              positioning: 'bottom-center',
              stopEvent: false,
              offset: [0, -10]
            });
            map.addOverlay(popupOverlay);

            // Load GeoJSON subdivisions (replace with your real subdivision file)
            fetch('/data/sample-country.geojson')
              .then(response => response.json())
              .then(gj => {
                // Read features into vector source (transform to map projection)
                const features = new ol.format.GeoJSON().readFeatures(gj, { featureProjection: 'EPSG:3857' });
                const vectorSource = new ol.source.Vector({ features: features });

                // Styles
                const defaultStyle = new ol.style.Style({
                  stroke: new ol.style.Stroke({ color: '#94a3b8', width: 1 }),
                  fill: new ol.style.Fill({ color: 'rgba(219,234,254,0.85)' })
                });
                const hoverStyle = new ol.style.Style({
                  stroke: new ol.style.Stroke({ color: '#2b6cb0', width: 2 }),
                  fill: new ol.style.Fill({ color: 'rgba(59,130,246,0.2)' })
                });
                const selectedStyle = new ol.style.Style({
                  stroke: new ol.style.Stroke({ color: '#ffcc00', width: 2 }),
                  fill: new ol.style.Fill({ color: 'rgba(255,209,102,0.9)' })
                });

                // Vector layer
                const vectorLayer = new ol.layer.Vector({
                  source: vectorSource,
                  style: function(feature) {
                    return defaultStyle;
                  }
                });
                map.addLayer(vectorLayer);

                // Fit view to data bounds
                if (!vectorSource.isEmpty()) {
                  view.fit(vectorSource.getExtent(), { size: map.getSize(), maxZoom: 8, padding: [20, 20, 20, 20] });
                }

                // Interaction: pointermove (hover)
                const pointerMoveSelect = new ol.interaction.Select({
                  condition: ol.events.condition.pointerMove,
                  layers: [vectorLayer],
                  style: hoverStyle
                });
                map.addInteraction(pointerMoveSelect);

                // Interaction: click select
                const clickSelect = new ol.interaction.Select({
                  condition: ol.events.condition.singleClick,
                  layers: [vectorLayer],
                  style: selectedStyle
                });
                map.addInteraction(clickSelect);

                // Show popup and center on selected feature
                clickSelect.on('select', function(e) {
                  const selected = e.selected[0];
                  if (selected) {
                    const props = selected.getProperties();
                    const name = props.name || props.NAME || props.NAME_1 || 'Unknown';
                    // compute center of geometry extent for overlay position
                    const extent = selected.getGeometry().getExtent();
                    const center = ol.extent.getCenter(extent);
                    popupEl.innerHTML = '<div style="font-weight:700;">' + name + '</div>';
                    popupOverlay.setPosition(center);
                    // optionally fit tighter to feature
                    view.fit(extent, { padding: [20,20,20,20], maxZoom: 9, duration: 400 });
                  } else {
                    popupOverlay.setPosition(undefined);
                  }
                });

                // Build subdivisions index list
                const listEl = document.getElementById('subdiv-list');
                features.forEach((f, idx) => {
                  const li = document.createElement('li');
                  li.style.marginBottom = '0.25rem';
                  const name = f.get('name') || f.get('NAME') || f.get('NAME_1') || ('Region ' + (idx+1));
                  const a = document.createElement('a');
                  a.href = '#';
                  a.textContent = name;
                  a.style.cursor = 'pointer';
                  a.onclick = function(ev) {
                    ev.preventDefault();
                    // zoom to feature and select it
                    const extent = f.getGeometry().getExtent();
                    view.fit(extent, { padding: [30,30,30,30], maxZoom: 9, duration: 400 });
                    clickSelect.getFeatures().clear();
                    clickSelect.getFeatures().push(f);
                  };
                  li.appendChild(a);
                  listEl.appendChild(li);
                });
              })
              .catch(err => {
                console.error('Could not load GeoJSON:', err);
                const errEl = document.createElement('div');
                errEl.textContent = 'Subdivision data failed to load.';
                errEl.style.color = '#ffb4c3';
                document.getElementById('olmap').parentNode.appendChild(errEl);
              });

          } catch (err) {
            console.error('OpenLayers init error:', err);
            const el = document.getElementById('olmap');
            if (el) el.innerHTML = '<div style="padding:1rem; color:#cbd5e1">Map failed to load.</div>';
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
    <li>OpenLayers interactive subdivisions demo loaded from <code>/data/sample-country.geojson</code>.</li>
    <li>Click a region on the map or pick it from the list to zoom and view details.</li>
  </ul>
</section>

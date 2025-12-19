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

<!-- MapLibre CSS (can be moved to your site head if preferred) -->
<link href="https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.css" rel="stylesheet" />

<section class="home-section">
  <div class="grid">
    <div class="grid-item hoi-panel">
      <h3>Live Map</h3>

      <!-- Map container (fills panel width) -->
      <div id="map" style="width:100%; height:360px; border-radius:4px; overflow:hidden;"></div>

      <!-- MapLibre JS (can be moved to site footer or head include) -->
      <script src="https://unpkg.com/maplibre-gl@latest/dist/maplibre-gl.js"></script>
      <script>
        // Basic MapLibre initialization. Adjust center/zoom/style as needed.
        document.addEventListener("DOMContentLoaded", function () {
          try {
            var map = new maplibregl.Map({
              container: 'map',
              style: 'https://demotiles.maplibre.org/style.json', // stylesheet location
              center: [-74.5, 40], // starting position [lng, lat]
              zoom: 3 // starting zoom (reduced for world view)
            });

            // Optional: add navigation controls
            map.addControl(new maplibregl.NavigationControl({ showCompass: true }), 'top-right');

            // Resize map when panel becomes visible (useful in some themes)
            setTimeout(function () { map.resize(); }, 200);
          } catch (e) {
            // If MapLibre fails to load, show a graceful fallback
            var el = document.getElementById('map');
            if (el) el.innerHTML = '<div style="padding:1rem; color:#cbd5e1">Map failed to load.</div>';
            console.error('MapLibre init error:', e);
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
    <li>Discord seeded with 120 alpha testers.</li>
    <li>Creator kit in progress (scenario editor + overlay).</li>
  </ul>
</section>

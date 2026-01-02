<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Strabanc - Visualiseur</title>
    
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
    
    <!-- Leaflet MarkerCluster CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.4.1/dist/MarkerCluster.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.4.1/dist/MarkerCluster.Default.css" />

    <style>
        :root {
            --primary-color: #8d6e63;
            --accent-color: #d84315;
            --gold-color: #FFD700;
            --bg-glass: rgba(255, 255, 255, 0.95);
            --shadow: 0 4px 6px rgba(0,0,0,0.1);
            --font-main: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body, html { margin: 0; padding: 0; height: 100%; width: 100%; font-family: var(--font-main); overflow: hidden; background: #f0f0f0; }
        #map { width: 100%; height: 100%; z-index: 1; }
        
        /* --- Strabanc UI Overlay --- */
        .ui-overlay {
            position: absolute; top: 20px; left: 20px; z-index: 1000;
            width: 280px; /* Un peu plus large */
            max-width: calc(100% - 40px);
            background: var(--bg-glass); 
            box-shadow: var(--shadow);
            border-radius: 25px; /* Arrondi */
            overflow: hidden;
            transition: all 0.3s ease;
        }

        /* En-tête cliquable (Toujours visible) */
        .panel-header {
            background: var(--primary-color);
            color: white;
            padding: 10px 20px;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-weight: bold;
            font-size: 1.1rem;
            user-select: none;
            transition: background 0.3s;
        }
        .panel-header:hover { background: #6d4c41; }
        .panel-header .arrow {
            transition: transform 0.3s ease;
            font-size: 0.8rem;
        }

        /* Contenu caché/affiché */
        .panel-content {
            padding: 20px;
            max-height: 500px; /* Hauteur max quand ouvert */
            opacity: 1;
            transition: max-height 0.4s ease, opacity 0.4s ease, padding 0.4s ease;
            overflow: hidden;
        }

        /* État Réduit (Collapsed) */
        .ui-overlay.collapsed .panel-content {
            max-height: 0;
            padding-top: 0;
            padding-bottom: 0;
            opacity: 0;
        }
        .ui-overlay.collapsed .panel-header {
            border-radius: 25px; /* Devient une pilule complète */
        }
        .ui-overlay.collapsed .arrow {
            transform: rotate(-90deg); /* Flèche vers le haut/bas */
        }

        p { font-size: 0.9rem; color: #666; margin-bottom: 15px; margin-top: 0; }
        
        .stats-panel { background: #f5f5f5; padding: 10px; border-radius: 8px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; font-size: 0.85rem; font-weight: 600; }
        .stat-value { color: var(--accent-color); }
        .reset-btn { width: 100%; padding: 8px; background: #e0e0e0; border: none; cursor: pointer; border-radius: 4px; font-size: 0.8rem; transition: background 0.2s; }
        .reset-btn:hover { background: #d0d0d0; }

        /* --- Icones & Animation --- */
        .bench-icon svg { width: 32px; height: 32px; filter: drop-shadow(1px 2px 2px rgba(0,0,0,0.3)); transition: transform 0.2s, fill 0.3s; }
        .bench-icon.gold svg { animation: goldPulse 2s infinite alternate; }
        @keyframes goldPulse { from { transform: scale(1); filter: drop-shadow(0 0 2px var(--gold-color)); } to { transform: scale(1.1); filter: drop-shadow(0 0 8px var(--gold-color)); } }

        /* --- Popup & Form --- */
        .leaflet-popup-content { margin: 10px; width: 260px !important; font-family: var(--font-main); }
        .popup-header { font-weight: bold; margin-bottom: 10px; border-bottom: 1px solid #eee; padding-bottom: 5px; display: flex; justify-content: space-between; align-items: center;}
        .review-form { background: #fafafa; padding: 10px; border-radius: 6px; margin-top: 10px; border: 1px solid #eee; }
        .form-group { margin-bottom: 8px; }
        .form-group label { display: block; font-size: 0.8rem; color: #555; margin-bottom: 2px; }
        .form-group input, .form-group textarea { width: 100%; padding: 5px; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box; font-size: 0.9rem; }
        .form-group textarea { resize: vertical; min-height: 40px; }
        
        .star-rating { cursor: pointer; display: flex; flex-direction: row-reverse; justify-content: flex-end; }
        .star-rating input { display: none; }
        .star-rating label { font-size: 1.5rem; color: #ddd; transition: color 0.2s; line-height: 1; margin-left: 2px;}
        .star-rating label:hover, .star-rating label:hover ~ label { color: var(--gold-color); }
        .star-rating input:checked ~ label { color: var(--gold-color); }
        .submit-btn { width: 100%; background: var(--primary-color); color: white; border: none; padding: 6px; border-radius: 4px; cursor: pointer; margin-top: 5px; }
        .submit-btn:hover { background: #6d4c41; }

        .comments-list { margin-top: 10px; max-height: 150px; overflow-y: auto; border-top: 1px dashed #ccc; padding-top: 5px; }
        .comment-item { font-size: 0.85rem; margin-bottom: 8px; padding-bottom: 8px; border-bottom: 1px solid #eee; }
        .comment-meta { font-weight: bold; font-size: 0.75rem; color: #777; display: flex; justify-content: space-between;}
        .comment-stars { color: var(--gold-color); }
        .no-reviews { font-size: 0.8rem; color: #999; font-style: italic; text-align: center; margin: 10px 0; }

        /* Loader & Toast */
        #loader { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); z-index: 999; background: rgba(255, 255, 255, 0.95); padding: 20px 30px; border-radius: 8px; box-shadow: 0 8px 16px rgba(0,0,0,0.2); display: none; flex-direction: column; align-items: center; gap: 10px; }
        .spinner { width: 30px; height: 30px; border: 4px solid #f3f3f3; border-top: 4px solid var(--primary-color); border-radius: 50%; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .toast { position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%) translateY(100px); background: #333; color: #fff; padding: 10px 20px; border-radius: 20px; font-size: 0.9rem; opacity: 0; transition: all 0.3s ease; z-index: 2000; }
        .toast.visible { transform: translateX(-50%) translateY(0); opacity: 1; }
    </style>
</head>
<body>

    <div id="map"></div>

    <!-- Strabanc UI Panel -->
    <div class="ui-overlay collapsed" id="uiPanel">
        <!-- En-tête cliquable -->
        <div class="panel-header" onclick="togglePanel()">
            <span>Strabanc</span>
            <span class="arrow">▼</span>
        </div>
        
        <!-- Corps du panneau -->
        <div class="panel-content" id="panelContent">
            <p>Déplacez-vous sans rechargement. Les données s'ajoutent progressivement.</p>
            
            <div class="stats-panel">
                <span>Zone chargée :</span> <span id="zoom-level">Zoom: --</span>
            </div>
            <div class="stats-panel">
                <span>Bancs affichés :</span> <span class="stat-value" id="bench-count">0</span>
            </div>
            <button class="reset-btn" onclick="clearAllData()">Effacer avis et carte</button>
        </div>
    </div>

    <div id="loader"><div class="spinner"></div><span class="loader-text">Chargement...</span></div>
    <div id="toast" class="toast">Message</div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
    <script src="https://unpkg.com/leaflet.markercluster@1.4.1/dist/leaflet.markercluster.js"></script>

    <script>
        // --- 1. Setup ---
        const map = L.map('map').setView([48.8566, 2.3522], 15);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { attribution: '&copy; OSM contributors', maxZoom: 19 }).addTo(map);

        const markerClusterGroup = L.markerClusterGroup({
            showCoverageOnHover: false, 
            zoomToBoundsOnClick: true, 
            disableClusteringAtZoom: 18, 
            maxClusterRadius: 80 
        });
        map.addLayer(markerClusterGroup);

        // Données locales
        let benchData = JSON.parse(localStorage.getItem('benchReviews')) || {};
        let markersMap = {};

        // --- 2. Fonction UI (Toggle Panel) ---
        function togglePanel() {
            const panel = document.getElementById('uiPanel');
            const content = document.getElementById('panelContent');
            panel.classList.toggle('collapsed');
        }

        // --- 3. Variables de gestion de la zone ---
        let loadedBounds = null; 
        const MIN_ZOOM = 14;
        const PADDING_FACTOR = 0.5; 
        let isLoading = false;

        // --- 4. Icon Factory ---
        function createBenchIcon(isGold = false) {
            const color = isGold ? 'var(--gold-color)' : 'var(--primary-color)';
            return L.divIcon({
                className: `bench-icon ${isGold ? 'gold' : ''}`,
                html: `<svg viewBox="0 0 24 24" style="fill: ${color};"><path d="M19,9V20H15V9H19M13,20H11V9H13V20M9,20H5V9H9V20M20,4H4V6H20V4Z" /></svg>`,
                iconSize: [32, 32], iconAnchor: [16, 16], popupAnchor: [0, -10]
            });
        }

        function showToast(msg) {
            const t = document.getElementById('toast');
            t.textContent = msg;
            t.classList.add('visible');
            setTimeout(() => t.classList.remove('visible'), 3000);
        }

        function clearAllData() {
            if(confirm("Tout effacer ?")) {
                benchData = {};
                localStorage.removeItem('benchReviews');
                loadedBounds = null;
                markersMap = {};
                markerClusterGroup.clearLayers();
                document.getElementById('bench-count').textContent = "0";
                fetchBenches(true);
                showToast("Réinitialisé");
            }
        }

        // --- 5. Popup & Review ---
        function generatePopupContent(benchId) {
            const data = benchData[benchId] || { reviews: [] };
            const reviews = data.reviews;
            let avgRating = 0;
            if (reviews.length > 0) {
                const sum = reviews.reduce((acc, r) => acc + parseInt(r.rating), 0);
                avgRating = (sum / reviews.length).toFixed(1);
            }

            let commentsHtml = reviews.length === 0 
                ? '<div class="no-reviews">Aucun avis.</div>' 
                : reviews.map(r => `
                    <div class="comment-item">
                        <div class="comment-meta"><span>${escapeHtml(r.user)}</span><span class="comment-stars">${'★'.repeat(r.rating)}${'☆'.repeat(5-r.rating)}</span></div>
                        <div>${escapeHtml(r.comment)}</div>
                    </div>
                `).join('');

            return `
                <div class="popup-header"><span>Banc #${benchId}</span><span style="color:var(--gold-color)">★ ${avgRating > 0 ? avgRating : '-'}</span></div>
                <div class="comments-list" id="comments-${benchId}">${commentsHtml}</div>
                <div class="review-form">
                    <form onsubmit="submitReview(event, ${benchId})">
                        <div class="form-group">
                            <label>Note :</label>
                            <div class="star-rating">
                                <input type="radio" id="s5-${benchId}" name="rating" value="5" /><label for="s5-${benchId}">★</label>
                                <input type="radio" id="s4-${benchId}" name="rating" value="4" /><label for="s4-${benchId}">★</label>
                                <input type="radio" id="s3-${benchId}" name="rating" value="3" /><label for="s3-${benchId}">★</label>
                                <input type="radio" id="s2-${benchId}" name="rating" value="2" /><label for="s2-${benchId}">★</label>
                                <input type="radio" id="s1-${benchId}" name="rating" value="1" /><label for="s1-${benchId}">★</label>
                            </div>
                        </div>
                        <div class="form-group"><label>Nom :</label><input type="text" id="user-${benchId}" required></div>
                        <div class="form-group"><label>Commentaire :</label><textarea id="comment-${benchId}" required></textarea></div>
                        <button type="submit" class="submit-btn">Noter</button>
                    </form>
                </div>
            `;
        }

        function escapeHtml(text) {
            if (!text) return text;
            return text.replace(/[&<>"']/g, m => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' }[m]));
        }

        window.submitReview = function(e, benchId) {
            e.preventDefault();
            const user = document.getElementById(`user-${benchId}`).value;
            const comment = document.getElementById(`comment-${benchId}`).value;
            const ratingInput = document.querySelector(`input[name="rating"]:checked`);
            if (!ratingInput) return;
            
            if (!benchData[benchId]) benchData[benchId] = { reviews: [] };
            benchData[benchId].reviews.push({ user, comment, rating: parseInt(ratingInput.value) });
            benchData[benchId].isGold = true;
            localStorage.setItem('benchReviews', JSON.stringify(benchData));

            if (markersMap[benchId]) markersMap[benchId].setIcon(createBenchIcon(true));
            
            const marker = markersMap[benchId];
            marker.closePopup();
            setTimeout(() => { marker.bindPopup(generatePopupContent(benchId)).openPopup(); showToast("Avis ajouté !"); }, 100);
        };

        // --- 6. LOGIQUE CHARGEMENT INCRÉMENTAL ---
        async function fetchBenches(forceReload = false) {
            const currentZoom = map.getZoom();
            document.getElementById('zoom-level').textContent = `Zoom: ${currentZoom}`;

            if (currentZoom < MIN_ZOOM) {
                markerClusterGroup.clearLayers();
                markersMap = {};
                loadedBounds = null; 
                document.getElementById('bench-count').textContent = "0";
                return;
            }

            if (isLoading && !forceReload) return;

            const currentBounds = map.getBounds();
            if (!forceReload && loadedBounds && loadedBounds.contains(currentBounds)) return;

            isLoading = true;
            document.getElementById('loader').style.display = 'flex';

            const queryBounds = currentBounds.pad(PADDING_FACTOR);
            const bbox = `${queryBounds.getSouth()},${queryBounds.getWest()},${queryBounds.getNorth()},${queryBounds.getEast()}`;
            const query = `[out:json][timeout:25]; ( node["amenity"="bench"](${bbox}); ); out body; >; out skel qt;`;

            try {
                const response = await fetch(`https://overpass-api.de/api/interpreter?data=${encodeURIComponent(query)}`);
                if (!response.ok) throw new Error("Erreur API");
                const data = await response.json();
                let addedCount = 0;

                data.elements.forEach(el => {
                    if (el.type === 'node' && el.lat && el.lon) {
                        const benchId = el.id;
                        if (!markersMap[benchId]) {
                            const isGold = benchData[benchId] && benchData[benchId].isGold;
                            const marker = L.marker([el.lat, el.lon], { icon: createBenchIcon(isGold) });
                            marker.bindPopup(generatePopupContent(benchId));
                            markersMap[benchId] = marker;
                            markerClusterGroup.addLayer(marker);
                            addedCount++;
                        }
                    }
                });

                loadedBounds = queryBounds;
                document.getElementById('bench-count').textContent = Object.keys(markersMap).length;
                if(addedCount > 0) showToast(`+${addedCount} bancs ajoutés`);

            } catch (err) {
                console.error(err);
                showToast("Erreur API ou connexion.");
            } finally {
                isLoading = false;
                document.getElementById('loader').style.display = 'none';
            }
        }

        // --- 7. Events ---
        let timeoutId;
        function onMapMove() {
            clearTimeout(timeoutId);
            timeoutId = setTimeout(() => fetchBenches(), 800);
        }

        map.on('moveend', onMapMove);
        map.on('zoomend', onMapMove);

        // Premier chargement
        fetchBenches();

    </script>
</body>
</html>

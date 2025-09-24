<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chargeur d'images avec Marqueurs</title>
    <!-- On importe Tailwind CSS pour un design moderne -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --main-marker-color: #f97316; /* Couleur par défaut (Orange vif) */
        }
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Conteneur pour l'image et les croix */
        #imageContainer {
            position: relative;
            display: inline-block; /* S'ajuste à la taille de l'image */
            max-width: 100%;
        }
        /* Style pour le marqueur déplaçable (croix + texte) */
        .draggable-marker {
            position: absolute;
            cursor: move;
            user-select: none; /* Empêche la sélection du texte pendant le glissement */
            -webkit-user-select: none;
            /* Centre le marqueur sur le point de positionnement et permet le changement de taille */
            transform: translate(-50%, -50%); 
            transform-origin: center center;
            transition: transform 0.1s ease-out, color 0.2s, background-color 0.2s;
            width: 24px; /* Taille explicite pour la croix */
            height: 24px;
            z-index: 5;
        }
        .draggable-marker.locked {
            cursor: default;
        }
        /* La croix géométrique à l'intérieur du marqueur */
        .marker-cross-h, .marker-cross-v {
            position: absolute;
            background-color: var(--main-marker-color); /* Utilise la variable de couleur */
            box-shadow: 0 0 2px white; /* Contour blanc pour la lisibilité */
        }
        .marker-cross-h {
            width: 100%;
            height: 2px;
            top: 50%;
            left: 0;
            transform: translateY(-50%);
        }
        .marker-cross-v {
            width: 2px;
            height: 100%;
            left: 50%;
            top: 0;
            transform: translateX(-50%);
        }
        .marker-label {
            /* On positionne le label de manière absolue pour ne pas affecter le centrage */
            position: absolute;
            left: 100%; /* A droite de la croix */
            top: 50%; /* Centré verticalement */
            transform: translateY(-50%); /* Ajustement vertical final */
            font-size: 14px;
            padding-left: 5px;
            font-weight: 500;
            white-space: nowrap; /* Empêche le nom de passer à la ligne */
            color: var(--main-marker-color); /* Utilise la variable de couleur */
            text-shadow: 0 0 3px white, 0 0 3px white;
        }
        /* Classe pour le contour du bouton actif, utilisant la variable de couleur */
        .active-marker-ring {
             --tw-ring-color: var(--main-marker-color);
        }
        /* Cache le label quand le marqueur est verrouillé */
        .draggable-marker.locked .marker-label {
            display: none;
        }
        /* Cache le bouton radio par défaut */
        .radio-selector {
            display: none;
        }
        /* Style pour le label qui sert de bouton radio custom */
        .radio-label {
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 1.5rem; /* 24px */
            height: 1.5rem; /* 24px */
            border: 2px solid #9CA3AF; /* gray-400 */
            border-radius: 9999px; /* full */
            cursor: pointer;
            transition: all 0.2s;
            font-size: 1.25rem; /* Taille de l'émoji */
            line-height: 1;
        }
        /* Style quand le bouton radio est sélectionné */
        .radio-selector:checked + .radio-label {
            background-color: transparent;
            border-color: transparent;
        }
        /* L'émoji couronne qui apparaît */
        .radio-selector:checked + .radio-label::before {
            content: '👑';
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }

        /* --- Style pour la loupe de zoom --- */
        #magnifier-glass {
            position: absolute;
            top: 10px;
            right: 10px;
            width: 150px;
            height: 150px;
            border: 3px solid #000;
            border-radius: 50%;
            overflow: hidden;
            pointer-events: none; /* Important! */
            z-index: 10;
            background-repeat: no-repeat;
            background-color: white;
        }
        .magnifier-crosshair-h, .magnifier-crosshair-v {
            position: absolute;
            background-color: rgba(255, 0, 0, 0.7);
            z-index: 11;
        }
        .magnifier-crosshair-h {
            top: 50%;
            left: 0;
            width: 100%;
            height: 1px;
            transform: translateY(-50%);
        }
        .magnifier-crosshair-v {
            left: 50%;
            top: 0;
            height: 100%;
            width: 1px;
            transform: translateX(-50%);
        }
        .lock-btn {
            background: none;
            border: none;
            cursor: pointer;
            padding: 0;
            line-height: 1;
            font-size: 1.25rem;
        }
        /* Styles pour la palette de couleurs */
        .color-swatch {
            transition: transform 0.1s ease-in-out, border-color 0.1s ease-in-out;
        }
        .color-swatch.selected {
            border-color: #3b82f6; /* Anneau bleu pour indiquer la sélection */
            transform: scale(1.15);
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div class="bg-white p-8 rounded-xl shadow-lg text-center max-w-2xl w-full">
        <!-- Titre de l'application -->
        <h1 class="text-2xl font-bold text-gray-800 mb-6">Easy Register (nom de la machine)</h1>

        <!-- Le bouton pour sélectionner une image -->
        <label for="imageUpload" class="inline-block bg-blue-500 text-white font-semibold py-3 px-6 rounded-lg cursor-pointer hover:bg-blue-600 transition-colors duration-300 ease-in-out shadow-md">
            Sélectionner une image
        </label>
        <input type="file" id="imageUpload" class="hidden" accept="image/*">

        <!-- La zone où l'image et les marqueurs seront affichés -->
        <div class="mt-8">
            <div id="imageContainer">
                <img id="imagePreview" src="" alt="Aperçu de l'image" class="max-w-full mx-auto rounded-lg shadow-md"/>
                 <!-- La loupe de zoom, cachée par défaut -->
                <div id="magnifier-glass" class="hidden">
                    <div class="magnifier-crosshair-h"></div>
                    <div class="magnifier-crosshair-v"></div>
                </div>
            </div>
        </div>
        
        <!-- Conteneur pour les boutons, caché par défaut -->
        <div id="controlsContainer" class="mt-6 hidden">
             <p class="text-sm text-gray-600 mb-3">Cliquez pour ajouter/supprimer. Sélectionnez une couronne pour définir la référence.</p>
            <div class="flex justify-center space-x-2 flex-wrap gap-y-2">
                <!-- Groupes F1-D -->
                <div class="flex flex-col items-center space-y-1"><button class="lock-btn" data-target-marker="F1">🔓</button><button data-marker-id="F1" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F1</button><input type="radio" name="marker-select" id="radio-F1" data-target-button="F1" class="radio-selector"><label for="radio-F1" class="radio-label"></label></div>
                <div class="flex flex-col items-center space-y-1"><button class="lock-btn" data-target-marker="F2">🔓</button><button data-marker-id="F2" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F2</button><input type="radio" name="marker-select" id="radio-F2" data-target-button="F2" class="radio-selector"><label for="radio-F2" class="radio-label"></label></div>
                <div class="flex flex-col items-center space-y-1"><button class="lock-btn" data-target-marker="F3">🔓</button><button data-marker-id="F3" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F3</button><input type="radio" name="marker-select" id="radio-F3" data-target-button="F3" class="radio-selector"><label for="radio-F3" class="radio-label"></label></div>
                <div class="flex flex-col items-center space-y-1"><button class="lock-btn" data-target-marker="F4">🔓</button><button data-marker-id="F4" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F4</button><input type="radio" name="marker-select" id="radio-F4" data-target-button="F4" class="radio-selector"><label for="radio-F4" class="radio-label"></label></div>
                <div class="flex flex-col items-center space-y-1"><button class="lock-btn" data-target-marker="F5">🔓</button><button data-marker-id="F5" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F5</button><input type="radio" name="marker-select" id="radio-F5" data-target-button="F5" class="radio-selector"><label for="radio-F5" class="radio-label"></label></div>
                <div class="flex flex-col items-center space-y-1"><button class="lock-btn" data-target-marker="F6">🔓</button><button data-marker-id="F6" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F6</button><input type="radio" name="marker-select" id="radio-F6" data-target-button="F6" class="radio-selector"><label for="radio-F6" class="radio-label"></label></div>
                <div class="flex flex-col items-center space-y-1"><button class="lock-btn" data-target-marker="D">🔓</button><button data-marker-id="D" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">D</button><input type="radio" name="marker-select" id="radio-D" data-target-button="D" class="radio-selector"><label for="radio-D" class="radio-label"></label></div>
            </div>
            
            <!-- Conteneur pour les boutons d'action -->
            <div class="mt-6 flex justify-center space-x-4">
                <button id="calculateBtn" class="bg-green-500 text-white font-semibold py-2 px-5 rounded-lg hover:bg-green-600 transition-colors">Calculer les corrections</button>
                <button id="optionsBtn" class="bg-gray-500 text-white font-semibold py-2 px-5 rounded-lg hover:bg-gray-600 transition-colors">Options</button>
            </div>
        </div>
        
        <!-- Conteneur pour afficher les calculs de correction, maintenant caché par défaut -->
        <div id="correctionsContainer" class="mt-6 text-left text-sm font-mono bg-gray-50 p-4 rounded-lg border hidden">
            <!-- Les résultats s'afficheront ici -->
        </div>
    </div>

    <!-- Fenêtre Modale pour les Options -->
    <div id="optionsModalBackdrop" class="fixed inset-0 bg-black bg-opacity-50 hidden z-40"></div>
    <div id="optionsModal" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl hidden z-50 w-full max-w-sm">
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold text-gray-800">Options</h2>
            <button id="closeModalBtn" class="text-gray-500 hover:text-gray-800 text-3xl leading-none">&times;</button>
        </div>
        <div class="space-y-6">
            <div>
                <label for="markerSizeSlider" class="block text-sm font-medium text-gray-700 mb-2">Taille des marqueurs: <span id="sliderValueSpan">5</span></label>
                <input type="range" id="markerSizeSlider" min="1" max="10" value="5" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer">
            </div>
             <!-- NOUVEAU: Palette de couleurs -->
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Couleur des éléments</label>
                <div id="color-palette" class="grid grid-cols-5 gap-3 justify-items-center">
                    <!-- Les pastilles de couleur seront injectées ici par le JS -->
                </div>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const imageUploadInput = document.getElementById('imageUpload');
            const imagePreviewElement = document.getElementById('imagePreview');
            const imageContainer = document.getElementById('imageContainer');
            const controlsContainer = document.getElementById('controlsContainer');
            const markerButtons = document.querySelectorAll('.marker-btn');
            const radioSelectors = document.querySelectorAll('.radio-selector');
            const correctionsContainer = document.getElementById('correctionsContainer');
            const calculateBtn = document.getElementById('calculateBtn');
            const lockButtons = document.querySelectorAll('.lock-btn');
            
            const optionsBtn = document.getElementById('optionsBtn');
            const optionsModal = document.getElementById('optionsModal');
            const optionsModalBackdrop = document.getElementById('optionsModalBackdrop');
            const closeModalBtn = document.getElementById('closeModalBtn');
            const markerSizeSlider = document.getElementById('markerSizeSlider');
            const sliderValueSpan = document.getElementById('sliderValueSpan');
            
            const magnifierGlass = document.getElementById('magnifier-glass');
            const zoomLevel = 4;

            let activeMarker = null;
            
            // --- GESTION DES COULEURS ---
            const colorPaletteContainer = document.getElementById('color-palette');
            const colors = [
                '#f97316', '#ef4444', '#3b82f6', '#22c55e', '#8b5cf6',
                '#ec4899', '#14b8a6', '#eab308', '#84cc16', '#1f2937'
            ];

            colors.forEach((color, index) => {
                const swatch = document.createElement('div');
                swatch.className = 'color-swatch w-8 h-8 rounded-full cursor-pointer border-2 border-transparent';
                swatch.style.backgroundColor = color;
                swatch.dataset.color = color;
                if (index === 0) {
                    swatch.classList.add('selected'); // Sélectionne la première couleur par défaut
                }
                swatch.addEventListener('click', () => {
                    // Met à jour la variable CSS globale
                    document.documentElement.style.setProperty('--main-marker-color', color);
                    
                    // Met à jour l'indicateur visuel de sélection
                    document.querySelectorAll('.color-swatch').forEach(s => s.classList.remove('selected'));
                    swatch.classList.add('selected');
                });
                colorPaletteContainer.appendChild(swatch);
            });

            imageUploadInput.addEventListener('change', function(event) {
                // 1. Sauvegarder la position du master s'il existe
                let savedMasterId = null;
                let savedPosition = null;
                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (selectedRadio) {
                    savedMasterId = selectedRadio.dataset.targetButton;
                    const mainMarker = document.querySelector(`.draggable-marker[data-marker-id="${savedMasterId}"]`);
                    if (mainMarker) {
                        savedPosition = {
                            left: mainMarker.style.left,
                            top: mainMarker.style.top
                        };
                    }
                }

                // 2. Réinitialiser l'interface
                // Supprimer tous les marqueurs
                document.querySelectorAll('.draggable-marker').forEach(marker => marker.remove());
                
                // Réinitialiser les boutons de contrôle
                markerButtons.forEach(btn => {
                    btn.classList.remove('bg-yellow-400', 'border-yellow-500', 'text-white', 'ring-2', 'active-marker-ring');
                    btn.classList.add('bg-gray-200', 'hover:bg-gray-300', 'text-gray-800');
                });
                
                // Décocher tous les boutons radio
                radioSelectors.forEach(radio => radio.checked = false);

                // Réinitialiser les cadenas
                lockButtons.forEach(lock => lock.textContent = '🔓');
                
                // Cacher le rapport de corrections
                correctionsContainer.innerHTML = '';
                correctionsContainer.classList.add('hidden');

                // 3. Charger la nouvelle image
                if (event.target.files && event.target.files[0]) {
                    const file = event.target.files[0];
                    const imageURL = URL.createObjectURL(file);
                    imagePreviewElement.src = imageURL;
                    magnifierGlass.style.backgroundImage = `url('${imageURL}')`;

                    imagePreviewElement.onload = () => {
                        controlsContainer.classList.remove('hidden');

                        // 4. Recréer le master s'il était sauvegardé
                        if (savedMasterId && savedPosition) {
                            createMarker(savedMasterId);
                            const newMasterMarker = document.querySelector(`.draggable-marker[data-marker-id="${savedMasterId}"]`);
                            if (newMasterMarker) {
                                newMasterMarker.style.left = savedPosition.left;
                                newMasterMarker.style.top = savedPosition.top;
                            }
                            
                            // 5. Sélectionner visuellement le master à nouveau
                            const masterRadio = document.querySelector(`#radio-${savedMasterId}`);
                            if (masterRadio) {
                                masterRadio.checked = true;
                                const masterButton = document.querySelector(`.marker-btn[data-marker-id="${savedMasterId}"]`);
                                 if (masterButton) {
                                    masterButton.classList.add('bg-yellow-400', 'border-yellow-500', 'text-white');
                                    masterButton.classList.remove('bg-gray-200', 'hover:bg-gray-300', 'text-gray-800');
                                }
                            }
                        }
                    };
                }
            });
            
            markerButtons.forEach(button => {
                button.addEventListener('click', () => {
                    const markerId = button.dataset.markerId;
                    const existingMarker = document.querySelector(`.draggable-marker[data-marker-id="${markerId}"]`);
                    if (existingMarker) {
                        existingMarker.remove();
                        button.classList.remove('ring-2', 'active-marker-ring'); // Désactive le contour
                        const lockButton = document.querySelector(`.lock-btn[data-target-marker="${markerId}"]`);
                        if (lockButton) {
                            lockButton.textContent = '🔓';
                        }
                    } else {
                        createMarker(markerId);
                    }
                });
            });

            lockButtons.forEach(button => {
                button.addEventListener('click', (event) => {
                    const button = event.currentTarget;
                    const markerId = button.dataset.targetMarker;
                    const marker = document.querySelector(`.draggable-marker[data-marker-id="${markerId}"]`);
                    if (!marker) return;

                    const isLocked = marker.classList.contains('locked');
                    if (isLocked) {
                        marker.classList.remove('locked');
                        button.textContent = '🔓';
                    } else {
                        marker.classList.add('locked');
                        button.textContent = '🔒';
                    }
                });
            });

            radioSelectors.forEach(radio => {
                radio.addEventListener('change', (event) => {
                    markerButtons.forEach(btn => {
                        btn.classList.remove('bg-yellow-400', 'border-yellow-500', 'text-white');
                        btn.classList.add('bg-gray-200', 'hover:bg-gray-300', 'text-gray-800');
                    });
                    if (event.target.checked) {
                        const targetButtonId = event.target.dataset.targetButton;
                        const targetButton = document.querySelector(`.marker-btn[data-marker-id="${targetButtonId}"]`);
                        if (targetButton) {
                            targetButton.classList.add('bg-yellow-400', 'border-yellow-500', 'text-white');
                            targetButton.classList.remove('bg-gray-200', 'hover:bg-gray-300', 'text-gray-800');
                        }
                    }
                });
            });
            
            calculateBtn.addEventListener('click', updateCorrections);

            function createMarker(id) {
                const marker = document.createElement('div');
                marker.className = 'draggable-marker';
                marker.dataset.markerId = id;
                marker.innerHTML = `<div class="marker-cross-h"></div><div class="marker-cross-v"></div><span class="marker-label">${id}</span>`;
                
                const imageRect = imagePreviewElement.getBoundingClientRect();
                const containerRect = imageContainer.getBoundingClientRect();
                const imageOffsetX = imageRect.left - containerRect.left;
                const imageOffsetY = imageRect.top - containerRect.top;
                marker.style.left = `${imageOffsetX + (imageRect.width * 0.05)}px`; 
                marker.style.top = `${imageOffsetY + (imageRect.height * 0.95)}px`;

                const currentSliderValue = markerSizeSlider.value;
                const scale = calculateScale(currentSliderValue);
                marker.style.transform = `translate(-50%, -50%) scale(${scale})`;
                imageContainer.appendChild(marker);
                marker.addEventListener('mousedown', dragStart);
                marker.addEventListener('touchstart', dragStart, { passive: false });
                
                // Met en évidence le bouton correspondant
                const correspondingButton = document.querySelector(`.marker-btn[data-marker-id="${id}"]`);
                if (correspondingButton) {
                   correspondingButton.classList.add('ring-2', 'active-marker-ring');
                }

                const lockButton = document.querySelector(`.lock-btn[data-target-marker="${id}"]`);
                if (lockButton) {
                    lockButton.textContent = '🔓';
                }
            }
            
            function updateCorrections() {
                correctionsContainer.innerHTML = '';
                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (!selectedRadio) {
                    correctionsContainer.innerHTML = '<p class="text-gray-500">Sélectionnez une référence (couronne) pour voir les calculs.</p>';
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainMarkerId = selectedRadio.dataset.targetButton;
                const mainMarker = document.querySelector(`.draggable-marker[data-marker-id="${mainMarkerId}"]`);
                if (!mainMarker) {
                    correctionsContainer.innerHTML = `<p class="text-gray-500">Ajoutez le marqueur de référence ${mainMarkerId} sur l'image.</p>`;
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainX = parseFloat(mainMarker.style.left);
                const mainY = parseFloat(mainMarker.style.top);
                const otherMarkers = document.querySelectorAll('.draggable-marker:not([data-marker-id="' + mainMarkerId + '"])');
                if (otherMarkers.length === 0) {
                     correctionsContainer.innerHTML = '<p class="text-gray-500">Ajoutez d\'autres marqueurs pour calculer les corrections.</p>';
                     correctionsContainer.classList.remove('hidden');
                     return;
                }
                otherMarkers.forEach(marker => {
                    const secondaryId = marker.dataset.markerId;
                    const secondaryX = parseFloat(marker.style.left);
                    const secondaryY = parseFloat(marker.style.top);
                    const correctionX = mainX - secondaryX;
                    const correctionY = mainY - secondaryY;
                    const resultLine = document.createElement('p');
                    
                    let displayName = secondaryId;
                    if (secondaryId === 'D') {
                        displayName = 'Die Cutter';
                    }

                    resultLine.textContent = `${displayName} : Registre (${correctionY.toFixed(2)}px) / Centrage (${correctionX.toFixed(2)}px)`;
                    correctionsContainer.appendChild(resultLine);
                });
                correctionsContainer.classList.remove('hidden');
            }

            // --- Logique pour la modale et le slider ---
            optionsBtn.addEventListener('click', () => {
                optionsModal.classList.remove('hidden');
                optionsModalBackdrop.classList.remove('hidden');
            });

            function closeModal() {
                optionsModal.classList.add('hidden');
                optionsModalBackdrop.classList.add('hidden');
            }
            closeModalBtn.addEventListener('click', closeModal);
            optionsModalBackdrop.addEventListener('click', closeModal);

            markerSizeSlider.addEventListener('input', (event) => {
                const value = event.target.value;
                sliderValueSpan.textContent = value;
                updateMarkerSizes(value);
            });



            function calculateScale(value) {
                value = parseInt(value);
                let scale;
                if (value >= 5) {
                    scale = 1 + (value - 5) * (3 / 5);
                } else {
                    scale = 0.25 + (value - 1) * (0.75 / 4);
                }
                return scale;
            }

            function updateMarkerSizes(value) {
                const scale = calculateScale(value);
                const markers = document.querySelectorAll('.draggable-marker');
                markers.forEach(marker => {
                    marker.style.transform = `translate(-50%, -50%) scale(${scale})`;
                });
            }

            // --- Fonctions pour la loupe de zoom ---
            function updateMagnifier(x_on_image, y_on_image) {
                const imageRect = imagePreviewElement.getBoundingClientRect();
                const glassWidth = magnifierGlass.offsetWidth;
                const glassHeight = magnifierGlass.offsetHeight;
                magnifierGlass.style.backgroundSize = `${imageRect.width * zoomLevel}px ${imageRect.height * zoomLevel}px`;
                let bgX = -(x_on_image * zoomLevel - glassWidth / 2);
                let bgY = -(y_on_image * zoomLevel - glassHeight / 2);
                magnifierGlass.style.backgroundPosition = `${bgX}px ${bgY}px`;
            }

            // --- Fonctions pour le glisser-déposer ---
            function dragStart(e) {
                activeMarker = e.target.closest('.draggable-marker');
                if (!activeMarker || activeMarker.classList.contains('locked')) {
                    activeMarker = null;
                    return;
                }
                e.preventDefault();
                magnifierGlass.classList.remove('hidden');
                const imageRect = imagePreviewElement.getBoundingClientRect();
                let clientX, clientY;
                if (e.type.startsWith("touch")) {
                    clientX = e.touches[0].clientX;
                    clientY = e.touches[0].clientY;
                } else {
                    clientX = e.clientX;
                    clientY = e.clientY;
                }
                let cursorX_on_image = clientX - imageRect.left;
                let cursorY_on_image = clientY - imageRect.top;
                cursorX_on_image = Math.max(0, Math.min(cursorX_on_image, imageRect.width));
                cursorY_on_image = Math.max(0, Math.min(cursorY_on_image, imageRect.height));
                updateMagnifier(cursorX_on_image, cursorY_on_image);
            }

            function dragEnd() {
                if(activeMarker) {
                    magnifierGlass.classList.add('hidden');
                }
                activeMarker = null;
            }

            function drag(e) {
                if (!activeMarker) return;
                e.preventDefault(); 
                const containerRect = imageContainer.getBoundingClientRect();
                const imageRect = imagePreviewElement.getBoundingClientRect();
                let clientX, clientY;
                if (e.type.startsWith("touch")) {
                    clientX = e.touches[0].clientX;
                    clientY = e.touches[0].clientY;
                } else {
                    clientX = e.clientX;
                    clientY = e.clientY;
                }
                let cursorX_on_image = clientX - imageRect.left;
                let cursorY_on_image = clientY - imageRect.top;
                cursorX_on_image = Math.max(0, Math.min(cursorX_on_image, imageRect.width));
                cursorY_on_image = Math.max(0, Math.min(cursorY_on_image, imageRect.height));
                const imageOffsetX = imageRect.left - containerRect.left;
                const imageOffsetY = imageRect.top - containerRect.top;
                let markerX = imageOffsetX + cursorX_on_image;
                let markerY = imageOffsetY + cursorY_on_image;
                activeMarker.style.left = `${markerX}px`;
                activeMarker.style.top = `${markerY}px`;
                updateMagnifier(cursorX_on_image, cursorY_on_image);
            }

            document.addEventListener('mouseup', dragEnd);
            document.addEventListener('touchend', dragEnd);
            document.addEventListener('mousemove', drag);
            document.addEventListener('touchmove', drag, { passive: false });
        });
    </script>

</body>
</html>


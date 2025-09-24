<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale-1.0">
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
        .draggable-marker.locked .marker-label {
            display: none;
        }
        /* Classe pour le contour du bouton actif, utilisant la variable de couleur */
        .active-marker-ring {
             --tw-ring-color: var(--main-marker-color);
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
            transition: left 0.2s ease-in-out, right 0.2s ease-in-out, top 0.2s ease-in-out, bottom 0.2s ease-in-out;
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
        /* Styles pour la palette de couleurs */
        .color-swatch {
            transition: transform 0.1s ease-in-out, border-color 0.1s ease-in-out;
        }
        .color-swatch.selected {
            border-color: #3b82f6; /* Anneau bleu pour indiquer la sélection */
            transform: scale(1.15);
        }
        /* Style pour la poubelle */
        #trashCan {
            position: absolute;
            top: 0.5rem;
            left: 0.5rem;
            width: 3rem;
            height: 3rem;
            background-color: rgba(255, 255, 255, 0.8);
            border-radius: 0.5rem;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5rem;
            transition: transform 0.2s, background-color 0.2s;
            z-index: 6; /* Au-dessus des marqueurs */
            cursor: pointer; /* Rendu cliquable */
        }
        #trashCan.hovering {
            transform: scale(1.1);
            background-color: #fecaca; /* red-200 */
        }
        /* Style pour la croix directionnelle (D-Pad) */
        #dpad-controls {
            position: absolute;
            bottom: 0.5rem;
            right: 0.5rem;
            width: 6rem; /* 96px */
            height: 6rem; /* 96px */
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            grid-template-rows: 1fr 1fr 1fr;
            z-index: 7;
            background-color: rgba(255, 255, 255, 0.5);
            border-radius: 50%;
            transition: background-color 0.2s, left 0.2s ease-in-out, right 0.2s ease-in-out, top 0.2s ease-in-out, bottom 0.2s ease-in-out;
        }
        .dpad-btn {
            background-color: rgba(243, 244, 246, 0.8); /* gray-100 */
            border: 1px solid rgba(209, 213, 219, 0.9); /* gray-300 */
            font-size: 1.25rem;
            font-weight: bold;
            color: #374151; /* gray-700 */
            transition: background-color 0.15s;
        }
        .dpad-btn:active {
            background-color: #d1d5db; /* gray-300 */
        }
        #dpad-up { grid-area: 1 / 2 / 2 / 3; border-radius: 10px 10px 0 0;}
        #dpad-left { grid-area: 2 / 1 / 3 / 2; border-radius: 10px 0 0 10px;}
        #dpad-right { grid-area: 2 / 3 / 3 / 4; border-radius: 0 10px 10px 0;}
        #dpad-down { grid-area: 3 / 2 / 4 / 3; border-radius: 0 0 10px 10px;}
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div class="bg-white p-8 rounded-xl shadow-lg text-center max-w-2xl w-full">
        <!-- Titre de l'application -->
        <h1 class="text-2xl font-bold text-gray-800 mb-6">Easy Register (nom de la machine)</h1>

        <!-- Conteneur pour les boutons du haut -->
        <div class="flex justify-center items-center space-x-4 mb-6">
            <!-- Le bouton pour sélectionner une image -->
            <label for="imageUpload" class="inline-block bg-blue-500 text-white font-semibold py-3 px-6 rounded-lg cursor-pointer hover:bg-blue-600 transition-colors duration-300 ease-in-out shadow-md">
                Sélectionner une image
            </label>
            <input type="file" id="imageUpload" class="hidden" accept="image/*">
            <!-- Le bouton Options -->
            <button id="optionsBtn" class="bg-gray-500 text-white font-semibold py-3 px-6 rounded-lg hover:bg-gray-600 transition-colors">Options</button>
        </div>

        <!-- La zone où l'image et les marqueurs seront affichés, cachée par défaut -->
        <div id="imageDisplayArea" class="mt-8 hidden">
            <div id="imageContainer">
                <img id="imagePreview" alt="Aperçu de l'image" class="max-w-full mx-auto rounded-lg shadow-md"/>
                <!-- La poubelle pour supprimer les marqueurs -->
                <div id="trashCan">🗑️</div>
                 <!-- La loupe de zoom, cachée par défaut -->
                <div id="magnifier-glass" class="hidden">
                    <div class="magnifier-crosshair-h"></div>
                    <div class="magnifier-crosshair-v"></div>
                </div>
                 <!-- La croix directionnelle (D-Pad) -->
                <div id="dpad-controls">
                    <button id="dpad-up" class="dpad-btn">↑</button>
                    <button id="dpad-left" class="dpad-btn">←</button>
                    <button id="dpad-right" class="dpad-btn">→</button>
                    <button id="dpad-down" class="dpad-btn">↓</button>
                </div>
            </div>
        </div>
        
        <!-- Conteneur pour les boutons, caché par défaut -->
        <div id="controlsContainer" class="mt-6 hidden">
             <p class="text-sm text-gray-600 mb-3">Cliquez pour ajouter/verrouiller une fléxo et définir la référence.</p>
            <div class="flex justify-center items-start space-x-3 flex-wrap gap-y-2">
                <!-- Groupes F1-D -->
                <div class="flex flex-col items-center space-y-2"><button data-marker-id="F1" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F1</button><div><input type="radio" name="marker-select" id="radio-F1" data-target-button="F1" class="radio-selector"><label for="radio-F1" class="radio-label"></label></div></div>
                <div class="flex flex-col items-center space-y-2"><button data-marker-id="F2" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F2</button><div><input type="radio" name="marker-select" id="radio-F2" data-target-button="F2" class="radio-selector"><label for="radio-F2" class="radio-label"></label></div></div>
                <div class="flex flex-col items-center space-y-2"><button data-marker-id="F3" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F3</button><div><input type="radio" name="marker-select" id="radio-F3" data-target-button="F3" class="radio-selector"><label for="radio-F3" class="radio-label"></label></div></div>
                <div class="flex flex-col items-center space-y-2"><button data-marker-id="F4" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F4</button><div><input type="radio" name="marker-select" id="radio-F4" data-target-button="F4" class="radio-selector"><label for="radio-F4" class="radio-label"></label></div></div>
                <div class="flex flex-col items-center space-y-2"><button data-marker-id="F5" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F5</button><div><input type="radio" name="marker-select" id="radio-F5" data-target-button="F5" class="radio-selector"><label for="radio-F5" class="radio-label"></label></div></div>
                <div class="flex flex-col items-center space-y-2"><button data-marker-id="F6" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F6</button><div><input type="radio" name="marker-select" id="radio-F6" data-target-button="F6" class="radio-selector"><label for="radio-F6" class="radio-label"></label></div></div>
                <div class="flex flex-col items-center space-y-2"><button data-marker-id="D" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">D</button><div><input type="radio" name="marker-select" id="radio-D" data-target-button="D" class="radio-selector"><label for="radio-D" class="radio-label"></label></div></div>
            </div>
        </div>
        
        <!-- Conteneur pour afficher les calculs de correction et le bouton d'envoi -->
        <div id="correctionsContainer" class="mt-6 text-left bg-gray-50 p-4 rounded-lg border hidden">
            <div id="correctionsText" class="text-sm font-mono">
                <!-- Les résultats s'afficheront ici -->
            </div>
            <button id="sendToScreenBtn" class="hidden mt-4 w-full bg-indigo-600 text-white font-semibold py-2 px-5 rounded-lg hover:bg-indigo-700 transition-colors">Envoyer à l'écran</button>
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
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Couleur des éléments</label>
                <div id="color-palette" class="grid grid-cols-5 gap-3 justify-items-center">
                    <!-- Les pastilles de couleur seront injectées ici par le JS -->
                </div>
            </div>
            <div>
                <button id="settingsBtn" class="w-full bg-gray-200 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-300 transition-colors">Paramètres</button>
            </div>
        </div>
    </div>

    <!-- Fenêtre Modale de Confirmation -->
    <div id="confirmationModalBackdrop" class="fixed inset-0 bg-black bg-opacity-50 hidden z-40"></div>
    <div id="confirmationModal" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl hidden z-50 w-full max-w-sm text-center">
        <h2 class="text-lg font-bold text-gray-800 mb-4">Confirmation</h2>
        <p class="text-gray-600 mb-6">Êtes-vous sûr de vouloir envoyer ces valeurs à l'écran ?</p>
        <div class="flex flex-col space-y-3">
            <div class="flex justify-center space-x-4">
                <button id="cancelSendBtn" class="bg-gray-300 text-gray-800 font-semibold py-2 px-6 rounded-lg hover:bg-gray-400 transition-colors">Non</button>
                <button id="confirmSendBtn" class="bg-green-500 text-white font-semibold py-2 px-6 rounded-lg hover:bg-green-600 transition-colors">Oui</button>
            </div>
            <button id="stopInputBtn" class="bg-red-600 text-white font-semibold py-2 px-6 rounded-lg hover:bg-red-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed">Arrêter la saisie</button>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const imageUploadInput = document.getElementById('imageUpload');
            const imagePreviewElement = document.getElementById('imagePreview');
            const imageDisplayArea = document.getElementById('imageDisplayArea');
            const imageContainer = document.getElementById('imageContainer');
            const controlsContainer = document.getElementById('controlsContainer');
            const markerButtons = document.querySelectorAll('.marker-btn');
            const radioSelectors = document.querySelectorAll('.radio-selector');
            const correctionsContainer = document.getElementById('correctionsContainer');
            const correctionsText = document.getElementById('correctionsText');
            const sendToScreenBtn = document.getElementById('sendToScreenBtn');
            const trashCan = document.getElementById('trashCan');
            
            const optionsBtn = document.getElementById('optionsBtn');
            const optionsModal = document.getElementById('optionsModal');
            const optionsModalBackdrop = document.getElementById('optionsModalBackdrop');
            const closeModalBtn = document.getElementById('closeModalBtn');
            const markerSizeSlider = document.getElementById('markerSizeSlider');
            const sliderValueSpan = document.getElementById('sliderValueSpan');
            
            const confirmationModalBackdrop = document.getElementById('confirmationModalBackdrop');
            const confirmationModal = document.getElementById('confirmationModal');
            const cancelSendBtn = document.getElementById('cancelSendBtn');
            const confirmSendBtn = document.getElementById('confirmSendBtn');
            const stopInputBtn = document.getElementById('stopInputBtn');
            
            const magnifierGlass = document.getElementById('magnifier-glass');
            const zoomLevel = 4;

            const dpadControls = document.getElementById('dpad-controls');
            const dpadUp = document.getElementById('dpad-up');
            const dpadDown = document.getElementById('dpad-down');
            const dpadLeft = document.getElementById('dpad-left');
            const dpadRight = document.getElementById('dpad-right');
            
            let activeMarker = null;
            let lastSelectedMarker = null; 
            let magnifierHideTimer = null;
            let lastValidPosition = null;
            
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
                    swatch.classList.add('selected');
                }
                swatch.addEventListener('click', () => {
                    document.documentElement.style.setProperty('--main-marker-color', color);
                    document.querySelectorAll('.color-swatch').forEach(s => s.classList.remove('selected'));
                    swatch.classList.add('selected');
                });
                colorPaletteContainer.appendChild(swatch);
            });

            imageUploadInput.addEventListener('change', function(event) {
                let savedMasterId = null;
                let savedPosition = null;
                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (selectedRadio) {
                    savedMasterId = selectedRadio.dataset.targetButton;
                    const mainMarker = document.querySelector(`.draggable-marker[data-marker-id="${savedMasterId}"]`);
                    if (mainMarker) {
                        savedPosition = { left: mainMarker.style.left, top: mainMarker.style.top };
                    }
                }
                resetSession(); // On réinitialise la session avant de charger la nouvelle image
                
                if (event.target.files && event.target.files[0]) {
                    const file = event.target.files[0];
                    const imageURL = URL.createObjectURL(file);
                    imagePreviewElement.src = imageURL;
                    magnifierGlass.style.backgroundImage = `url('${imageURL}')`;
                    imagePreviewElement.onload = () => {
                        imageDisplayArea.classList.remove('hidden'); 
                        controlsContainer.classList.remove('hidden');
                        if (savedMasterId && savedPosition) {
                            createMarker(savedMasterId);
                            const newMasterMarker = document.querySelector(`.draggable-marker[data-marker-id="${savedMasterId}"]`);
                            if (newMasterMarker) {
                                newMasterMarker.style.left = savedPosition.left;
                                newMasterMarker.style.top = savedPosition.top;
                                lastSelectedMarker = newMasterMarker;
                            }
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
                        existingMarker.classList.toggle('locked');
                    } else {
                        createMarker(markerId);
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
                    updateCorrections();
                });
            });
            
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
                
                const correspondingButton = document.querySelector(`.marker-btn[data-marker-id="${id}"]`);
                if (correspondingButton) {
                   correspondingButton.classList.add('ring-2', 'active-marker-ring');
                }
                updateCorrections();
            }
            
            function updateCorrections() {
                correctionsText.innerHTML = '';
                sendToScreenBtn.classList.add('hidden');

                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (!selectedRadio) {
                    correctionsText.innerHTML = '<p class="text-gray-500">Sélectionnez une référence (couronne) pour voir les calculs.</p>';
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainMarkerId = selectedRadio.dataset.targetButton;
                const mainMarker = document.querySelector(`.draggable-marker[data-marker-id="${mainMarkerId}"]`);
                if (!mainMarker) {
                    correctionsText.innerHTML = `<p class="text-gray-500">Ajoutez le marqueur de référence ${mainMarkerId} sur l'image.</p>`;
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainX = parseFloat(mainMarker.style.left);
                const mainY = parseFloat(mainMarker.style.top);
                const otherMarkers = document.querySelectorAll('.draggable-marker:not([data-marker-id="' + mainMarkerId + '"])');
                if (otherMarkers.length === 0) {
                     correctionsText.innerHTML = '<p class="text-gray-500">Ajoutez d\'autres marqueurs pour calculer les corrections.</p>';
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
                    correctionsText.appendChild(resultLine);
                });
                sendToScreenBtn.classList.remove('hidden');
                correctionsContainer.classList.remove('hidden');
            }

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
            
            function resetSession() {
                const allMarkers = document.querySelectorAll('.draggable-marker');
                if (allMarkers.length > 0) {
                    allMarkers.forEach(marker => marker.remove());
                }
                markerButtons.forEach(btn => {
                    btn.classList.remove('ring-2', 'active-marker-ring', 'bg-yellow-400', 'border-yellow-500', 'text-white');
                    btn.classList.add('bg-gray-200', 'hover:bg-gray-300', 'text-gray-800');
                });
                radioSelectors.forEach(radio => radio.checked = false);
                lastSelectedMarker = null;
                updateCorrections();
            }
            trashCan.addEventListener('click', resetSession);

            // --- Logique de la modale de confirmation ---
            sendToScreenBtn.addEventListener('click', () => {
                // Griser le bouton "Arrêter la saisie" à l'ouverture
                stopInputBtn.disabled = true;
                confirmationModal.classList.remove('hidden');
                confirmationModalBackdrop.classList.remove('hidden');
            });

            function closeConfirmationModal() {
                confirmationModal.classList.add('hidden');
                confirmationModalBackdrop.classList.add('hidden');
            }

            cancelSendBtn.addEventListener('click', closeConfirmationModal);
            confirmationModalBackdrop.addEventListener('click', closeConfirmationModal);

            confirmSendBtn.addEventListener('click', () => {
                // Ne PAS fermer la modale
                // Activer le bouton "Arrêter la saisie"
                stopInputBtn.disabled = false;
                
                console.log('Valeurs envoyées !'); // Action simulée
                
                // Donner le retour visuel sur le bouton principal
                sendToScreenBtn.textContent = 'Envoyé !';
                sendToScreenBtn.classList.remove('bg-indigo-600', 'hover:bg-indigo-700');
                sendToScreenBtn.classList.add('bg-green-500');
                
                setTimeout(() => {
                    sendToScreenBtn.textContent = "Envoyer à l'écran";
                    sendToScreenBtn.classList.add('bg-indigo-600', 'hover:bg-indigo-700');
                    sendToScreenBtn.classList.remove('bg-green-500');
                }, 2000);
            });
            
            stopInputBtn.addEventListener('click', () => {
                if (stopInputBtn.disabled) return; // Sécurité supplémentaire
                
                // On ferme la modale D'ABORD
                closeConfirmationModal();
                
                console.log('Saisie arrêtée.');
                
                // Ensuite, on réinitialise la session
                resetSession();
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
                document.querySelectorAll('.draggable-marker').forEach(marker => {
                    marker.style.transform = `translate(-50%, -50%) scale(${scale})`;
                });
            }

            function updateMagnifier(x_on_image, y_on_image) {
                const imageRect = imagePreviewElement.getBoundingClientRect();
                const glassWidth = magnifierGlass.offsetWidth;
                const glassHeight = magnifierGlass.offsetHeight;
                magnifierGlass.style.backgroundSize = `${imageRect.width * zoomLevel}px ${imageRect.height * zoomLevel}px`;
                let bgX = -(x_on_image * zoomLevel - glassWidth / 2);
                let bgY = -(y_on_image * zoomLevel - glassHeight / 2);
                magnifierGlass.style.backgroundPosition = `${bgX}px ${bgY}px`;
            }

            function moveLastMarker(dx, dy) {
                if (!lastSelectedMarker || lastSelectedMarker.classList.contains('locked')) return;
                if (magnifierHideTimer) clearTimeout(magnifierHideTimer);
                const currentX = parseFloat(lastSelectedMarker.style.left);
                const currentY = parseFloat(lastSelectedMarker.style.top);
                lastSelectedMarker.style.left = `${currentX + dx}px`;
                lastSelectedMarker.style.top = `${currentY + dy}px`;
                const imageRect = imagePreviewElement.getBoundingClientRect();
                const containerRect = imageContainer.getBoundingClientRect();
                const imageOffsetX = imageRect.left - containerRect.left;
                const imageOffsetY = imageRect.top - containerRect.top;
                const x_on_image = (currentX + dx) - imageOffsetX;
                const y_on_image = (currentY + dy) - imageOffsetY;
                magnifierGlass.classList.remove('hidden');
                updateMagnifier(x_on_image, y_on_image);
                magnifierHideTimer = setTimeout(() => {
                    magnifierGlass.classList.add('hidden');
                }, 3000);
                updateCorrections();
            }

            dpadUp.addEventListener('click', () => moveLastMarker(0, -1));
            dpadDown.addEventListener('click', () => moveLastMarker(0, 1));
            dpadLeft.addEventListener('click', () => moveLastMarker(-1, 0));
            dpadRight.addEventListener('click', () => moveLastMarker(1, 0));

            function dragStart(e) {
                if (magnifierHideTimer) clearTimeout(magnifierHideTimer);
                activeMarker = e.target.closest('.draggable-marker');
                if (!activeMarker || activeMarker.classList.contains('locked')) {
                    activeMarker = null;
                    return;
                }
                lastValidPosition = { left: activeMarker.style.left, top: activeMarker.style.top };
                lastSelectedMarker = activeMarker;
                e.preventDefault();
                magnifierGlass.classList.remove('hidden');
                
                const imageRect = imagePreviewElement.getBoundingClientRect();
                let clientX = e.type.startsWith("touch") ? e.touches[0].clientX : e.clientX;
                let clientY = e.type.startsWith("touch") ? e.touches[0].clientY : e.clientY;
                let cursorX_on_image = Math.max(0, Math.min(clientX - imageRect.left, imageRect.width));
                let cursorY_on_image = Math.max(0, Math.min(clientY - imageRect.top, imageRect.height));
                updateMagnifier(cursorX_on_image, cursorY_on_image);
            }

            function dragEnd(e) {
                if(activeMarker) {
                    const markerRect = activeMarker.getBoundingClientRect();
                    const trashRect = trashCan.getBoundingClientRect();
                    const isOverTrash = checkCollision(markerRect, trashRect);
                   
                    if (isOverTrash) {
                        const markerId = activeMarker.dataset.markerId;
                        activeMarker.remove();
                        if (lastSelectedMarker === activeMarker) lastSelectedMarker = null;
                        const button = document.querySelector(`.marker-btn[data-marker-id="${markerId}"]`);
                        if (button) button.classList.remove('ring-2', 'active-marker-ring');
                        const radio = document.querySelector(`#radio-${markerId}`);
                        if (radio && radio.checked) {
                            radio.checked = false;
                            if (button) {
                                button.classList.remove('bg-yellow-400', 'border-yellow-500', 'text-white');
                                button.classList.add('bg-gray-200', 'hover:bg-gray-300', 'text-gray-800');
                            }
                        }
                    }

                    // Réinitialise la position des outils
                    resetToolPositions();

                    if (magnifierHideTimer) clearTimeout(magnifierHideTimer);
                    magnifierHideTimer = setTimeout(() => {
                        magnifierGlass.classList.add('hidden');
                    }, 3000);
                    
                    trashCan.classList.remove('hovering');
                    updateCorrections();
                }
                activeMarker = null;
                lastValidPosition = null;
            }

            function drag(e) {
                if (!activeMarker) return;
                e.preventDefault(); 
                
                const containerRect = imageContainer.getBoundingClientRect();
                const imageRect = imagePreviewElement.getBoundingClientRect();
                let clientX = e.type.startsWith("touch") ? e.touches[0].clientX : e.clientX;
                let clientY = e.type.startsWith("touch") ? e.touches[0].clientY : e.clientY;
                let cursorX_on_image = Math.max(0, Math.min(clientX - imageRect.left, imageRect.width));
                let cursorY_on_image = Math.max(0, Math.min(clientY - imageRect.top, imageRect.height));
                
                const imageOffsetX = imageRect.left - containerRect.left;
                const imageOffsetY = imageRect.top - containerRect.top;
                let markerX = imageOffsetX + cursorX_on_image;
                let markerY = imageOffsetY + cursorY_on_image;
                activeMarker.style.left = `${markerX}px`;
                activeMarker.style.top = `${markerY}px`;
                updateMagnifier(cursorX_on_image, cursorY_on_image);

                const markerRect = activeMarker.getBoundingClientRect();
                
                // --- Gestion Intelligente des Outils ---
                handleToolRepositioning(markerRect);
            }

            // --- NOUVELLES FONCTIONS D'AIDE ---
            
            // Fonction pour vérifier la collision entre deux rectangles
            function checkCollision(rect1, rect2) {
                return !(rect1.right < rect2.left || rect1.left > rect2.right || rect1.bottom < rect2.top || rect1.top > rect2.bottom);
            }
            
            // Fonction pour réinitialiser la position des outils
            function resetToolPositions() {
                magnifierGlass.style.top = '10px';
                magnifierGlass.style.bottom = 'auto';
                magnifierGlass.style.right = '10px';
                magnifierGlass.style.left = 'auto';

                dpadControls.style.top = 'auto';
                dpadControls.style.bottom = '0.5rem';
                dpadControls.style.right = '0.5rem';
                dpadControls.style.left = 'auto';
            }

            // Gère le repositionnement de la loupe et du D-pad
            function handleToolRepositioning(markerRect) {
                 const containerRect = imageContainer.getBoundingClientRect();
                 const margin = 10;

                // Définir les zones par défaut des outils par rapport au conteneur
                const glassDefaultRect = {
                    top: containerRect.top + margin,
                    left: containerRect.right - 150 - margin,
                    width: 150, height: 150
                };
                glassDefaultRect.right = glassDefaultRect.left + 150;
                glassDefaultRect.bottom = glassDefaultRect.top + 150;

                const dpadDefaultRect = {
                    top: containerRect.bottom - 96 - margin,
                    left: containerRect.right - 96 - margin,
                    width: 96, height: 96
                };
                dpadDefaultRect.right = dpadDefaultRect.left + 96;
                dpadDefaultRect.bottom = dpadDefaultRect.top + 96;

                // Gérer la poubelle
                const trashRect = trashCan.getBoundingClientRect();
                trashCan.classList.toggle('hovering', checkCollision(markerRect, trashRect));


                // Gérer la loupe
                if (checkCollision(markerRect, glassDefaultRect)) {
                    magnifierGlass.style.right = 'auto';
                    magnifierGlass.style.left = '10px';
                } else {
                    magnifierGlass.style.left = 'auto';
                    magnifierGlass.style.right = '10px';
                }
                
                // Gérer le D-Pad
                if (checkCollision(markerRect, dpadDefaultRect)) {
                    const obstacles = [
                        magnifierGlass.getBoundingClientRect(),
                        trashCan.getBoundingClientRect(),
                        ...Array.from(document.querySelectorAll('.draggable-marker:not([data-marker-id="' + activeMarker.dataset.markerId + '"])')).map(m => m.getBoundingClientRect())
                    ];

                    const safeCorner = findSafeCorner(dpadControls, obstacles);
                    
                    // Appliquer la nouvelle position
                    dpadControls.style.top = safeCorner.top;
                    dpadControls.style.bottom = safeCorner.bottom;
                    dpadControls.style.left = safeCorner.left;
                    dpadControls.style.right = safeCorner.right;
                } else {
                    // Revenir à la position par défaut
                    resetDpadPosition();
                }
            }
            
            function resetDpadPosition(){
                dpadControls.style.top = 'auto';
                dpadControls.style.bottom = '0.5rem';
                dpadControls.style.left = 'auto';
                dpadControls.style.right = '0.5rem';
            }


            // Trouve un coin sûr pour un élément
            function findSafeCorner(element, obstacles) {
                const elRect = element.getBoundingClientRect();
                const containerRect = imageContainer.getBoundingClientRect();
                const margin = 8; // 0.5rem

                // Liste des coins possibles [bottom-left, top-right, top-left] - on exclut bottom-right qui est la position par défaut
                const corners = [
                    { bottom: `${margin}px`, left: `${margin}px`, top: 'auto', right: 'auto' },
                    { top: `${margin}px`, right: `${margin}px`, bottom: 'auto', left: 'auto' },
                    { top: `${margin}px`, left: `${margin}px`, bottom: 'auto', right: 'auto' }
                ];
                
                for (const corner of corners) {
                    let potentialRect = {
                        width: elRect.width,
                        height: elRect.height,
                        top: corner.top !== 'auto' ? containerRect.top + margin : containerRect.bottom - elRect.height - margin,
                        left: corner.left !== 'auto' ? containerRect.left + margin : containerRect.right - elRect.width - margin
                    };
                    potentialRect.right = potentialRect.left + potentialRect.width;
                    potentialRect.bottom = potentialRect.top + potentialRect.height;
                    
                    let isSafe = true;
                    for (const obstacle of obstacles) {
                        if (checkCollision(potentialRect, obstacle)) {
                            isSafe = false;
                            break;
                        }
                    }
                    if (isSafe) return corner;
                }
                
                // Par défaut, si aucun n'est sûr, on retourne le premier coin testé.
                return corners[0];
            }


            document.addEventListener('mouseup', dragEnd);
            document.addEventListener('touchend', dragEnd);
            document.addEventListener('mousemove', drag);
            document.addEventListener('touchmove', drag, { passive: false });
        });
    </script>

</body>
</html>


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
        <h1 class="text-2xl font-bold text-gray-800 mb-6">Easy Register (<span id="machineNameDisplay">nom de la machine</span>)</h1>

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
            <div id="marker-buttons-wrapper" class="flex justify-center items-start space-x-3 flex-wrap gap-y-2">
                <!-- Les boutons de marqueurs seront générés ici par le JS -->
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
            <button id="closeOptionsModalBtn" class="text-gray-500 hover:text-gray-800 text-3xl leading-none">&times;</button>
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
                <button id="bluetoothSearchBtn" class="w-full bg-blue-500 text-white font-semibold py-2 px-4 rounded-lg hover:bg-blue-600 transition-colors flex items-center justify-center space-x-2">
                    <span id="bluetoothBtnText">Rechercher via Bluetooth</span>
                    <span id="bluetoothStatus" class="w-3 h-3 bg-red-500 rounded-full border-2 border-white"></span>
                </button>
            </div>
            <div>
                <button id="openSettingsBtn" class="w-full bg-gray-200 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-300 transition-colors">Paramètres</button>
            </div>
        </div>
    </div>
    
    <!-- Fenêtre Modale pour le Mot de Passe -->
    <div id="passwordModalBackdrop" class="fixed inset-0 bg-black bg-opacity-50 hidden z-40"></div>
    <div id="passwordModal" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl hidden z-50 w-full max-w-sm">
        <h2 class="text-xl font-bold text-gray-800 mb-4">Accès Protégé</h2>
        <p class="text-gray-600 mb-4">Veuillez entrer le mot de passe administrateur.</p>
        <input type="password" id="passwordInput" class="w-full border border-gray-300 rounded-lg px-3 py-2 mb-2">
        <p id="passwordError" class="text-red-500 text-sm h-4 mb-4"></p>
        <div class="flex justify-end space-x-3">
            <button id="cancelPasswordBtn" class="bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-400">Annuler</button>
            <button id="submitPasswordBtn" class="bg-blue-500 text-white font-semibold py-2 px-4 rounded-lg hover:bg-blue-600">Valider</button>
        </div>
    </div>

    <!-- Fenêtre Modale pour les Paramètres -->
    <div id="settingsModalBackdrop" class="fixed inset-0 bg-black bg-opacity-50 hidden z-40"></div>
    <div id="settingsModal" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl hidden z-50 w-full max-w-sm">
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold text-gray-800">Paramètres</h2>
            <button id="closeSettingsModalBtn" class="text-gray-500 hover:text-gray-800 text-3xl leading-none">&times;</button>
        </div>
        <div class="space-y-6">
            <div>
                <label for="machineNameInput" class="block text-sm font-medium text-gray-700 mb-2">Nom de la machine</label>
                <input type="text" id="machineNameInput" class="w-full border border-gray-300 rounded-lg px-3 py-2">
            </div>
             <div>
                <label for="flexoCountSlider" class="block text-sm font-medium text-gray-700 mb-2">Nombre de flexos (F): <span id="flexoCountValue">6</span></label>
                <input type="range" id="flexoCountSlider" min="1" max="9" value="6" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer">
            </div>
            <div>
                <label for="pixelToMmRatioInput" class="block text-sm font-medium text-gray-700 mb-2">Pixels par millimètre (px/mm)</label>
                <input type="number" id="pixelToMmRatioInput" value="10" class="w-full border border-gray-300 rounded-lg px-3 py-2">
            </div>
            <div>
                <label for="bluetoothDeviceNameInput" class="block text-sm font-medium text-gray-700 mb-2">Nom de l'appareil Bluetooth</label>
                <input type="text" id="bluetoothDeviceNameInput" placeholder="Ex: ESP32_Printer" class="w-full border border-gray-300 rounded-lg px-3 py-2">
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
            const markerButtonsWrapper = document.getElementById('marker-buttons-wrapper');
            const correctionsContainer = document.getElementById('correctionsContainer');
            const correctionsText = document.getElementById('correctionsText');
            const sendToScreenBtn = document.getElementById('sendToScreenBtn');
            const trashCan = document.getElementById('trashCan');
            
            // --- Modale Options ---
            const optionsBtn = document.getElementById('optionsBtn');
            const optionsModal = document.getElementById('optionsModal');
            const optionsModalBackdrop = document.getElementById('optionsModalBackdrop');
            const closeOptionsModalBtn = document.getElementById('closeOptionsModalBtn');
            const markerSizeSlider = document.getElementById('markerSizeSlider');
            const sliderValueSpan = document.getElementById('sliderValueSpan');
            
            // --- Modale Confirmation ---
            const confirmationModalBackdrop = document.getElementById('confirmationModalBackdrop');
            const confirmationModal = document.getElementById('confirmationModal');
            const cancelSendBtn = document.getElementById('cancelSendBtn');
            const confirmSendBtn = document.getElementById('confirmSendBtn');
            const stopInputBtn = document.getElementById('stopInputBtn');
            
            // --- Modale Mot de Passe ---
            const openSettingsBtn = document.getElementById('openSettingsBtn');
            const passwordModal = document.getElementById('passwordModal');
            const passwordModalBackdrop = document.getElementById('passwordModalBackdrop');
            const passwordInput = document.getElementById('passwordInput');
            const passwordError = document.getElementById('passwordError');
            const cancelPasswordBtn = document.getElementById('cancelPasswordBtn');
            const submitPasswordBtn = document.getElementById('submitPasswordBtn');

            // --- Modale Paramètres ---
            const settingsModal = document.getElementById('settingsModal');
            const settingsModalBackdrop = document.getElementById('settingsModalBackdrop');
            const closeSettingsModalBtn = document.getElementById('closeSettingsModalBtn');
            const machineNameInput = document.getElementById('machineNameInput');
            const machineNameDisplay = document.getElementById('machineNameDisplay');
            const flexoCountSlider = document.getElementById('flexoCountSlider');
            const flexoCountValue = document.getElementById('flexoCountValue');
            const bluetoothDeviceNameInput = document.getElementById('bluetoothDeviceNameInput');
            const pixelToMmRatioInput = document.getElementById('pixelToMmRatioInput');

            // --- Bluetooth ---
            const bluetoothSearchBtn = document.getElementById('bluetoothSearchBtn');
            const bluetoothStatus = document.getElementById('bluetoothStatus');
            const bluetoothBtnText = document.getElementById('bluetoothBtnText');
            let connectedDevice = null;
            let btDeviceName = '';

            // --- Autres éléments ---
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
                '#FF0000', '#00FF00', '#0000FF', '#FFFF00', '#FF00FF',
                '#00FFFF', '#FF4500', '#7FFF00', '#9400D3', '#00FA9A'
            ];

            colors.forEach((color, index) => {
                const swatch = document.createElement('div');
                swatch.className = 'color-swatch w-8 h-8 rounded-full cursor-pointer border-2 border-transparent';
                swatch.style.backgroundColor = color;
                swatch.dataset.color = color;
                if (index === 0) {
                    swatch.classList.add('selected');
                    document.documentElement.style.setProperty('--main-marker-color', color);
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
                resetSession();
                
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
            
            function addMarkerButtonListeners() {
                document.querySelectorAll('.marker-btn').forEach(button => {
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

                document.querySelectorAll('.radio-selector').forEach(radio => {
                    radio.addEventListener('change', (event) => {
                        document.querySelectorAll('.marker-btn').forEach(btn => {
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
                            
                            const existingMarker = document.querySelector(`.draggable-marker[data-marker-id="${targetButtonId}"]`);
                            if (!existingMarker) {
                                createMarker(targetButtonId);
                            }
                        }
                        updateCorrections();
                    });
                });
            }
            
            function generateMarkerButtons(count) {
                markerButtonsWrapper.innerHTML = '';
                let htmlContent = '';
                for(let i = 1; i <= count; i++) {
                    htmlContent += `
                        <div class="flex flex-col items-center space-y-2">
                            <button data-marker-id="F${i}" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">F${i}</button>
                            <div><input type="radio" name="marker-select" id="radio-F${i}" data-target-button="F${i}" class="radio-selector"><label for="radio-F${i}" class="radio-label"></label></div>
                        </div>`;
                }
                htmlContent += `
                    <div class="flex flex-col items-center space-y-2">
                        <button data-marker-id="D" class="marker-btn bg-gray-200 hover:bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg transition-colors">D</button>
                        <div><input type="radio" name="marker-select" id="radio-D" data-target-button="D" class="radio-selector"><label for="radio-D" class="radio-label"></label></div>
                    </div>`;
                markerButtonsWrapper.innerHTML = htmlContent;
                addMarkerButtonListeners();
            }
            
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

                const pixelToMmRatio = parseFloat(pixelToMmRatioInput.value) || 1;

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
                    
                    const correctionX_px = mainX - secondaryX;
                    const correctionY_px = mainY - secondaryY;

                    const correctionX_mm = correctionX_px / pixelToMmRatio;
                    const correctionY_mm = correctionY_px / pixelToMmRatio;

                    const resultLine = document.createElement('p');
                    
                    let displayName = secondaryId;
                    if (secondaryId === 'D') {
                        displayName = 'Die Cutter';
                    }

                    resultLine.textContent = `${displayName} : Registre (${correctionY_mm.toFixed(2)}mm) / Centrage (${correctionX_mm.toFixed(2)}mm)`;
                    correctionsText.appendChild(resultLine);
                });
                sendToScreenBtn.classList.remove('hidden');
                correctionsContainer.classList.remove('hidden');
            }

            optionsBtn.addEventListener('click', () => {
                optionsModal.classList.remove('hidden');
                optionsModalBackdrop.classList.remove('hidden');
            });

            function closeModal(modal, backdrop) {
                modal.classList.add('hidden');
                backdrop.classList.add('hidden');
            }
            closeOptionsModalBtn.addEventListener('click', () => closeModal(optionsModal, optionsModalBackdrop));
            optionsModalBackdrop.addEventListener('click', () => closeModal(optionsModal, optionsModalBackdrop));

            markerSizeSlider.addEventListener('input', (event) => {
                const value = event.target.value;
                sliderValueSpan.textContent = value;
                updateMarkerSizes(value);
            });
            
            function resetSession() {
                document.querySelectorAll('.draggable-marker').forEach(marker => marker.remove());
                document.querySelectorAll('.marker-btn').forEach(btn => {
                    btn.classList.remove('ring-2', 'active-marker-ring', 'bg-yellow-400', 'border-yellow-500', 'text-white');
                    btn.classList.add('bg-gray-200', 'hover:bg-gray-300', 'text-gray-800');
                });
                document.querySelectorAll('.radio-selector').forEach(radio => radio.checked = false);
                lastSelectedMarker = null;
                updateCorrections();
            }
            trashCan.addEventListener('click', resetSession);

            sendToScreenBtn.addEventListener('click', () => {
                stopInputBtn.disabled = true;
                confirmationModal.classList.remove('hidden');
                confirmationModalBackdrop.classList.remove('hidden');
            });

            cancelSendBtn.addEventListener('click', () => closeModal(confirmationModal, confirmationModalBackdrop));
            confirmationModalBackdrop.addEventListener('click', () => closeModal(confirmationModal, confirmationModalBackdrop));

            confirmSendBtn.addEventListener('click', () => {
                stopInputBtn.disabled = false;
                sendToScreenBtn.textContent = 'Envoyé !';
                sendToScreenBtn.classList.replace('bg-indigo-600', 'bg-green-500');
                sendToScreenBtn.classList.replace('hover:bg-indigo-700', 'hover:bg-green-600');
                setTimeout(() => {
                    sendToScreenBtn.textContent = "Envoyer à l'écran";
                     sendToScreenBtn.classList.replace('bg-green-500', 'bg-indigo-600');
                     sendToScreenBtn.classList.replace('hover:bg-green-600', 'hover:bg-indigo-700');
                }, 2000);
            });
            
            stopInputBtn.addEventListener('click', () => {
                if (stopInputBtn.disabled) return;
                closeModal(confirmationModal, confirmationModalBackdrop);
                console.log('Saisie arrêtée.');
            });

            // --- Logique Paramètres & Mot de Passe ---
            openSettingsBtn.addEventListener('click', () => {
                closeModal(optionsModal, optionsModalBackdrop);
                passwordError.textContent = '';
                passwordInput.value = '';
                passwordModal.classList.remove('hidden');
                passwordModalBackdrop.classList.remove('hidden');
            });

            cancelPasswordBtn.addEventListener('click', () => closeModal(passwordModal, passwordModalBackdrop));
            submitPasswordBtn.addEventListener('click', () => {
                if (passwordInput.value === 'ERadmin!') {
                    closeModal(passwordModal, passwordModalBackdrop);
                    settingsModal.classList.remove('hidden');
                    settingsModalBackdrop.classList.remove('hidden');
                } else {
                    passwordError.textContent = 'Mot de passe incorrect.';
                }
            });
            closeSettingsModalBtn.addEventListener('click', () => closeModal(settingsModal, settingsModalBackdrop));

            machineNameInput.addEventListener('input', (e) => {
                machineNameDisplay.textContent = e.target.value || 'nom de la machine';
            });
            
            flexoCountSlider.addEventListener('input', (e) => {
                const count = e.target.value;
                flexoCountValue.textContent = count;
                resetSession();
                generateMarkerButtons(count);
            });
            
            pixelToMmRatioInput.addEventListener('input', updateCorrections);
            
            bluetoothDeviceNameInput.addEventListener('input', (e) => {
                btDeviceName = e.target.value;
            });

            // --- Logique Bluetooth ---
            bluetoothSearchBtn.addEventListener('click', async () => {
                if (connectedDevice) {
                    connectedDevice.gatt.disconnect();
                    return;
                }
                if (!btDeviceName) {
                    alert("Veuillez d'abord renseigner le nom de l'appareil Bluetooth dans les paramètres.");
                    return;
                }
                 if (!navigator.bluetooth) {
                    alert("L'API Web Bluetooth n'est pas supportée sur ce navigateur.");
                    return;
                }

                try {
                    console.log(`Recherche de l'appareil: ${btDeviceName}`);
                    const device = await navigator.bluetooth.requestDevice({
                        filters: [{ name: btDeviceName }],
                        optionalServices: ['battery_service', 'device_information'] 
                    });

                    console.log('Appareil trouvé:', device.name);
                    connectedDevice = device;
                    connectedDevice.addEventListener('gattserverdisconnected', onDisconnected);
                    
                    const server = await connectedDevice.gatt.connect();
                    console.log('Connecté au serveur GATT:', server);

                    bluetoothStatus.classList.replace('bg-red-500', 'bg-green-500');
                    bluetoothBtnText.textContent = device.name;
                    alert(`Connecté à ${device.name}`);

                } catch(error) {
                    console.error('Erreur Bluetooth:', error);
                    alert(`Erreur: ${error.message}`);
                }
            });

            function onDisconnected() {
                console.log('Appareil déconnecté.');
                bluetoothStatus.classList.replace('bg-green-500', 'bg-red-500');
                bluetoothBtnText.textContent = 'Rechercher via Bluetooth';
                connectedDevice = null;
                alert('Appareil déconnecté.');
            }


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
                handleToolRepositioning(markerRect);
            }

            function checkCollision(rect1, rect2) {
                return !(rect1.right < rect2.left || rect1.left > rect2.right || rect1.bottom < rect2.top || rect1.top > rect2.bottom);
            }
            
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

            function handleToolRepositioning(markerRect) {
                 const containerRect = imageContainer.getBoundingClientRect();
                 const margin = 10;
                const glassDefaultRect = { top: containerRect.top + margin, left: containerRect.right - 150 - margin, width: 150, height: 150 };
                glassDefaultRect.right = glassDefaultRect.left + 150;
                glassDefaultRect.bottom = glassDefaultRect.top + 150;
                const dpadDefaultRect = { top: containerRect.bottom - 96 - margin, left: containerRect.right - 96 - margin, width: 96, height: 96 };
                dpadDefaultRect.right = dpadDefaultRect.left + 96;
                dpadDefaultRect.bottom = dpadDefaultRect.top + 96;
                const trashRect = trashCan.getBoundingClientRect();
                trashCan.classList.toggle('hovering', checkCollision(markerRect, trashRect));

                if (checkCollision(markerRect, glassDefaultRect)) {
                    magnifierGlass.style.right = 'auto';
                    magnifierGlass.style.left = '10px';
                } else {
                    magnifierGlass.style.left = 'auto';
                    magnifierGlass.style.right = '10px';
                }
                
                if (checkCollision(markerRect, dpadDefaultRect)) {
                    const obstacles = [
                        magnifierGlass.getBoundingClientRect(),
                        trashCan.getBoundingClientRect(),
                        ...Array.from(document.querySelectorAll('.draggable-marker:not([data-marker-id="' + activeMarker.dataset.markerId + '"])')).map(m => m.getBoundingClientRect())
                    ];
                    const safeCorner = findSafeCorner(dpadControls, obstacles);
                    dpadControls.style.top = safeCorner.top;
                    dpadControls.style.bottom = safeCorner.bottom;
                    dpadControls.style.left = safeCorner.left;
                    dpadControls.style.right = safeCorner.right;
                } else {
                    resetDpadPosition();
                }
            }
            
            function resetDpadPosition(){
                dpadControls.style.top = 'auto';
                dpadControls.style.bottom = '0.5rem';
                dpadControls.style.left = 'auto';
                dpadControls.style.right = '0.5rem';
            }

            function findSafeCorner(element, obstacles) {
                const elRect = element.getBoundingClientRect();
                const containerRect = imageContainer.getBoundingClientRect();
                const margin = 8;
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
                return corners[0];
            }

            // --- Initialisation ---
            generateMarkerButtons(6);
            machineNameInput.value = 'nom de la machine';
            pixelToMmRatioInput.value = '10';

            document.addEventListener('mouseup', dragEnd);
            document.addEventListener('touchend', dragEnd);
            document.addEventListener('mousemove', drag);
            document.addEventListener('touchmove', drag, { passive: false });
        });
    </script>

</body>
</html>


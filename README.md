<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimum-scale=1.0">
    <title>Chargeur d'images avec Marqueurs</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root {
            --main-marker-color: #FF0000; /* Couleur par défaut (Rouge Vif) */
        }
        body {
            font-family: sans-serif;
            touch-action: none; /* Empêche le pinch-to-zoom sur la page entière */
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
            width: 150px;
            height: 150px;
            border: 3px solid #000;
            border-radius: 50%;
            overflow: hidden;
            pointer-events: none; /* Important! */
            z-index: 10;
            background-repeat: no-repeat;
            background-color: white;
            /* La position est maintenant gérée par JS */
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
        
        #up-arrow {
            position: absolute;
            top: 0.5rem;
            left: 50%;
            transform: translateX(-50%);
            width: 3rem;
            height: 3rem;
            background-color: rgba(255, 255, 255, 0.8);
            border-radius: 0.5rem;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 6;
            pointer-events: none;
        }

        .dragging {
            opacity: 0.5;
            background: #e0e7ff;
        }

        .drag-over {
            border-top: 2px solid #4f46e5;
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div class="bg-white p-8 rounded-xl shadow-lg text-center max-w-2xl w-full">
        <!-- Titre de l'application -->
        <h1 class="text-2xl font-bold text-gray-800 mb-6">Easy Register (<span id="machineNameDisplay">nom de la machine</span>)</h1>

        <!-- Conteneur pour les boutons du haut -->
        <div class="flex justify-center items-center space-x-4 mb-6">
            <label for="imageUpload" class="inline-block bg-blue-500 text-white font-semibold py-3 px-6 rounded-lg cursor-pointer hover:bg-blue-600 transition-colors duration-300 ease-in-out shadow-md">
                Sélectionner une image
            </label>
            <input type="file" id="imageUpload" class="hidden" accept="image/*">
            <button id="optionsBtn" class="bg-gray-500 text-white font-semibold py-3 px-6 rounded-lg hover:bg-gray-600 transition-colors">Options</button>
        </div>

        <!-- La zone où l'image et les marqueurs seront affichés, cachée par défaut -->
        <div id="imageDisplayArea" class="mt-8 hidden">
            <div id="imageContainer">
                <img id="imagePreview" alt="Aperçu de l'image" class="max-w-full mx-auto rounded-lg shadow-md"/>
                <!-- La poubelle pour supprimer les marqueurs -->
                <div id="trashCan">🗑️</div>
                <!-- Flèche vers le haut -->
                <div id="up-arrow" class="hidden">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-gray-700" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                      <path stroke-linecap="round" stroke-linejoin="round" d="M5 10l7-7m0 0l7 7m-7-7v18" />
                    </svg>
                </div>
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
            <button id="sendToScreenBtn" class="mt-4 w-full bg-indigo-600 text-white font-semibold py-2 px-5 rounded-lg hover:bg-indigo-700 transition-colors">Envoyer à l'écran</button>
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
        <h2 class="text-xl font-bold text-gray-800 mb-4">Accès aux Paramètres</h2>
        <p class="text-gray-600 mb-4">Appuyez sur Entrée ou Valider pour continuer.</p>
        <input type="password" id="passwordInput" class="w-full border border-gray-300 rounded-lg px-3 py-2 mb-4">
        <div class="flex justify-end space-x-3">
            <button id="cancelPasswordBtn" class="bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-400">Annuler</button>
            <button id="submitPasswordBtn" class="bg-blue-500 text-white font-semibold py-2 px-4 rounded-lg hover:bg-blue-600">Valider</button>
        </div>
    </div>

    <!-- Fenêtre Modale pour les Paramètres -->
    <div id="settingsModalBackdrop" class="fixed inset-0 bg-black bg-opacity-50 hidden z-40"></div>
    <div id="settingsModal" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl hidden z-50 w-full max-w-md h-5/6 flex flex-col">
        <div class="flex justify-between items-center mb-4 flex-shrink-0">
            <h2 class="text-xl font-bold text-gray-800">Paramètres</h2>
            <button id="closeSettingsModalBtn" class="text-gray-500 hover:text-gray-800 text-3xl leading-none">&times;</button>
        </div>
        <div class="space-y-6 overflow-y-auto pr-4 flex-grow">
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
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Sauvegarde Automatique</label>
                <div class="flex items-center space-x-3">
                    <input type="checkbox" id="autoSaveCheckbox" class="h-5 w-5 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500">
                    <label for="autoSaveCheckbox" class="text-sm text-gray-600">Sauvegarder l'image après envoi</label>
                </div>
                <div class="mt-2">
                     <button id="selectFolderBtn" class="w-full bg-gray-200 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-300 transition-colors">Sélectionner un dossier</button>
                     <p id="folderNameDisplay" class="text-xs text-gray-500 mt-1"></p>
                </div>
            </div>
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Sauvegarde automatique avec croix</label>
                <div class="flex items-center space-x-3">
                    <input type="checkbox" id="autoSaveWithMarkersCheckbox" class="h-5 w-5 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500">
                    <label for="autoSaveWithMarkersCheckbox" class="text-sm text-gray-600">Sauvegarder l'image avec les croix et le rapport</label>
                </div>
                <div class="mt-2">
                     <button id="selectFolderWithMarkersBtn" class="w-full bg-gray-200 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-300 transition-colors">Sélectionner un dossier</button>
                     <p id="folderNameWithMarkersDisplay" class="text-xs text-gray-500 mt-1"></p>
                </div>
            </div>
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">Gestion de la Configuration</label>
                <div class="flex space-x-2">
                    <button id="importConfigBtn" class="w-full bg-blue-500 text-white font-semibold py-2 px-4 rounded-lg hover:bg-blue-600">Importer</button>
                    <button id="exportConfigBtn" class="w-full bg-green-500 text-white font-semibold py-2 px-4 rounded-lg hover:bg-green-600">Exporter</button>
                </div>
                <input type="file" id="configFileUpload" class="hidden" accept=".json, .txt">
            </div>
            <div>
                <button id="openSequenceEditorBtn" class="w-full bg-gray-200 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-300 transition-colors">Éditeur de Séquence</button>
            </div>
        </div>
    </div>
    
    <!-- Fenêtre Modale pour l'Éditeur de Séquence -->
    <div id="sequenceEditorModalBackdrop" class="fixed inset-0 bg-black bg-opacity-50 hidden z-40"></div>
    <div id="sequenceEditorModal" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl hidden z-50 w-full max-w-2xl h-5/6 flex flex-col">
        <div class="flex justify-between items-center mb-4 flex-shrink-0">
            <h2 class="text-xl font-bold text-gray-800">Éditeur de Séquence</h2>
            <div>
                <button id="addSequenceActionBtn" class="bg-blue-500 text-white rounded-full h-8 w-8 flex items-center justify-center hover:bg-blue-600 text-xl font-bold">+</button>
            </div>
        </div>
        <div id="sequence-fields-container" class="overflow-y-auto pr-2 space-y-2">
             <!-- Les champs de coordonnées seront générés ici -->
        </div>
        <div class="mt-4 pt-4 border-t flex-shrink-0 flex justify-end">
             <button id="saveSequenceBtn" class="bg-green-500 text-white font-semibold py-2 px-4 rounded-lg hover:bg-green-600 flex items-center space-x-2">
                 <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                   <path d="M7.707 10.293a1 1 0 10-1.414 1.414l3 3a1 1 0 001.414 0l3-3a1 1 0 00-1.414-1.414L11 11.586V6a1 1 0 10-2 0v5.586l-1.293-1.293zM5 4a2 2 0 012-2h6a2 2 0 012 2v2a1 1 0 102 0V4a4 4 0 00-4-4H7a4 4 0 00-4 4v12a4 4 0 004 4h6a4 4 0 004-4v-2a1 1 0 10-2 0v2a2 2 0 01-2 2H7a2 2 0 01-2-2V4z" />
                 </svg>
                 <span>Enregistrer</span>
               </button>
             <button id="closeSequenceEditorBtn" class="ml-4 bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-lg hover:bg-gray-400">Fermer</button>
        </div>
    </div>
    
    <!-- Fenêtre Modale pour l'affichage de la Séquence -->
    <div id="sequenceDisplayModalBackdrop" class="fixed inset-0 bg-black bg-opacity-50 hidden z-40"></div>
    <div id="sequenceDisplayModal" class="fixed top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-white p-6 rounded-lg shadow-xl hidden z-50 w-full max-w-md">
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold text-gray-800">Séquence Générée</h2>
            <button id="closeSequenceDisplayBtn" class="text-gray-500 hover:text-gray-800 text-3xl leading-none">&times;</button>
        </div>
        <pre id="sequence-output" class="bg-gray-100 p-4 rounded text-left text-sm whitespace-pre-wrap h-64 overflow-y-auto"></pre>
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
            // --- Récupération des éléments du DOM (partie 1) ---
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
            const upArrow = document.getElementById('up-arrow');
            
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
            const importConfigBtn = document.getElementById('importConfigBtn');
            const exportConfigBtn = document.getElementById('exportConfigBtn');
            const configFileUpload = document.getElementById('configFileUpload');
            const autoSaveCheckbox = document.getElementById('autoSaveCheckbox');
            const selectFolderBtn = document.getElementById('selectFolderBtn');
            const folderNameDisplay = document.getElementById('folderNameDisplay');
            const autoSaveWithMarkersCheckbox = document.getElementById('autoSaveWithMarkersCheckbox');
            const selectFolderWithMarkersBtn = document.getElementById('selectFolderWithMarkersBtn');
            const folderNameWithMarkersDisplay = document.getElementById('folderNameWithMarkersDisplay');
            
            // --- Modale Éditeur de Séquence ---
            const openSequenceEditorBtn = document.getElementById('openSequenceEditorBtn');
            const sequenceEditorModal = document.getElementById('sequenceEditorModal');
            const sequenceEditorModalBackdrop = document.getElementById('sequenceEditorModalBackdrop');
            const closeSequenceEditorBtn = document.getElementById('closeSequenceEditorBtn');
            const sequenceFieldsContainer = document.getElementById('sequence-fields-container');
            const addSequenceActionBtn = document.getElementById('addSequenceActionBtn');
            const saveSequenceBtn = document.getElementById('saveSequenceBtn');


            // --- Modale Affichage de Séquence ---
            const sequenceDisplayModal = document.getElementById('sequenceDisplayModal');
            const sequenceDisplayModalBackdrop = document.getElementById('sequenceDisplayModalBackdrop');
            const closeSequenceDisplayBtn = document.getElementById('closeSequenceDisplayBtn');
            const sequenceOutput = document.getElementById('sequence-output');

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
            let sequence = [];
            let autoSaveEnabled = false;
            let directoryHandle = null;
            let savedDirectoryName = '';
            let autoSaveWithMarkersEnabled = false;
            let directoryHandleWithMarkers = null;
            let savedDirectoryNameWithMarkers = '';
            
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
            
            function saveState() {
                const config = {
                    machineName: machineNameInput.value,
                    flexoCount: parseInt(flexoCountSlider.value, 10),
                    pixelToMmRatio: parseFloat(pixelToMmRatioInput.value),
                    bluetoothDeviceName: bluetoothDeviceNameInput.value,
                    sequence: sequence,
                    autoSaveEnabled: autoSaveCheckbox.checked,
                    savedDirectoryName: savedDirectoryName,
                    autoSaveWithMarkersEnabled: autoSaveWithMarkersCheckbox.checked,
                    savedDirectoryNameWithMarkers: savedDirectoryNameWithMarkers
                };
                localStorage.setItem('easyRegisterState', JSON.stringify(config));
            }
            
            function applyConfig(config) {
                if (config.machineName) {
                    machineNameInput.value = config.machineName;
                    machineNameDisplay.textContent = config.machineName;
                }
                if (config.flexoCount) {
                    const count = parseInt(config.flexoCount, 10);
                    flexoCountSlider.value = count;
                    flexoCountValue.textContent = count;
                    generateMarkerButtons(count);
                }
                if (config.pixelToMmRatio) {
                    pixelToMmRatioInput.value = config.pixelToMmRatio;
                }
                if (config.bluetoothDeviceName) {
                    bluetoothDeviceNameInput.value = config.bluetoothDeviceName;
                    btDeviceName = config.bluetoothDeviceName;
                }
                if (config.sequence) {
                    sequence = config.sequence;
                }
                if (config.autoSaveEnabled) {
                    autoSaveCheckbox.checked = !!config.autoSaveEnabled;
                    autoSaveEnabled = !!config.autoSaveEnabled;
                }
                if (config.savedDirectoryName) {
                    savedDirectoryName = config.savedDirectoryName;
                    folderNameDisplay.textContent = `Dossier : ${savedDirectoryName}`;
                }
                if (config.autoSaveWithMarkersEnabled) {
                    autoSaveWithMarkersCheckbox.checked = !!config.autoSaveWithMarkersEnabled;
                    autoSaveWithMarkersEnabled = !!config.autoSaveWithMarkersEnabled;
                }
                if (config.savedDirectoryNameWithMarkers) {
                    savedDirectoryNameWithMarkers = config.savedDirectoryNameWithMarkers;
                    folderNameWithMarkersDisplay.textContent = `Dossier : ${savedDirectoryNameWithMarkers}`;
                }

                updateCorrections();
            }

            function loadState() {
                const savedState = localStorage.getItem('easyRegisterState');
                if (savedState) {
                    try {
                        const config = JSON.parse(savedState);
                        applyConfig(config);
                    } catch (e) {
                        console.error("Erreur lors du chargement de la configuration sauvegardée:", e);
                        const initialFlexoCount = 6;
                        generateMarkerButtons(initialFlexoCount);
                        initializeDefaultSequence(initialFlexoCount);
                    }
                } else {
                    const initialFlexoCount = 6;
                    generateMarkerButtons(initialFlexoCount);
                    initializeDefaultSequence(initialFlexoCount);
                }
            }
            
            imageUploadInput.addEventListener('change', function(event) {
                if (event.target.files && event.target.files[0]) {
                    const file = event.target.files[0];
                    const imageURL = URL.createObjectURL(file);
                    setImageAsPreview(imageURL, file);
                }
            });

            function setImageAsPreview(imageURL, blob) {
                resetSession();
                imagePreviewElement.src = imageURL;
                 if (blob) {
                    imagePreviewElement.dataset.blob = URL.createObjectURL(blob); // Keep blob for saving
                 } else {
                    delete imagePreviewElement.dataset.blob;
                 }
                magnifierGlass.style.backgroundImage = `url('${imageURL}')`;
                imagePreviewElement.onload = () => {
                    imageDisplayArea.classList.remove('hidden'); 
                    controlsContainer.classList.remove('hidden');
                    upArrow.classList.remove('hidden');
                };
            }
            
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
                   correspondingButton.classList.add('ring-2', 'ring-offset-2', 'active-marker-ring');
                }
                updateCorrections();
            }
            
            function updateCorrections() {
                correctionsText.innerHTML = '';
                sendToScreenBtn.classList.add('hidden');

                const pixelToMmRatio = parseFloat(pixelToMmRatioInput.value) || 1;

                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (!selectedRadio) {
                    correctionsText.innerHTML = '<p class="text-gray-500">Sélectionnez une référence pour voir les corrections.</p>';
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
                    btn.classList.remove('ring-2', 'ring-offset-2', 'active-marker-ring', 'bg-yellow-400', 'border-yellow-500', 'text-white');
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
                
                generateAndShowSequence();
                handleAutoSave();
                handleAutoSaveWithMarkers();
                
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

            // --- Logique Paramètres & Mot de Passe & Séquence ---
            openSettingsBtn.addEventListener('click', () => {
                closeModal(optionsModal, optionsModalBackdrop);
                passwordInput.value = '';
                passwordModal.classList.remove('hidden');
                passwordModalBackdrop.classList.remove('hidden');
                passwordInput.focus();
            });
            
            openSequenceEditorBtn.addEventListener('click', () => {
                populateSequenceEditor();
                closeModal(settingsModal, settingsModalBackdrop);
                sequenceEditorModal.classList.remove('hidden');
                sequenceEditorModalBackdrop.classList.remove('hidden');
            });

            closeSequenceEditorBtn.addEventListener('click', () => closeModal(sequenceEditorModal, sequenceEditorModalBackdrop));
            closeSequenceDisplayBtn.addEventListener('click', () => closeModal(sequenceDisplayModal, sequenceDisplayModalBackdrop));

            async function checkPassword(inputPassword) {
                const storedHash = '36a9e7f1c95b82ffb99743e0c5c4ce95d83c9a430aac59f84ef3cbfab6145068'; // SHA-256 hash for " " (a single space)
                const encoder = new TextEncoder();
                const data = encoder.encode(inputPassword);
                const hashBuffer = await crypto.subtle.digest('SHA-256', data);
                const hashArray = Array.from(new Uint8Array(hashBuffer));
                const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
                return hashHex === storedHash;
            }

            async function handlePasswordSubmit() {
                if (await checkPassword(passwordInput.value)) {
                    openSettings();
                } else {
                    passwordInput.value = ''; // Clear incorrect password
                    alert("Mot de passe incorrect.");
                }
            }
            
            function openSettings() {
                closeModal(passwordModal, passwordModalBackdrop);
                settingsModal.classList.remove('hidden');
                settingsModalBackdrop.classList.remove('hidden');
            }

            cancelPasswordBtn.addEventListener('click', () => closeModal(passwordModal, passwordModalBackdrop));
            
            submitPasswordBtn.addEventListener('click', handlePasswordSubmit);

            passwordInput.addEventListener('keypress', (e) => {
                if (e.key === 'Enter') {
                    handlePasswordSubmit();
                }
            });

            closeSettingsModalBtn.addEventListener('click', () => closeModal(settingsModal, settingsModalBackdrop));

            machineNameInput.addEventListener('input', (e) => {
                machineNameDisplay.textContent = e.target.value || 'nom de la machine';
                saveState();
            });
            
            flexoCountSlider.addEventListener('input', (e) => {
                const count = parseInt(e.target.value, 10);
                flexoCountValue.textContent = count;
                resetSession();
                generateMarkerButtons(count);
                initializeDefaultSequence(count);
            });
            
            pixelToMmRatioInput.addEventListener('input', () => {
                updateCorrections();
                saveState();
            });
            
            bluetoothDeviceNameInput.addEventListener('input', (e) => {
                btDeviceName = e.target.value;
                saveState();
            });
            
             function createSequenceAction(id, type, flexoId, name, x = '', y = '') {
                 return { id, type, flexoId, name, x, y };
            }

            function initializeDefaultSequence(flexoCount) {
                sequence = []; // Reset the global sequence
                for (let i = 1; i <= flexoCount; i++) {
                    sequence.push(createSequenceAction(`f${i}_reg`, 'FLEXO_REG', `F${i}`, `F${i} Registre`));
                    sequence.push(createSequenceAction(`f${i}_cen`, 'FLEXO_CEN', `F${i}`, `F${i} Centrage`));
                }
                sequence.push(createSequenceAction('d_reg', 'DIECUT_REG', 'D', 'Découpe Registre'));
                sequence.push(createSequenceAction('d_cen', 'DIECUT_CEN', 'D', 'Découpe Centrage'));
                saveState();
            }

            
            function populateSequenceEditor() {
                sequenceFieldsContainer.innerHTML = ''; // Clear previous fields
                const currentSequence = sequence || [];
                
                currentSequence.forEach(action => {
                    const actionEl = createSequenceFieldHTML(action);
                    sequenceFieldsContainer.appendChild(actionEl);
                });
                 addDragAndDropListeners();
            }

            function createSequenceFieldHTML(action) {
                const { id, type, name, x, y } = action;
                const isIntermediate = type === 'INTERMEDIATE_CLICK';

                const element = document.createElement('div');
                element.className = 'p-2 border rounded-md bg-white flex items-center space-x-2';
                element.dataset.actionId = id;
                
                let content = `
                    <span class="cursor-move text-gray-400">⠿</span>
                    <input type="text" value="${name}" class="flex-grow font-semibold border-b p-1">
                `;

                if (isIntermediate) {
                    content += `
                        <div class="flex items-center space-x-1">
                            <input type="number" placeholder="X" value="${x}" class="w-20 border rounded p-1">
                            <input type="number" placeholder="Y" value="${y}" class="w-20 border rounded p-1">
                        </div>
                        <button class="text-red-500 hover:text-red-700 font-bold text-xl">&times;</button>
                    `;
                } else {
                     content += `<span class="text-sm text-gray-500 italic ml-auto">Saisie de valeur</span>`;
                }
                
                element.innerHTML = content;
                element.setAttribute('draggable', true);
                
                const nameInput = element.querySelector('input[type="text"]');
                if (isIntermediate) {
                    element.querySelector('button').addEventListener('click', () => {
                        sequence = sequence.filter(a => a.id !== id);
                        populateSequenceEditor();
                    });
                } else {
                    nameInput.disabled = true;
                    nameInput.classList.add('bg-gray-100');
                }
                return element;
            }
            
            function saveSequenceFromEditor() {
                const newSequence = [];
                sequenceFieldsContainer.querySelectorAll('[data-action-id]').forEach(el => {
                    const id = el.dataset.actionId;
                    const originalAction = sequence.find(a => a.id === id);
                    const name = el.querySelector('input[type="text"]').value;
                    const xInput = el.querySelectorAll('input[type="number"]')[0];
                    const yInput = el.querySelectorAll('input[type="number"]')[1];

                    newSequence.push({ 
                        ...originalAction, 
                        name, 
                        x: xInput ? xInput.value : '', 
                        y: yInput ? yInput.value : '' 
                    });
                });
                sequence = newSequence;
                saveState();
                
                // Visual feedback for save button
                const saveText = saveSequenceBtn.querySelector('span');
                const originalText = saveText.textContent;
                saveSequenceBtn.classList.replace('bg-green-500', 'bg-blue-500');
                saveSequenceBtn.classList.replace('hover:bg-green-600', 'hover:bg-blue-600');
                saveText.textContent = "Enregistré !";
                setTimeout(() => {
                    saveText.textContent = originalText;
                    saveSequenceBtn.classList.replace('bg-blue-500', 'bg-green-500');
                    saveSequenceBtn.classList.replace('hover:bg-blue-600', 'hover:bg-green-600');
                }, 1500);
            }
            
            addSequenceActionBtn.addEventListener('click', () => {
                const newAction = createSequenceAction(`custom_${Date.now()}`, 'INTERMEDIATE_CLICK', null, 'Nouvelle Action');
                if (!sequence) {
                    sequence = [];
                }
                sequence.push(newAction);
                populateSequenceEditor();
            });

            saveSequenceBtn.addEventListener('click', saveSequenceFromEditor);

            function addDragAndDropListeners() {
                let draggingElement = null;
                sequenceFieldsContainer.querySelectorAll('[draggable="true"]').forEach(item => {
                    item.addEventListener('dragstart', (e) => {
                        draggingElement = e.target.closest('[data-action-id]');
                        setTimeout(() => {
                           if(draggingElement) draggingElement.classList.add('dragging');
                        }, 0);
                    });

                    item.addEventListener('dragend', () => {
                        if (draggingElement) {
                             draggingElement.classList.remove('dragging');
                        }
                        draggingElement = null;
                        document.querySelectorAll('.drag-over').forEach(el => el.classList.remove('drag-over'));
                    });
                });

                sequenceFieldsContainer.addEventListener('dragover', (e) => {
                    e.preventDefault();
                    const afterElement = getDragAfterElement(sequenceFieldsContainer, e.clientY);
                    document.querySelectorAll('.drag-over').forEach(el => el.classList.remove('drag-over'));
                    if(afterElement) {
                        afterElement.classList.add('drag-over');
                    }
                });

                 sequenceFieldsContainer.addEventListener('drop', (e) => {
                    e.preventDefault();
                    if (!draggingElement) return;
                    const afterElement = getDragAfterElement(sequenceFieldsContainer, e.clientY);
                    if (afterElement == null) {
                        sequenceFieldsContainer.appendChild(draggingElement);
                    } else {
                        sequenceFieldsContainer.insertBefore(draggingElement, afterElement);
                    }
                });
            }

            function getDragAfterElement(container, y) {
                const draggableElements = [...container.querySelectorAll('[draggable="true"]:not(.dragging)')];
                return draggableElements.reduce((closest, child) => {
                    const box = child.getBoundingClientRect();
                    const offset = y - box.top - box.height / 2;
                    if (offset < 0 && offset > closest.offset) {
                        return { offset: offset, element: child };
                    } else {
                        return closest;
                    }
                }, { offset: Number.NEGATIVE_INFINITY }).element;
            }


            function generateAndShowSequence() {
                let generatedSequence = [];
                const savedSequence = sequence || [];
                const pixelToMmRatio = parseFloat(pixelToMmRatioInput.value) || 1;
                
                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (!selectedRadio) {
                    alert("Veuillez sélectionner une référence (couronne).");
                    return;
                }
                const mainMarker = document.querySelector(`.draggable-marker[data-marker-id="${selectedRadio.dataset.targetButton}"]`);
                if (!mainMarker) {
                    alert("Marqueur de référence non trouvé sur l'image.");
                    return;
                }

                const mainX = parseFloat(mainMarker.style.left);
                const mainY = parseFloat(mainMarker.style.top);
                const activeMarkers = new Map();
                document.querySelectorAll('.draggable-marker').forEach(m => activeMarkers.set(m.dataset.markerId, m));
                
                savedSequence.forEach(action => {
                    if (action.type === 'INTERMEDIATE_CLICK') {
                        if(action.x && action.y) generatedSequence.push(`CLIC,${action.x},${action.y}`);
                    } else {
                        const marker = activeMarkers.get(action.flexoId);
                        if(marker && marker !== mainMarker) {
                            const secondaryX = parseFloat(marker.style.left);
                            const secondaryY = parseFloat(marker.style.top);
                            const correctionY_mm = ((mainY - secondaryY) / pixelToMmRatio).toFixed(2);
                            const correctionX_mm = ((mainX - secondaryX) / pixelToMmRatio).toFixed(2);
                            
                            if (action.type.includes('REG')) {
                                generatedSequence.push(`ECRIRE,${correctionY_mm}`);
                            }
                            if (action.type.includes('CEN')) {
                                generatedSequence.push(`ECRIRE,${correctionX_mm}`);
                            }
                        }
                    }
                });


                sequenceOutput.textContent = generatedSequence.join('\n');
                sequenceDisplayModal.classList.remove('hidden');
                sequenceDisplayModalBackdrop.classList.remove('hidden');
            }


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
                    if (error.name === 'SecurityError') {
                        alert("Erreur de permission Bluetooth : Cet environnement ne permet pas l'accès au Bluetooth. Veuillez essayer d'exécuter ce fichier sur un serveur local ou dans un autre navigateur pour utiliser cette fonctionnalité.");
                    } else {
                        alert(`Erreur Bluetooth: ${error.message}`);
                    }
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
                positionMagnifierNextTo(lastSelectedMarker);

                magnifierHideTimer = setTimeout(() => {
                    magnifierGlass.classList.add('hidden');
                }, 3000);
                updateCorrections();
            }

            function positionMagnifierNextTo(element) {
                const markerRect = element.getBoundingClientRect();
                const containerRect = imageContainer.getBoundingClientRect();
                const offsetX = 20;
                const offsetY = 20;

                let magnifierX = (markerRect.left - containerRect.left + markerRect.width / 2) + offsetX;
                let magnifierY = (markerRect.top - containerRect.top + markerRect.height / 2) + offsetY;

                const glassWidth = magnifierGlass.offsetWidth;
                const glassHeight = magnifierGlass.offsetHeight;
                if (magnifierX + glassWidth > containerRect.width) {
                    magnifierX = (markerRect.left - containerRect.left + markerRect.width / 2) - glassWidth - offsetX;
                }
                if (magnifierY + glassHeight > containerRect.height) {
                    magnifierY = (markerRect.top - containerRect.top + markerRect.height / 2) - glassHeight - offsetY;
                }
                
                magnifierGlass.style.left = `${magnifierX}px`;
                magnifierGlass.style.top = `${magnifierY}px`;
                magnifierGlass.style.right = 'auto';
                magnifierGlass.style.bottom = 'auto';
            }


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
                        if (button) button.classList.remove('ring-2', 'ring-offset-2');
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
                
                const offsetX = 20;
                const offsetY = 20;
                let magnifierX = (clientX - containerRect.left) + offsetX;
                let magnifierY = (clientY - containerRect.top) + offsetY;

                const glassWidth = magnifierGlass.offsetWidth;
                const glassHeight = magnifierGlass.offsetHeight;
                if (magnifierX + glassWidth > containerRect.width) {
                    magnifierX = (clientX - containerRect.left) - glassWidth - offsetX;
                }
                if (magnifierY + glassHeight > containerRect.height) {
                    magnifierY = (clientY - containerRect.top) - glassHeight - offsetY;
                }

                magnifierGlass.style.left = `${magnifierX}px`;
                magnifierGlass.style.top = `${magnifierY}px`;
                magnifierGlass.style.right = 'auto';
                magnifierGlass.style.bottom = 'auto';

                const markerRect = activeMarker.getBoundingClientRect();
                handleToolRepositioning(markerRect);
            }

            function checkCollision(rect1, rect2) {
                return !(rect1.right < rect2.left || rect1.left > rect2.right || rect1.bottom < rect2.top || rect1.top > rect2.bottom);
            }
            
            function resetToolPositions() {
                dpadControls.style.top = 'auto';
                dpadControls.style.bottom = '0.5rem';
                dpadControls.style.right = '0.5rem';
                dpadControls.style.left = 'auto';
            }

            function handleToolRepositioning(markerRect) {
                const containerRect = imageContainer.getBoundingClientRect();
                const margin = 10;
                const dpadDefaultRect = { top: containerRect.bottom - 96 - margin, left: containerRect.right - 96 - margin, width: 96, height: 96 };
                dpadDefaultRect.right = dpadDefaultRect.left + 96;
                dpadDefaultRect.bottom = dpadDefaultRect.top + 96;
                const trashRect = trashCan.getBoundingClientRect();
                trashCan.classList.toggle('hovering', checkCollision(markerRect, trashRect));
                
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
            
            // --- Logique pour l'import/export de configuration ---
            exportConfigBtn.addEventListener('click', () => {
                const config = {
                    machineName: machineNameInput.value,
                    flexoCount: parseInt(flexoCountSlider.value, 10),
                    pixelToMmRatio: parseFloat(pixelToMmRatioInput.value),
                    bluetoothDeviceName: bluetoothDeviceNameInput.value,
                    sequence: sequence,
                    autoSaveEnabled: autoSaveCheckbox.checked,
                    savedDirectoryName: savedDirectoryName,
                    autoSaveWithMarkersEnabled: autoSaveWithMarkersCheckbox.checked,
                    savedDirectoryNameWithMarkers: savedDirectoryNameWithMarkers
                };
                const configJson = JSON.stringify(config, null, 2);
                const blob = new Blob([configJson], { type: 'application/json' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `easy-register-config-${config.machineName.replace(/\s+/g, '_') || 'default'}.json`;
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
                URL.revokeObjectURL(url);
            });

            importConfigBtn.addEventListener('click', () => {
                configFileUpload.click();
            });

            configFileUpload.addEventListener('change', (event) => {
                const file = event.target.files[0];
                if (!file) return;

                const reader = new FileReader();
                reader.onload = (e) => {
                    try {
                        const config = JSON.parse(e.target.result);
                        applyConfig(config);
                        saveState();
                        alert('Configuration importée avec succès !');
                        closeModal(settingsModal, settingsModalBackdrop);
                    } catch (error) {
                        console.error("Erreur d'importation:", error);
                        alert("Le fichier de configuration est invalide ou corrompu.");
                    }
                };
                reader.onerror = () => alert("Erreur lors de la lecture du fichier.");
                reader.readAsText(file);
                event.target.value = ''; // Permet de ré-importer le même fichier
            });

            // --- Logique pour le D-Pad ---
            let dpadInterval = null;
            const startDpadMove = (dx, dy) => {
                if (dpadInterval) clearInterval(dpadInterval);
                moveLastMarker(dx, dy);
                dpadInterval = setInterval(() => moveLastMarker(dx, dy), 100);
            };
            const stopDpadMove = () => {
                clearInterval(dpadInterval);
                dpadInterval = null;
            };

            ['mousedown', 'touchstart'].forEach(evt => {
                dpadUp.addEventListener(evt, (e) => { e.preventDefault(); startDpadMove(0, -1); });
                dpadDown.addEventListener(evt, (e) => { e.preventDefault(); startDpadMove(0, 1); });
                dpadLeft.addEventListener(evt, (e) => { e.preventDefault(); startDpadMove(-1, 0); });
                dpadRight.addEventListener(evt, (e) => { e.preventDefault(); startDpadMove(1, 0); });
            });
            ['mouseup', 'mouseleave', 'touchend', 'touchcancel'].forEach(evt => {
                document.body.addEventListener(evt, stopDpadMove);
            });

            // --- Logique pour le collage depuis le presse-papiers ---
             document.addEventListener('paste', (event) => {
                const items = (event.clipboardData || event.originalEvent.clipboardData).items;
                for (let i = 0; i < items.length; i++) {
                    if (items[i].type.indexOf('image') !== -1) {
                        const blob = items[i].getAsFile();
                        if (blob) {
                            const imageUrl = URL.createObjectURL(blob);
                            setImageAsPreview(imageUrl, blob);
                            event.preventDefault();
                            return;
                        }
                    }
                }
            });

            // --- Logique de sauvegarde automatique ---
            autoSaveCheckbox.addEventListener('change', (e) => {
                autoSaveEnabled = e.target.checked;
                saveState();
            });

            selectFolderBtn.addEventListener('click', async () => {
                try {
                    if (!window.showDirectoryPicker) {
                        alert("Votre navigateur ne supporte pas la sélection de dossier. Cette fonctionnalité pourrait ne pas être disponible.");
                        return;
                    }
                    directoryHandle = await window.showDirectoryPicker();
                    savedDirectoryName = directoryHandle.name;
                    folderNameDisplay.textContent = `Dossier : ${directoryHandle.name}`;
                    saveState();
                } catch (err) {
                    console.warn("Sélection de dossier annulée ou échouée:", err.name, err.message);
                }
            });
            
            autoSaveWithMarkersCheckbox.addEventListener('change', (e) => {
                autoSaveWithMarkersEnabled = e.target.checked;
                saveState();
            });

            selectFolderWithMarkersBtn.addEventListener('click', async () => {
                try {
                    if (!window.showDirectoryPicker) {
                        alert("Votre navigateur ne supporte pas la sélection de dossier.");
                        return;
                    }
                    directoryHandleWithMarkers = await window.showDirectoryPicker();
                    savedDirectoryNameWithMarkers = directoryHandleWithMarkers.name;
                    folderNameWithMarkersDisplay.textContent = `Dossier : ${directoryHandleWithMarkers.name}`;
                    saveState();
                } catch (err) {
                    console.warn("Sélection de dossier (avec croix) annulée ou échouée:", err.name, err.message);
                }
            });

            function getFormattedDateTime() {
                const now = new Date();
                const day = String(now.getDate()).padStart(2, '0');
                const month = String(now.getMonth() + 1).padStart(2, '0'); // Les mois sont basés sur 0
                const year = now.getFullYear();
                const hours = String(now.getHours()).padStart(2, '0');
                const minutes = String(now.getMinutes()).padStart(2, '0');
                const seconds = String(now.getSeconds()).padStart(2, '0');
                return `${day}-${month}-${year}_${hours}-${minutes}-${seconds}`;
            }

            async function handleAutoSave() {
                if (!autoSaveEnabled || !directoryHandle || !imagePreviewElement.dataset.blob) {
                    if(autoSaveEnabled && !directoryHandle) console.warn("Sauvegarde auto activée mais aucun dossier sélectionné.");
                    return;
                }

                try {
                    if (await directoryHandle.queryPermission({ mode: 'readwrite' }) !== 'granted') {
                        if (await directoryHandle.requestPermission({ mode: 'readwrite' }) !== 'granted') {
                            alert("La permission d'écrire dans le dossier a été refusée.");
                            directoryHandle = null;
                            savedDirectoryName = '';
                            folderNameDisplay.textContent = '';
                            saveState();
                            return;
                        }
                    }

                    const blobUrl = imagePreviewElement.dataset.blob;
                    const response = await fetch(blobUrl);
                    const imageBlob = await response.blob();
                    
                    const fileName = `${getFormattedDateTime()}.png`;
                    const fileHandle = await directoryHandle.getFileHandle(fileName, { create: true });
                    const writable = await fileHandle.createWritable();
                    await writable.write(imageBlob);
                    await writable.close();
                    console.log(`Image sauvegardée avec succès : ${fileName}`);

                } catch(error) {
                    console.error("Erreur lors de la sauvegarde automatique de l'image:", error);
                    alert("Une erreur est survenue lors de la sauvegarde de l'image.");
                }
            }
            
            async function handleAutoSaveWithMarkers() {
                if (!autoSaveWithMarkersEnabled || !directoryHandleWithMarkers || !imagePreviewElement.src || imagePreviewElement.src.endsWith('#')) {
                    if(autoSaveWithMarkersEnabled && !directoryHandleWithMarkers) console.warn("Sauvegarde auto (avec croix) activée mais aucun dossier sélectionné.");
                    return;
                }

                try {
                    if (await directoryHandleWithMarkers.queryPermission({ mode: 'readwrite' }) !== 'granted') {
                        if (await directoryHandleWithMarkers.requestPermission({ mode: 'readwrite' }) !== 'granted') {
                            alert("La permission d'écrire dans le dossier a été refusée pour la sauvegarde avec croix.");
                            directoryHandleWithMarkers = null;
                            savedDirectoryNameWithMarkers = '';
                            folderNameWithMarkersDisplay.textContent = '';
                            saveState();
                            return;
                        }
                    }

                    const canvas = document.createElement('canvas');
                    const ctx = canvas.getContext('2d');
                    canvas.width = imagePreviewElement.naturalWidth;
                    canvas.height = imagePreviewElement.naturalHeight;
                    ctx.drawImage(imagePreviewElement, 0, 0, canvas.width, canvas.height);

                    const markers = document.querySelectorAll('.draggable-marker');
                    const displayedImgRect = imagePreviewElement.getBoundingClientRect();
                    const scaleX = canvas.width / displayedImgRect.width;
                    const scaleY = canvas.height / displayedImgRect.height;
                    const mainColor = getComputedStyle(document.documentElement).getPropertyValue('--main-marker-color').trim();

                    markers.forEach(marker => {
                        const markerRect = marker.getBoundingClientRect();
                        const markerCenterXInDisplayedImg = (markerRect.left + markerRect.width / 2) - displayedImgRect.left;
                        const markerCenterYInDisplayedImg = (markerRect.top + markerRect.height / 2) - displayedImgRect.top;
                        const canvasX = markerCenterXInDisplayedImg * scaleX;
                        const canvasY = markerCenterYInDisplayedImg * scaleY;
                        const id = marker.dataset.markerId;
                        const labelEl = marker.querySelector('.marker-label');
                        const labelStyle = window.getComputedStyle(labelEl);
                        const currentMarkerScale = parseFloat(marker.style.transform.split('scale(')[1]) || 1;
                        const baseMarkerWidth = 24;
                        const scaledMarkerWidth = baseMarkerWidth * currentMarkerScale;
                        const markerWidthOnCanvas = scaledMarkerWidth * scaleX;
                        const markerHeightOnCanvas = scaledMarkerWidth * scaleY;
                        const crosshairThicknessOnCanvas = 2 * Math.min(scaleX, scaleY);
                        
                        ctx.fillStyle = mainColor;
                        ctx.fillRect(canvasX - markerWidthOnCanvas / 2, canvasY - crosshairThicknessOnCanvas / 2, markerWidthOnCanvas, crosshairThicknessOnCanvas);
                        ctx.fillRect(canvasX - crosshairThicknessOnCanvas / 2, canvasY - markerHeightOnCanvas / 2, crosshairThicknessOnCanvas, markerHeightOnCanvas);
                        
                        ctx.font = `${labelStyle.fontWeight} ${parseInt(labelStyle.fontSize) * scaleY}px sans-serif`;
                        ctx.fillStyle = mainColor;
                        ctx.textAlign = 'left';
                        ctx.textBaseline = 'middle';
                        ctx.strokeStyle = 'white';
                        ctx.lineWidth = 2 * Math.min(scaleX, scaleY);
                        ctx.strokeText(id, canvasX + markerWidthOnCanvas / 2 + (5 * scaleX), canvasY);
                        ctx.fillText(id, canvasX + markerWidthOnCanvas / 2 + (5 * scaleX), canvasY);
                    });
                    
                    const baseFileName = `${getFormattedDateTime()}_X`;
                    const imageFileName = `${baseFileName}.png`;
                    const textFileName = `${baseFileName}.txt`;

                    canvas.toBlob(async (blob) => {
                        if (blob) {
                            const imageFileHandle = await directoryHandleWithMarkers.getFileHandle(imageFileName, { create: true });
                            const writableImage = await imageFileHandle.createWritable();
                            await writableImage.write(blob);
                            await writableImage.close();
                            console.log(`Image avec croix sauvegardée : ${imageFileName}`);
                            
                            const reportText = correctionsText.innerText;
                            const textFileHandle = await directoryHandleWithMarkers.getFileHandle(textFileName, { create: true });
                            const writableText = await textFileHandle.createWritable();
                            await writableText.write(reportText);
                            await writableText.close();
                            console.log(`Rapport sauvegardé : ${textFileName}`);
                        }
                    }, 'image/png');

                } catch (error) {
                    console.error("Erreur lors de la sauvegarde avec croix:", error);
                    alert("Une erreur est survenue lors de la sauvegarde de l'image et du rapport.");
                }
            }


            // --- Initialisation ---
            loadState();
            
            document.addEventListener('mouseup', dragEnd);
            document.addEventListener('touchend', dragEnd);
            document.addEventListener('mousemove', drag);
            document.addEventListener('touchmove', drag, { passive: false });
        });
    </script>

</body>
</html>






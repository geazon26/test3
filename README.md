<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimum-scale=1.0">
    <title>Chargeur d'images avec Marqueurs</title>
    <style>
        /* --- Variables et Styles Généraux --- */
        :root {
            --main-marker-color: #FF0000; /* Couleur par défaut (Rouge Vif) */
            --color-primary: #3B82F6;
            --color-primary-hover: #2563EB;
            --color-secondary: #6B7280;
            --color-secondary-hover: #4B5563;
            --color-success: #10B981;
            --color-success-hover: #059669;
            --color-danger: #EF4444;
            --color-danger-hover: #DC2626;
            --color-info: #4F46E5;
            --color-info-hover: #4338CA;
            --color-text-dark: #1F2937;
            --color-text-medium: #374151;
            --color-text-light: #6B7280;
            --color-bg-light: #F3F4F6;
            --color-bg-white: #FFFFFF;
            --color-border: #D1D5DB;
        }

        *, *::before, *::after {
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
            background-color: var(--color-bg-light);
            color: var(--color-text-dark);
            margin: 0;
            padding: 1rem;
            min-height: 100vh;
        }
        
        /* --- Conteneurs Principaux --- */
        .main-container {
            background-color: var(--color-bg-white);
            padding: 2rem;
            border-radius: 0.75rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            width: 100%;
            max-width: 42rem;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            text-align: center;
        }
        
        /* Style pour l'affichage initial (sans image) */
        .main-container.no-image {
             min-height: calc(100vh - 4rem); /* Prend de la hauteur pour que la marge auto fonctionne */
             padding-top: 5rem;
        }

        .header-controls {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 1rem;
            margin-bottom: 1.5rem;
        }
        
        .footer-controls {
            position: fixed;
            bottom: 1.5rem;
            left: 50%;
            transform: translateX(-50%);
            z-index: 20;
        }

        /* --- Typographie --- */
        h1 {
            font-size: 1.5rem;
            font-weight: bold;
            color: var(--color-text-dark);
            margin: 0 0 1.5rem 0;
        }
        h2 {
            font-size: 1.25rem;
            font-weight: bold;
            color: var(--color-text-dark);
            margin: 0;
        }
        p {
            color: var(--color-text-light);
            font-size: 0.875rem;
        }
        label, .label-text {
            display: block;
            text-align: left;
            font-size: 0.875rem;
            font-weight: 500;
            color: var(--color-text-medium);
            margin-bottom: 0.5rem;
        }

        /* --- Boutons --- */
        .btn {
            font-weight: 600;
            font-size: 0.875rem;
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            cursor: pointer;
            transition: background-color 0.2s ease-in-out;
            border: none;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            display: inline-flex;
            align-items: center;
            justify-content: center;
            color: var(--color-bg-white);
        }
        .btn-primary { background-color: var(--color-primary); }
        .btn-primary:hover { background-color: var(--color-primary-hover); }
        .btn-secondary { background-color: var(--color-secondary); }
        .btn-secondary:hover { background-color: var(--color-secondary-hover); }
        .btn-success { background-color: var(--color-success); }
        .btn-success:hover { background-color: var(--color-success-hover); }
        .btn-danger { background-color: var(--color-danger); }
        .btn-danger:hover { background-color: var(--color-danger-hover); }
        .btn-info { background-color: var(--color-info); }
        .btn-info:hover { background-color: var(--color-info-hover); }
        .btn-light {
            background-color: #E5E7EB;
            color: var(--color-text-dark);
        }
        .btn-light:hover { background-color: #D1D5DB; }

        .btn.full-width {
            width: 100%;
        }
        .btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        /* --- Modales --- */
        .modal-backdrop {
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background-color: rgba(0, 0, 0, 0.5);
            z-index: 40;
        }
        .modal {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: var(--color-bg-white);
            padding: 1.5rem;
            border-radius: 0.5rem;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            z-index: 50;
            width: 90%;
            max-width: 28rem;
        }
        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 1rem;
        }
        .modal-close-btn {
            background: none;
            border: none;
            font-size: 2rem;
            line-height: 1;
            color: var(--color-text-light);
            cursor: pointer;
            padding: 0;
        }
        .modal-close-btn:hover { color: var(--color-text-dark); }
        .modal-body {
            overflow-y: auto;
            padding-right: 1rem;
            flex-grow: 1;
        }
        .modal-body > * + * {
            margin-top: 1.5rem;
        }

        /* --- Groupes d'éléments pour l'alignement --- */
        .button-group {
            display: flex;
            gap: 0.5rem;
        }
        .button-group.centered {
            justify-content: center;
        }
        .modal-actions {
            display: flex;
            justify-content: flex-end;
            gap: 0.75rem;
        }
         .checkbox-group {
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }
        .checkbox-group label {
            margin-bottom: 0;
        }
        .confirmation-actions {
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
        }

        /* --- Formulaires --- */
        input[type="text"], input[type="password"], input[type="number"] {
            width: 100%;
            border: 1px solid var(--color-border);
            border-radius: 0.5rem;
            padding: 0.5rem 0.75rem;
            font-size: 1rem;
        }
        input[type="range"] {
            -webkit-appearance: none;
            width: 100%;
            height: 0.5rem;
            background: #E5E7EB;
            border-radius: 0.5rem;
            cursor: pointer;
        }
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: 1rem;
            width: 1rem;
            border-radius: 50%;
            background: var(--color-primary);
        }
        input[type="checkbox"] {
            height: 1.25rem;
            width: 1.25rem;
            border-radius: 0.25rem;
            border-color: var(--color-border);
            color: var(--color-info);
        }

        /* --- Composants Spécifiques --- */
        .hidden { display: none !important; }
        
        #imageDisplayArea {
            margin-top: 2rem;
        }
        #imagePreview {
            max-width: 100%;
            margin-left: auto;
            margin-right: auto;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }

        #controlsContainer {
            margin-top: 1.5rem;
        }
        #controlsContainer p {
            margin-bottom: 0.75rem;
        }
        #marker-buttons-wrapper {
            display: flex;
            justify-content: center;
            align-items: flex-start;
            flex-wrap: wrap;
            gap: 0.75rem;
        }
        .marker-btn-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 0.5rem;
        }

        .marker-btn {
            font-weight: 600;
            padding: 0.5rem 1rem;
            border-radius: 0.5rem;
            background-color: #E5E7EB;
            color: var(--color-text-dark);
            border: none;
            cursor: pointer;
            transition: background-color 0.2s;
        }
        .marker-btn:hover {
            background-color: #D1D5DB;
        }
        .marker-btn.active-marker-highlight {
            box-shadow: 0 0 0 2px white, 0 0 0 4px var(--main-marker-color);
        }
        .marker-btn.reference {
             background-color: #FBBF24;
             border-color: #F59E0B;
             color: white;
        }
        .marker-btn.reference:hover {
             background-color: #F59E0B;
        }


        #correctionsContainer {
            margin-top: 1.5rem;
            text-align: left;
            background-color: #F9FAFB;
            padding: 1rem;
            border-radius: 0.5rem;
            border: 1px solid #E5E7EB;
        }
        #correctionsText {
            font-family: "Courier New", Courier, monospace;
            font-size: 0.875rem;
        }
        #sendToScreenBtn {
            margin-top: 1rem;
            width: 100%;
        }
        
        #settingsModal {
            max-width: 32rem;
            height: 83.333%;
            display: flex;
            flex-direction: column;
        }
        #settingsModal.hidden { display: none; }

        /* --- Palette de couleurs (Options) --- */
        #color-palette {
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 0.75rem;
            justify-items: center;
        }
        .color-swatch {
            width: 2rem;
            height: 2rem;
            border-radius: 9999px;
            cursor: pointer;
            border: 2px solid transparent;
            transition: transform 0.1s ease-in-out, border-color 0.1s ease-in-out;
        }
        .color-swatch.selected {
            border-color: #3b82f6;
            transform: scale(1.15);
        }

        /* --- Éditeur de Séquence --- */
        #sequenceEditorModal {
            max-width: 42rem;
            height: 83.333%;
            display: flex;
            flex-direction: column;
        }
        #sequenceEditorModal.hidden { display: none; }
        #sequence-fields-container {
            overflow-y: auto;
            padding-right: 0.5rem;
        }
        #sequence-fields-container > * + * { margin-top: 0.5rem; }
        .sequence-action-item {
            padding: 0.5rem;
            border: 1px solid var(--color-border);
            border-radius: 0.375rem;
            background-color: var(--color-bg-white);
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        .sequence-action-item.dragging {
            opacity: 0.5;
            background: #e0e7ff;
        }
        .sequence-action-item.drag-over {
            border-top: 2px solid #4f46e5;
        }
        .sequence-action-item input[type="text"] {
            flex-grow: 1;
            font-weight: 600;
            border: none;
            border-bottom: 1px solid transparent;
            padding: 0.25rem;
        }
         .sequence-action-item input[type="text"]:not(:disabled) {
             border-bottom: 1px solid var(--color-border);
         }
        .sequence-action-item input[type="text"]:disabled {
            background-color: #F9FAFB;
        }
        .sequence-action-item input[type="number"] {
            width: 5rem;
        }
        .sequence-action-item .drag-handle {
            cursor: move;
            color: var(--color-text-light);
        }
        .sequence-action-item .delete-btn {
            background: none; border: none; color: var(--color-danger); font-weight: bold; font-size: 1.25rem; cursor: pointer;
        }
        .modal-footer {
            margin-top: 1rem;
            padding-top: 1rem;
            border-top: 1px solid var(--color-border);
            flex-shrink: 0;
            display: flex;
            justify-content: flex-end;
            gap: 0.75rem;
        }

        /* --- Styles existants pour les marqueurs (conservés et adaptés) --- */
        #imageContainer {
            position: relative;
            display: inline-block;
            max-width: 100%;
        }
        .draggable-marker {
            position: absolute;
            cursor: move;
            user-select: none;
            -webkit-user-select: none;
            transform: translate(-50%, -50%); 
            transform-origin: center center;
            transition: transform 0.1s ease-out, color 0.2s, background-color 0.2s;
            width: 24px;
            height: 24px;
            z-index: 5;
        }
        .draggable-marker.locked { cursor: default; }
        .marker-cross-h, .marker-cross-v {
            position: absolute;
            background-color: var(--main-marker-color);
            box-shadow: 0 0 2px white;
        }
        .marker-cross-h { width: 100%; height: 2px; top: 50%; left: 0; transform: translateY(-50%); }
        .marker-cross-v { width: 2px; height: 100%; left: 50%; top: 0; transform: translateX(-50%); }
        .marker-label {
            position: absolute;
            left: 100%;
            top: 50%;
            transform: translateY(-50%);
            font-size: 14px;
            padding-left: 5px;
            font-weight: 500;
            white-space: nowrap;
            color: var(--main-marker-color);
            text-shadow: 0 0 3px white, 0 0 3px white;
        }
        .draggable-marker.locked .marker-label { display: none; }
        .radio-selector { display: none; }
        .radio-label {
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 1.5rem;
            height: 1.5rem;
            border: 2px solid #9CA3AF;
            border-radius: 9999px;
            cursor: pointer;
            transition: all 0.2s;
            font-size: 1.25rem;
            line-height: 1;
        }
        .radio-selector:checked + .radio-label {
            background-color: transparent;
            border-color: transparent;
        }
        .radio-selector:checked + .radio-label::before {
            content: '👑';
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }

        /* --- Loupe, Poubelle, D-Pad (conservés) --- */
        #magnifier-glass { position: absolute; width: 150px; height: 150px; border: 3px solid #000; border-radius: 50%; overflow: hidden; pointer-events: none; z-index: 10; background-repeat: no-repeat; background-color: white; top: 0.5rem; right: 0.5rem; }
        .magnifier-crosshair-h, .magnifier-crosshair-v { position: absolute; background-color: rgba(255, 0, 0, 0.7); z-index: 11; }
        .magnifier-crosshair-h { top: 50%; left: 0; width: 100%; height: 1px; transform: translateY(-50%); }
        .magnifier-crosshair-v { left: 50%; top: 0; height: 100%; width: 1px; transform: translateX(-50%); }
        #trashCan { position: absolute; top: 0.5rem; left: 0.5rem; width: 3rem; height: 3rem; background-color: rgba(255, 255, 255, 0.8); border-radius: 0.5rem; box-shadow: 0 2px 4px rgba(0,0,0,0.1); display: flex; align-items: center; justify-content: center; font-size: 1.5rem; transition: transform 0.2s, background-color 0.2s; z-index: 6; cursor: pointer; }
        #trashCan.hovering { transform: scale(1.1); background-color: #fecaca; }
        #dpad-controls { position: absolute; bottom: 0.5rem; right: 0.5rem; width: 6rem; height: 6rem; display: grid; grid-template-columns: 1fr 1fr 1fr; grid-template-rows: 1fr 1fr 1fr; z-index: 7; background-color: rgba(255, 255, 255, 0.5); border-radius: 50%; transition: background-color 0.2s, left 0.2s ease-in-out, right 0.2s ease-in-out, top 0.2s ease-in-out, bottom 0.2s ease-in-out; }
        .dpad-btn { background-color: rgba(243, 244, 246, 0.8); border: 1px solid rgba(209, 213, 219, 0.9); font-size: 1.25rem; font-weight: bold; color: #374151; transition: background-color 0.15s; }
        .dpad-btn:active { background-color: #d1d5db; }
        #dpad-up { grid-area: 1 / 2 / 2 / 3; border-radius: 10px 10px 0 0;}
        #dpad-left { grid-area: 2 / 1 / 3 / 2; border-radius: 10px 0 0 10px;}
        #dpad-right { grid-area: 2 / 3 / 3 / 4; border-radius: 0 10px 10px 0;}
        #dpad-down { grid-area: 3 / 2 / 4 / 3; border-radius: 0 0 10px 10px;}
        #up-arrow { position: absolute; top: 0.5rem; left: 50%; transform: translateX(-50%); width: 3rem; height: 3rem; background-color: rgba(255, 255, 255, 0.8); border-radius: 0.5rem; box-shadow: 0 2px 4px rgba(0,0,0,0.1); display: flex; align-items: center; justify-content: center; z-index: 6; pointer-events: none; }
    
        #bluetoothSearchBtn {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 0.5rem;
        }

        .options-wrapper {
            margin-top: auto; /* Pousse le bouton en bas quand le conteneur est grand */
            padding-top: 2rem; /* Ajoute de l'espace au-dessus quand le contenu est affiché */
        }
    </style>
</head>
<body>

    <div class="main-container no-image">
        <h1>(<span id="machineNameDisplay">nom de la machine</span>)</h1>

        <div class="header-controls">
            <label for="imageUpload" class="btn btn-primary">
                Capture
            </label>
            <input type="file" id="imageUpload" class="hidden" accept="image/*">
        </div>

        <div id="imageDisplayArea" class="hidden">
            <div id="imageContainer">
                <img id="imagePreview" alt="Aperçu de l'image"/>
                <div id="trashCan">🗑️</div>
                <div id="up-arrow" class="hidden">
                    <svg xmlns="http://www.w3.org/2000/svg" style="height: 2rem; width: 2rem; color: #374151;" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                      <path stroke-linecap="round" stroke-linejoin="round" d="M5 10l7-7m0 0l7 7m-7-7v18" />
                    </svg>
                </div>
                <div id="magnifier-glass" class="hidden">
                    <div class="magnifier-crosshair-h"></div>
                    <div class="magnifier-crosshair-v"></div>
                </div>
                <div id="dpad-controls">
                    <button id="dpad-up" class="dpad-btn">↑</button>
                    <button id="dpad-left" class="dpad-btn">←</button>
                    <button id="dpad-right" class="dpad-btn">→</button>
                    <button id="dpad-down" class="dpad-btn">↓</button>
                </div>
            </div>
        </div>
        
        <div id="controlsContainer" class="hidden">
             <p>Cliquez pour ajouter/verrouiller une fléxo et définir la référence.</p>
            <div id="marker-buttons-wrapper">
                <!-- Les boutons de marqueurs seront générés ici par le JS -->
            </div>
        </div>
        
        <div id="correctionsContainer" class="hidden">
            <div id="correctionsText">
                <!-- Les résultats s'afficheront ici -->
            </div>
            <button id="sendToScreenBtn" class="btn btn-info">Envoyer à l'écran</button>
        </div>

        <div class="options-wrapper">
            <button id="optionsBtn" class="btn btn-secondary">Options</button>
        </div>
    </div>

    <!-- Fenêtre Modale pour les Options -->
    <div id="optionsModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="optionsModal" class="modal hidden">
        <div class="modal-header">
            <h2>Options</h2>
            <button id="closeOptionsModalBtn" class="modal-close-btn">&times;</button>
        </div>
        <div class="modal-body">
            <div>
                <label for="markerSizeSlider">Taille des marqueurs: <span id="sliderValueSpan">5</span></label>
                <input type="range" id="markerSizeSlider" min="1" max="10" value="5">
            </div>
            <div>
                <label>Couleur des éléments</label>
                <div id="color-palette">
                    <!-- Les pastilles de couleur seront injectées ici par le JS -->
                </div>
            </div>
             <div>
                <button id="bluetoothSearchBtn" class="btn btn-primary full-width">
                    <span id="bluetoothBtnText">Rechercher via Bluetooth</span>
                    <span id="bluetoothStatus" style="width: 0.75rem; height: 0.75rem; background-color: var(--color-danger); border-radius: 9999px; border: 2px solid white;"></span>
                </button>
            </div>
            <div>
                <button id="openSettingsBtn" class="btn btn-light full-width">Paramètres</button>
            </div>
        </div>
    </div>
    
    <!-- Fenêtre Modale pour le Mot de Passe -->
    <div id="passwordModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="passwordModal" class="modal hidden">
        <h2 style="margin-bottom: 1rem;">Accès aux Paramètres</h2>
        <p style="margin-bottom: 1rem;">Appuyez sur Entrée ou Valider pour continuer.</p>
        <input type="password" id="passwordInput" style="margin-bottom: 1rem;">
        <div class="modal-actions">
            <button id="cancelPasswordBtn" class="btn btn-light">Annuler</button>
            <button id="submitPasswordBtn" class="btn btn-primary">Valider</button>
        </div>
    </div>

    <!-- Fenêtre Modale pour les Paramètres -->
    <div id="settingsModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="settingsModal" class="modal hidden">
        <div class="modal-header flex-shrink-0">
            <h2>Paramètres</h2>
            <button id="closeSettingsModalBtn" class="modal-close-btn">&times;</button>
        </div>
        <div class="modal-body">
            <div>
                <label for="machineNameInput">Nom de la machine</label>
                <input type="text" id="machineNameInput">
            </div>
             <div>
                <label for="flexoCountSlider">Nombre de flexos (F): <span id="flexoCountValue">6</span></label>
                <input type="range" id="flexoCountSlider" min="1" max="9" value="6">
            </div>
            <div>
                <label for="pixelToMmRatioInput">Pixels par millimètre (px/mm)</label>
                <input type="number" id="pixelToMmRatioInput" value="10">
            </div>
            <div>
                <label for="bluetoothDeviceNameInput">Nom de l'appareil Bluetooth</label>
                <input type="text" id="bluetoothDeviceNameInput" placeholder="Ex: ESP32_Printer">
            </div>
             <div class="checkbox-group" style="margin-top: 1rem; justify-content: flex-start;">
                <input type="checkbox" id="saveImageCheckbox">
                <label for="saveImageCheckbox">Sauvegarde image</label>
            </div>
            <div>
                <label>Gestion de la Configuration</label>
                <div class="button-group">
                    <button id="importConfigBtn" class="btn btn-primary full-width">Importer</button>
                    <button id="exportConfigBtn" class="btn btn-success full-width">Exporter</button>
                </div>
                <input type="file" id="configFileUpload" class="hidden" accept=".json, .txt">
            </div>
            <div>
                <button id="openSequenceEditorBtn" class="btn btn-light full-width">Éditeur de Séquence</button>
            </div>
        </div>
    </div>
    
    <!-- Fenêtre Modale pour l'Éditeur de Séquence -->
    <div id="sequenceEditorModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="sequenceEditorModal" class="modal hidden">
        <div class="modal-header">
            <h2>Éditeur de Séquence</h2>
            <button id="addSequenceActionBtn" class="btn-primary" style="border-radius: 9999px; height: 2rem; width: 2rem; display: flex; align-items: center; justify-content: center; font-size: 1.5rem; font-weight: bold; padding: 0;">+</button>
        </div>
        <div id="sequence-fields-container" class="modal-body">
             <!-- Les champs de coordonnées seront générés ici -->
        </div>
        <div class="modal-footer">
             <button id="saveSequenceBtn" class="btn btn-success">
                 <svg xmlns="http://www.w3.org/2000/svg" style="height: 1.25rem; width: 1.25rem; vertical-align: middle; margin-right: 0.5rem;" viewBox="0 0 20 20" fill="currentColor">
                   <path d="M7.707 10.293a1 1 0 10-1.414 1.414l3 3a1 1 0 001.414 0l3-3a1 1 0 00-1.414-1.414L11 11.586V6a1 1 0 10-2 0v5.586l-1.293-1.293zM5 4a2 2 0 012-2h6a2 2 0 012 2v2a1 1 0 102 0V4a4 4 0 00-4-4H7a4 4 0 00-4 4v12a4 4 0 004 4h6a4 4 0 004-4v-2a1 1 0 10-2 0v2a2 2 0 01-2 2H7a2 2 0 01-2-2V4z" />
                 </svg>
                 <span>Enregistrer</span>
               </button>
             <button id="closeSequenceEditorBtn" class="btn btn-light">Fermer</button>
        </div>
    </div>
    
    <!-- Fenêtre Modale pour l'affichage de la Séquence -->
    <div id="sequenceDisplayModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="sequenceDisplayModal" class="modal hidden" style="max-width: 32rem;">
        <div class="modal-header">
            <h2>Séquence Générée</h2>
            <button id="closeSequenceDisplayBtn" class="modal-close-btn">&times;</button>
        </div>
        <pre id="sequence-output" style="background-color: #F9FAFB; padding: 1rem; border-radius: 0.25rem; text-align: left; font-size: 0.875rem; white-space: pre-wrap; height: 16rem; overflow-y: auto;"></pre>
    </div>

    <!-- Fenêtre Modale de Confirmation -->
    <div id="confirmationModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="confirmationModal" class="modal hidden" style="max-width: 24rem; text-align: center;">
        <h2 style="margin-bottom: 1rem;">Confirmation</h2>
        <p style="margin-bottom: 1.5rem;">Êtes-vous sûr de vouloir envoyer ces valeurs à l'écran ?</p>
        <div class="confirmation-actions">
            <div class="button-group centered" style="gap: 1rem;">
                <button id="cancelSendBtn" class="btn btn-light" style="padding-left: 1.5rem; padding-right: 1.5rem;">Non</button>
                <button id="confirmSendBtn" class="btn btn-success" style="padding-left: 1.5rem; padding-right: 1.5rem;">Oui</button>
            </div>
            <button id="stopInputBtn" class="btn btn-danger">Arrêter la saisie</button>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // --- Récupération des éléments du DOM (partie 1) ---
            const mainContainer = document.querySelector('.main-container');
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
            const saveImageCheckbox = document.getElementById('saveImageCheckbox');
            const pixelToMmRatioInput = document.getElementById('pixelToMmRatioInput');
            const importConfigBtn = document.getElementById('importConfigBtn');
            const exportConfigBtn = document.getElementById('exportConfigBtn');
            const configFileUpload = document.getElementById('configFileUpload');
            
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
            let imageBlobForSaving = null;
            let magnifierHideTimer = null;
            let lastValidPosition = null;
            let sequence = [];
            
            const colorPaletteContainer = document.getElementById('color-palette');
            const colors = [
                '#FF0000', '#00FF00', '#0000FF', '#FFFF00', '#FF00FF',
                '#00FFFF', '#FF4500', '#7FFF00', '#9400D3', '#00FA9A'
            ];

            colors.forEach((color, index) => {
                const swatch = document.createElement('div');
                swatch.className = 'color-swatch';
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
                    saveImage: saveImageCheckbox.checked,
                    sequence: sequence,
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
                if (typeof config.saveImage === 'boolean') {
                    saveImageCheckbox.checked = config.saveImage;
                }
                if (config.sequence) {
                    sequence = config.sequence;
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
                resetSession(true); // Passer true pour ne pas réinitialiser la mise en page
                mainContainer.classList.remove('no-image');
                imagePreviewElement.src = imageURL;
                 if (blob) {
                    imageBlobForSaving = blob;
                 } else {
                    imageBlobForSaving = null;
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
                            btn.classList.remove('reference');
                        });
                        if (event.target.checked) {
                            const targetButtonId = event.target.dataset.targetButton;
                            const targetButton = document.querySelector(`.marker-btn[data-marker-id="${targetButtonId}"]`);
                            if (targetButton) {
                                targetButton.classList.add('reference');
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
                        <div class="marker-btn-container">
                            <button data-marker-id="F${i}" class="marker-btn">F${i}</button>
                            <div><input type="radio" name="marker-select" id="radio-F${i}" data-target-button="F${i}" class="radio-selector"><label for="radio-F${i}" class="radio-label"></label></div>
                        </div>`;
                }
                htmlContent += `
                    <div class="marker-btn-container">
                        <button data-marker-id="D" class="marker-btn">D</button>
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
                   correspondingButton.classList.add('active-marker-highlight');
                }
                updateCorrections();
            }
            
            function updateCorrections() {
                correctionsText.innerHTML = '';
                sendToScreenBtn.classList.add('hidden');

                const pixelToMmRatio = parseFloat(pixelToMmRatioInput.value) || 1;

                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (!selectedRadio) {
                    correctionsText.innerHTML = '<p>Sélectionnez une référence pour voir les corrections.</p>';
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainMarkerId = selectedRadio.dataset.targetButton;
                const mainMarker = document.querySelector(`.draggable-marker[data-marker-id="${mainMarkerId}"]`);
                if (!mainMarker) {
                    correctionsText.innerHTML = `<p>Ajoutez le marqueur de référence ${mainMarkerId} sur l'image.</p>`;
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainX = parseFloat(mainMarker.style.left);
                const mainY = parseFloat(mainMarker.style.top);
                const otherMarkers = document.querySelectorAll('.draggable-marker:not([data-marker-id="' + mainMarkerId + '"])');
                if (otherMarkers.length === 0) {
                     correctionsText.innerHTML = '<p>Ajoutez d\'autres marqueurs pour calculer les corrections.</p>';
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

            function showModal(modal, backdrop) {
                modal.classList.remove('hidden');
                backdrop.classList.remove('hidden');
            }
            function closeModal(modal, backdrop) {
                modal.classList.add('hidden');
                backdrop.classList.add('hidden');
            }
            
            optionsBtn.addEventListener('click', () => showModal(optionsModal, optionsModalBackdrop));
            closeOptionsModalBtn.addEventListener('click', () => closeModal(optionsModal, optionsModalBackdrop));
            optionsModalBackdrop.addEventListener('click', () => closeModal(optionsModal, optionsModalBackdrop));

            markerSizeSlider.addEventListener('input', (event) => {
                const value = event.target.value;
                sliderValueSpan.textContent = value;
                updateMarkerSizes(value);
            });
            
            function resetSession(keepLayout = false) {
                document.querySelectorAll('.draggable-marker').forEach(marker => marker.remove());
                document.querySelectorAll('.marker-btn').forEach(btn => {
                    btn.classList.remove('active-marker-highlight', 'reference');
                });
                document.querySelectorAll('.radio-selector').forEach(radio => radio.checked = false);
                lastSelectedMarker = null;
                updateCorrections();
                
                if (!keepLayout) {
                    mainContainer.classList.add('no-image');
                    imageDisplayArea.classList.add('hidden');
                    controlsContainer.classList.add('hidden');
                    correctionsContainer.classList.add('hidden');
                    upArrow.classList.add('hidden');
                }
            }
            trashCan.addEventListener('click', () => resetSession(false));

            sendToScreenBtn.addEventListener('click', () => {
                stopInputBtn.disabled = true;
                showModal(confirmationModal, confirmationModalBackdrop);
            });

            cancelSendBtn.addEventListener('click', () => closeModal(confirmationModal, confirmationModalBackdrop));
            confirmationModalBackdrop.addEventListener('click', () => closeModal(confirmationModal, confirmationModalBackdrop));

            confirmSendBtn.addEventListener('click', () => {
                stopInputBtn.disabled = false;
                
                if (saveImageCheckbox.checked && imageBlobForSaving) {
                    const blobUrl = URL.createObjectURL(imageBlobForSaving);
                    const a = document.createElement('a');
                    a.href = blobUrl;
                    const machineName = machineNameInput.value.replace(/\s+/g, '_') || 'capture';
                    const date = new Date();
                    const timestamp = `${date.getFullYear()}${(date.getMonth() + 1).toString().padStart(2, '0')}${date.getDate().toString().padStart(2, '0')}_${date.getHours().toString().padStart(2, '0')}${date.getMinutes().toString().padStart(2, '0')}`;
                    a.download = `easy-register_${machineName}_${timestamp}.png`;
                    document.body.appendChild(a);
                    a.click();
                    document.body.removeChild(a);
                    URL.revokeObjectURL(blobUrl);
                }
                
                generateAndShowSequence();
                
                closeModal(confirmationModal, confirmationModalBackdrop);

                const originalText = sendToScreenBtn.textContent;
                sendToScreenBtn.textContent = 'Envoyé !';
                sendToScreenBtn.classList.remove('btn-info');
                sendToScreenBtn.classList.add('btn-success');
                setTimeout(() => {
                    sendToScreenBtn.textContent = originalText;
                    sendToScreenBtn.classList.remove('btn-success');
                    sendToScreenBtn.classList.add('btn-info');
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
                showModal(passwordModal, passwordModalBackdrop);
                passwordInput.focus();
            });
            
            openSequenceEditorBtn.addEventListener('click', () => {
                populateSequenceEditor();
                closeModal(settingsModal, settingsModalBackdrop);
                showModal(sequenceEditorModal, sequenceEditorModalBackdrop);
            });

            closeSequenceEditorBtn.addEventListener('click', () => closeModal(sequenceEditorModal, sequenceEditorModalBackdrop));
            closeSequenceDisplayBtn.addEventListener('click', () => closeModal(sequenceDisplayModal, sequenceDisplayModalBackdrop));

            async function checkPassword(inputPassword) {
                const storedHash = '36a9e7f1c95b82ffb99743e0c5c4ce95d83c9a430aac59f84ef3cbfab6145068'; // SHA-256 hash for " " (a single space)
                try {
                    const encoder = new TextEncoder();
                    const data = encoder.encode(inputPassword);
                    const hashBuffer = await crypto.subtle.digest('SHA-256', data);
                    const hashArray = Array.from(new Uint8Array(hashBuffer));
                    const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
                    return hashHex === storedHash;
                } catch (e) {
                    console.error("Crypto API non disponible:", e);
                    alert("La vérification du mot de passe n'est pas possible dans ce contexte (non-sécurisé).");
                    return false;
                }
            }

            async function handlePasswordSubmit() {
                if (await checkPassword(passwordInput.value)) {
                    openSettings();
                } else {
                    passwordInput.value = ''; // Clear incorrect password
                    // L'alerte est déjà dans checkPassword en cas d'échec
                }
            }
            
            function openSettings() {
                closeModal(passwordModal, passwordModalBackdrop);
                showModal(settingsModal, settingsModalBackdrop);
            }

            cancelPasswordBtn.addEventListener('click', () => closeModal(passwordModal, passwordModalBackdrop));
            submitPasswordBtn.addEventListener('click', handlePasswordSubmit);
            passwordInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') { handlePasswordSubmit(); } });
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
            
            saveImageCheckbox.addEventListener('change', saveState);
            
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
                element.className = 'sequence-action-item';
                element.dataset.actionId = id;
                
                let content = `
                    <span class="drag-handle">⠿</span>
                    <input type="text" value="${name}" class="sequence-name-input">
                `;

                if (isIntermediate) {
                    content += `
                        <div class="button-group" style="margin-left: auto;">
                            <input type="number" placeholder="X" value="${x}" class="sequence-coord-input">
                            <input type="number" placeholder="Y" value="${y}" class="sequence-coord-input">
                        </div>
                        <button class="delete-btn">&times;</button>
                    `;
                } else {
                     content += `<span style="font-size: 0.875rem; color: var(--color-text-light); font-style: italic; margin-left: auto;">Saisie de valeur</span>`;
                }
                
                element.innerHTML = content;
                element.setAttribute('draggable', true);
                
                const nameInput = element.querySelector('.sequence-name-input');
                if (isIntermediate) {
                    element.querySelector('.delete-btn').addEventListener('click', () => {
                        sequence = sequence.filter(a => a.id !== id);
                        populateSequenceEditor();
                    });
                } else {
                    nameInput.disabled = true;
                }
                return element;
            }
            
            function saveSequenceFromEditor() {
                const newSequence = [];
                sequenceFieldsContainer.querySelectorAll('[data-action-id]').forEach(el => {
                    const id = el.dataset.actionId;
                    const originalAction = sequence.find(a => a.id === id);
                    const name = el.querySelector('.sequence-name-input').value;
                    const xInput = el.querySelectorAll('.sequence-coord-input')[0];
                    const yInput = el.querySelectorAll('.sequence-coord-input')[1];

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
                saveSequenceBtn.classList.remove('btn-success');
                saveSequenceBtn.classList.add('btn-primary');
                saveText.textContent = "Enregistré !";
                setTimeout(() => {
                    saveText.textContent = originalText;
                    saveSequenceBtn.classList.remove('btn-primary');
                    saveSequenceBtn.classList.add('btn-success');
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
                showModal(sequenceDisplayModal, sequenceDisplayModalBackdrop);
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
                    alert("L'API Web Bluetooth n'est pas supportée sur ce navigateur. Elle ne fonctionne que sur un site sécurisé (https).");
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

                    bluetoothStatus.style.backgroundColor = 'var(--color-success)';
                    bluetoothBtnText.textContent = device.name;
                    alert(`Connecté à ${device.name}`);

                } catch(error) {
                    console.error('Erreur Bluetooth:', error);
                    alert(`Erreur Bluetooth: ${error.message}. Assurez-vous d'être sur une page sécurisée (https).`);
                }
            });

            function onDisconnected() {
                console.log('Appareil déconnecté.');
                bluetoothStatus.style.backgroundColor = 'var(--color-danger)';
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

            function resetMagnifierPosition() {
                magnifierGlass.style.top = '0.5rem';
                magnifierGlass.style.right = '0.5rem';
                magnifierGlass.style.left = 'auto';
                magnifierGlass.style.bottom = 'auto';
            }

            function handleMagnifierRepositioning(markerRect) {
                const containerRect = imageContainer.getBoundingClientRect();
                const margin = 8;
                const glassWidth = 150;
                const glassHeight = 150;

                const corners = {
                    topRight: {
                        style: { top: `${margin}px`, right: `${margin}px`, left: 'auto', bottom: 'auto' },
                        rect: { top: margin, left: containerRect.width - glassWidth - margin, width: glassWidth, height: glassHeight }
                    },
                    topLeft: {
                        style: { top: `${margin}px`, left: `${margin}px`, right: 'auto', bottom: 'auto' },
                        rect: { top: margin, left: margin, width: glassWidth, height: glassHeight }
                    },
                    bottomRight: {
                        style: { bottom: `${margin}px`, right: `${margin}px`, top: 'auto', left: 'auto' },
                        rect: { top: containerRect.height - glassHeight - margin, left: containerRect.width - glassWidth - margin, width: glassWidth, height: glassHeight }
                    },
                    bottomLeft: {
                        style: { bottom: `${margin}px`, left: `${margin}px`, top: 'auto', right: 'auto' },
                        rect: { top: containerRect.height - glassHeight - margin, left: margin, width: glassWidth, height: glassHeight }
                    }
                };
                
                for (const key in corners) {
                    corners[key].rect.right = corners[key].rect.left + corners[key].rect.width;
                    corners[key].rect.bottom = corners[key].rect.top + corners[key].rect.height;
                }

                const relativeMarkerRect = {
                    top: markerRect.top - containerRect.top,
                    left: markerRect.left - containerRect.left,
                    width: markerRect.width,
                    height: markerRect.height
                };
                relativeMarkerRect.right = relativeMarkerRect.left + relativeMarkerRect.width;
                relativeMarkerRect.bottom = relativeMarkerRect.top + relativeMarkerRect.height;

                const topRightObstructed = checkCollision(relativeMarkerRect, corners.topRight.rect);
                
                let chosenCorner = corners.topRight;

                if (topRightObstructed) {
                    if (!checkCollision(relativeMarkerRect, corners.topLeft.rect)) {
                        chosenCorner = corners.topLeft;
                    } else if (!checkCollision(relativeMarkerRect, corners.bottomRight.rect)) {
                        chosenCorner = corners.bottomRight;
                    } else {
                        chosenCorner = corners.bottomLeft;
                    }
                }
                
                Object.assign(magnifierGlass.style, chosenCorner.style);
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
                handleMagnifierRepositioning(lastSelectedMarker.getBoundingClientRect());

                magnifierHideTimer = setTimeout(() => {
                    magnifierGlass.classList.add('hidden');
                }, 3000);
                updateCorrections();
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
                        if (button) button.classList.remove('active-marker-highlight');
                        const radio = document.querySelector(`#radio-${markerId}`);
                        if (radio && radio.checked) {
                            radio.checked = false;
                            if (button) {
                                button.classList.remove('reference');
                            }
                        }
                    }

                    resetToolPositions();
                    resetMagnifierPosition();
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
                handleMagnifierRepositioning(markerRect);
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
                    saveImage: saveImageCheckbox.checked,
                    sequence: sequence
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
                event.target.value = '';
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


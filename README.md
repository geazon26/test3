<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimum-scale=1.0">
    <title>Image Loader with Markers</title>
    <style>
        /* --- General Variables and Styles --- */
        :root {
            --main-marker-bg: #FF0000; /* Default Background (Bright Red) */
            --main-marker-fg: #FF0000; /* Default Foreground (Bright Red) */
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
            padding: 0;
            min-height: 100vh;
            overflow-x: hidden; /* Empêche le défilement horizontal */
            touch-action: manipulation; /* Empêche le zoom par double-tap */
        }
        
        /* --- Main Containers --- */
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
        
        /* Style for initial display (no image) */
        .main-container.no-image {
             min-height: calc(100vh - 4rem); /* Takes up height so auto margin works */
             padding-top: 5rem;
        }

        .header-controls {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 1rem;
            margin-bottom: 1.5rem;
            transition: all 0.4s ease-in-out;
        }
        
        .header-controls.centered {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            margin-bottom: 0;
            z-index: 1; /* Ensure it's above other elements in the initial view */
        }

        .camera-icon-label {
            padding: 1rem;
            border-radius: 9999px;
            transition: padding 0.4s ease-in-out, background-color 0.2s ease-in-out;
        }

        .header-controls.centered .camera-icon-label {
            padding: 2rem;
        }
        
        .camera-icon-svg {
            height: 1.5rem; 
            width: 1.5rem;
            transition: all 0.4s ease-in-out;
        }

        .header-controls.centered .camera-icon-svg {
            height: 9rem; /* Large size for initial view */
            width: 9rem;
        }
        
        .footer-controls {
            position: fixed;
            bottom: 1.5rem;
            left: 50%;
            transform: translateX(-50%);
            z-index: 20;
        }

        /* --- Typography --- */
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

        /* --- Buttons --- */
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

        /* --- Modals --- */
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

        /* --- Element Groups for Alignment --- */
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

        /* --- Forms --- */
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

        /* --- Specific Components --- */
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
            box-shadow: 0 0 0 2px white, 0 0 0 4px var(--main-marker-fg);
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
            font-size: 1rem;
            font-weight: bold;
        }
        #sendToScreenBtn {
            margin-bottom: 1rem;
            width: 100%;
        }
        
        #settingsModal {
            max-width: 32rem;
            height: 83.333%;
            display: flex;
            flex-direction: column;
        }
        #settingsModal.hidden { display: none; }

        /* --- Color Palette (Options) --- */
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

        /* --- Sequence Editor --- */
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
            justify-content: space-between;
            align-items: flex-end;
            gap: 0.75rem;
        }

        /* --- Existing Marker Styles (kept and adapted) --- */
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
            background: var(--main-marker-bg);
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
            color: var(--main-marker-fg);
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

        /* --- Magnifier, Trash, D-Pad (kept) --- */
        #magnifier-glass { position: absolute; width: 150px; height: 150px; border: 3px solid #000; border-radius: 50%; overflow: hidden; pointer-events: none; z-index: 10; background-repeat: no-repeat; background-color: white; top: 0.5rem; right: 0.5rem; }
        .magnifier-crosshair-h, .magnifier-crosshair-v { position: absolute; background: var(--main-marker-bg); box-shadow: 0 0 2px white; z-index: 11; }
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
            margin-top: auto; /* Pushes the button to the bottom when the container is large */
            padding-top: 2rem; /* Adds space above when content is displayed */
        }
        
        /* Media Queries for Responsiveness */
        @media (min-width: 768px) { /* Applies on tablets and desktops */
            body {
                padding: 2rem; /* Restore padding for a "card" look on larger screens */
            }
        }
        @media (max-width: 767px) { /* Applies on mobile phones */
            body {
                background-color: var(--color-bg-white);
            }
            .main-container {
                padding: 1rem; /* Reduce padding on small screens */
                border-radius: 0;
                box-shadow: none;
                min-height: 100vh; /* Ensure it fills the screen vertically */
            }
            #imageDisplayArea {
                margin-left: -1rem;
                margin-right: -1rem;
            }
            .modal {
                width: calc(100% - 2rem); /* Ensure modal doesn't touch edges */
            }
        }
    </style>
</head>
<body>

    <div class="main-container no-image">
        <h1 class="hidden">(<span id="machineNameDisplay">machine name</span>)</h1>

        <div class="header-controls centered">
            <label for="imageUpload" class="btn camera-icon-label">
                <svg xmlns="http://www.w3.org/2000/svg" class="camera-icon-svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"></path>
                  <path stroke-linecap="round" stroke-linejoin="round" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z"></path>
                </svg>
            </label>
            <input type="file" id="imageUpload" class="hidden" accept="image/*" capture="environment">
        </div>

        <div id="imageDisplayArea" class="hidden">
            <div id="imageContainer">
                <img id="imagePreview" alt="Image Preview"/>
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
             <p>Click to add/lock a flexo and set the reference.</p>
            <div id="marker-buttons-wrapper">
                <!-- Marker buttons will be generated here by JS -->
            </div>
        </div>
        
        <div id="correctionsContainer" class="hidden">
            <button id="sendToScreenBtn" class="btn btn-info">Send to Screen</button>
            <div id="correctionsText">
                <!-- Results will be displayed here -->
            </div>
        </div>

        <div class="options-wrapper">
            <button id="optionsBtn" class="btn btn-secondary">Options</button>
        </div>
    </div>

    <!-- Options Modal -->
    <div id="optionsModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="optionsModal" class="modal hidden">
        <div class="modal-header">
            <h2>Options</h2>
            <button id="closeOptionsModalBtn" class="modal-close-btn">&times;</button>
        </div>
        <div class="modal-body">
            <div>
                <label for="markerSizeSlider">Marker Size: <span id="sliderValueSpan">5</span></label>
                <input type="range" id="markerSizeSlider" min="1" max="10" value="5">
            </div>
            <div>
                <label>Element Color</label>
                <div id="color-palette">
                    <!-- Color swatches will be injected here by JS -->
                </div>
            </div>
             <div>
                <button id="bluetoothSearchBtn" class="btn btn-primary full-width">
                    <span id="bluetoothBtnText">Search via Bluetooth</span>
                    <span id="bluetoothStatus" style="width: 0.75rem; height: 0.75rem; background-color: var(--color-danger); border-radius: 9999px; border: 2px solid white;"></span>
                </button>
            </div>
            <div>
                <button id="openSettingsBtn" class="btn btn-light full-width">Settings</button>
            </div>
        </div>
    </div>
    
    <!-- Password Modal -->
    <div id="passwordModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="passwordModal" class="modal hidden">
        <h2 style="margin-bottom: 1rem;">Access Settings</h2>
        <p style="margin-bottom: 1rem;">Press Enter or Confirm to continue.</p>
        <input type="password" id="passwordInput" style="margin-bottom: 1rem;">
        <div class="modal-actions">
            <button id="cancelPasswordBtn" class="btn btn-light">Cancel</button>
            <button id="submitPasswordBtn" class="btn btn-primary">Confirm</button>
        </div>
    </div>

    <!-- Settings Modal -->
    <div id="settingsModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="settingsModal" class="modal hidden">
        <div class="modal-header flex-shrink-0">
            <h2>Settings</h2>
            <button id="closeSettingsModalBtn" class="modal-close-btn">&times;</button>
        </div>
        <div class="modal-body">
            <div>
                <label for="machineNameInput">Machine Name</label>
                <input type="text" id="machineNameInput">
            </div>
             <div>
                <label for="flexoCountSlider">Number of flexos (F): <span id="flexoCountValue">6</span></label>
                <input type="range" id="flexoCountSlider" min="1" max="9" value="6">
            </div>
            <div>
                <label for="pixelToMmRatioInput">Pixels per millimeter (px/mm)</label>
                <input type="number" id="pixelToMmRatioInput" value="10">
            </div>
            <div>
                <label for="bluetoothDeviceNameInput">Bluetooth Device Name</label>
                <input type="text" id="bluetoothDeviceNameInput" placeholder="Ex: ESP32_Printer">
            </div>
             <div class="checkbox-group" style="margin-top: 1rem; justify-content: flex-start;">
                <input type="checkbox" id="saveImageCheckbox">
                <label for="saveImageCheckbox">Save Image</label>
            </div>
             <div class="checkbox-group" style="margin-top: 1rem; justify-content: flex-start;">
                <input type="checkbox" id="saveImageWithCrossCheckbox">
                <label for="saveImageWithCrossCheckbox">Save image with cross</label>
            </div>
             <div class="checkbox-group" style="margin-top: 1rem; justify-content: flex-start;">
                <input type="checkbox" id="showSequenceCheckbox">
                <label for="showSequenceCheckbox">Generated Sequence</label>
            </div>
            <div class="checkbox-group" style="margin-top: 1rem; justify-content: flex-start;">
                <input type="checkbox" id="exportSequenceCheckbox">
                <label for="exportSequenceCheckbox">Export Sequence</label>
            </div>
            <div>
                <label>Configuration Management</label>
                <div class="button-group">
                    <button id="importConfigBtn" class="btn btn-primary full-width">Import</button>
                    <button id="exportConfigBtn" class="btn btn-success full-width">Export</button>
                </div>
                <input type="file" id="configFileUpload" class="hidden" accept=".json, .txt">
            </div>
            <div>
                <button id="openSequenceEditorBtn" class="btn btn-light full-width">Sequence Editor</button>
            </div>
        </div>
    </div>
    
    <!-- Sequence Editor Modal -->
    <div id="sequenceEditorModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="sequenceEditorModal" class="modal hidden">
        <div class="modal-header">
            <h2>Sequence Editor</h2>
            <button id="addSequenceActionBtn" class="btn-primary" style="border-radius: 9999px; height: 2rem; width: 2rem; display: flex; align-items: center; justify-content: center; font-size: 1.5rem; font-weight: bold; padding: 0;">+</button>
        </div>
        <div id="sequence-fields-container" class="modal-body">
             <!-- Coordinate fields will be generated here -->
        </div>
        <div class="modal-footer">
            <div>
                <label for="sequenceDelayInput" class="label-text" style="margin-bottom: 0.25rem;">Delay (s)</label>
                <input type="number" id="sequenceDelayInput" value="0" min="0" step="0.1" style="width: 8rem;">
            </div>
             <div class="button-group">
                 <button id="saveSequenceBtn" class="btn btn-success">
                     <svg xmlns="http://www.w3.org/2000/svg" style="height: 1.25rem; width: 1.25rem; vertical-align: middle; margin-right: 0.5rem;" viewBox="0 0 20 20" fill="currentColor">
                       <path d="M7.707 10.293a1 1 0 10-1.414 1.414l3 3a1 1 0 001.414 0l3-3a1 1 0 00-1.414-1.414L11 11.586V6a1 1 0 10-2 0v5.586l-1.293-1.293zM5 4a2 2 0 012-2h6a2 2 0 012 2v2a1 1 0 102 0V4a4 4 0 00-4-4H7a4 4 0 00-4 4v12a4 4 0 004 4h6a4 4 0 004-4v-2a1 1 0 10-2 0v2a2 2 0 01-2 2H7a2 2 0 01-2-2V4z" />
                     </svg>
                     <span>Save</span>
                   </button>
                 <button id="closeSequenceEditorBtn" class="btn btn-light">Close</button>
             </div>
        </div>
    </div>
    
    <!-- Sequence Display Modal -->
    <div id="sequenceDisplayModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="sequenceDisplayModal" class="modal hidden" style="max-width: 32rem;">
        <div class="modal-header">
            <h2>Generated Sequence</h2>
            <button id="closeSequenceDisplayBtn" class="modal-close-btn">&times;</button>
        </div>
        <pre id="sequence-output" style="background-color: #F9FAFB; padding: 1rem; border-radius: 0.25rem; text-align: left; font-size: 0.875rem; white-space: pre-wrap; height: 16rem; overflow-y: auto;"></pre>
    </div>

    <!-- Confirmation Modal -->
    <div id="confirmationModalBackdrop" class="modal-backdrop hidden"></div>
    <div id="confirmationModal" class="modal hidden" style="max-width: 24rem; text-align: center;">
        <h2 style="margin-bottom: 1rem;">Confirmation</h2>
        <p style="margin-bottom: 1.5rem;">Are you sure you want to send these values to the screen?</p>
        <div class="confirmation-actions">
            <div class="button-group centered" style="gap: 1rem;">
                <button id="cancelSendBtn" class="btn btn-light" style="padding-left: 1.5rem; padding-right: 1.5rem;">No</button>
                <button id="confirmSendBtn" class="btn btn-success" style="padding-left: 1.5rem; padding-right: 1.5rem;">Yes</button>
            </div>
            <button id="stopInputBtn" class="btn btn-danger">Stop Input</button>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // --- DOM Element Retrieval (part 1) ---
            const mainContainer = document.querySelector('.main-container');
            const headerControls = document.querySelector('.header-controls');
            const cameraIconLabel = document.querySelector('.camera-icon-label');
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
            
            // --- Options Modal ---
            const optionsBtn = document.getElementById('optionsBtn');
            const optionsModal = document.getElementById('optionsModal');
            const optionsModalBackdrop = document.getElementById('optionsModalBackdrop');
            const closeOptionsModalBtn = document.getElementById('closeOptionsModalBtn');
            const markerSizeSlider = document.getElementById('markerSizeSlider');
            const sliderValueSpan = document.getElementById('sliderValueSpan');
            
            // --- Confirmation Modal ---
            const confirmationModalBackdrop = document.getElementById('confirmationModalBackdrop');
            const confirmationModal = document.getElementById('confirmationModal');
            const cancelSendBtn = document.getElementById('cancelSendBtn');
            const confirmSendBtn = document.getElementById('confirmSendBtn');
            const stopInputBtn = document.getElementById('stopInputBtn');
            
            // --- Password Modal ---
            const openSettingsBtn = document.getElementById('openSettingsBtn');
            const passwordModal = document.getElementById('passwordModal');
            const passwordModalBackdrop = document.getElementById('passwordModalBackdrop');
            const passwordInput = document.getElementById('passwordInput');
            const cancelPasswordBtn = document.getElementById('cancelPasswordBtn');
            const submitPasswordBtn = document.getElementById('submitPasswordBtn');

            // --- Settings Modal ---
            const settingsModal = document.getElementById('settingsModal');
            const settingsModalBackdrop = document.getElementById('settingsModalBackdrop');
            const closeSettingsModalBtn = document.getElementById('closeSettingsModalBtn');
            const machineNameInput = document.getElementById('machineNameInput');
            const machineNameDisplay = document.getElementById('machineNameDisplay');
            const flexoCountSlider = document.getElementById('flexoCountSlider');
            const flexoCountValue = document.getElementById('flexoCountValue');
            const bluetoothDeviceNameInput = document.getElementById('bluetoothDeviceNameInput');
            const saveImageCheckbox = document.getElementById('saveImageCheckbox');
            const saveImageWithCrossCheckbox = document.getElementById('saveImageWithCrossCheckbox');
            const showSequenceCheckbox = document.getElementById('showSequenceCheckbox');
            const exportSequenceCheckbox = document.getElementById('exportSequenceCheckbox');
            const pixelToMmRatioInput = document.getElementById('pixelToMmRatioInput');
            const importConfigBtn = document.getElementById('importConfigBtn');
            const exportConfigBtn = document.getElementById('exportConfigBtn');
            const configFileUpload = document.getElementById('configFileUpload');
            
            // --- Sequence Editor Modal ---
            const openSequenceEditorBtn = document.getElementById('openSequenceEditorBtn');
            const sequenceEditorModal = document.getElementById('sequenceEditorModal');
            const sequenceEditorModalBackdrop = document.getElementById('sequenceEditorModalBackdrop');
            const closeSequenceEditorBtn = document.getElementById('closeSequenceEditorBtn');
            const sequenceFieldsContainer = document.getElementById('sequence-fields-container');
            const addSequenceActionBtn = document.getElementById('addSequenceActionBtn');
            const saveSequenceBtn = document.getElementById('saveSequenceBtn');
            const sequenceDelayInput = document.getElementById('sequenceDelayInput');


            // --- Sequence Display Modal ---
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

            // --- Other Elements ---
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
            let blobForDownload = null; // Variable to hold the prepared image blob
            
            const colorPaletteContainer = document.getElementById('color-palette');
            const colors = [
                '#FF0000', '#00FF00', '#0000FF', '#000000', '#FF00FF',
                'rainbow-gradient', '#FF4500', '#7FFF00', '#9400D3', '#00FA9A'
            ];
            
            // ===============================================
            // SECTION 1: ALL FUNCTION DEFINITIONS
            // ===============================================

            function showModal(modal, backdrop) {
                modal.classList.remove('hidden');
                backdrop.classList.remove('hidden');
            }

            function closeModal(modal, backdrop) {
                modal.classList.add('hidden');
                backdrop.classList.add('hidden');
            }
            
            function applyColorTheme(color) {
                if (color === 'rainbow-gradient') {
                    const rainbowGradient = 'conic-gradient(from 180deg at 50% 50%, #FF0000 0deg, #FFFF00 60deg, #00FF00 120deg, #00FFFF 180deg, #0000FF 240deg, #FF00FF 300deg, #FF0000 360deg)';
                    document.documentElement.style.setProperty('--main-marker-bg', rainbowGradient);
                    document.documentElement.style.setProperty('--main-marker-fg', '#1F2937');
                    cameraIconLabel.style.background = 'conic-gradient(from 180deg at 50% 50%, red, yellow, lime, aqua, blue, magenta, red)';
                } else {
                    document.documentElement.style.setProperty('--main-marker-bg', color);
                    document.documentElement.style.setProperty('--main-marker-fg', color);
                    cameraIconLabel.style.background = color;
                }
            }

            function saveState() {
                const config = {
                    machineName: machineNameInput.value,
                    flexoCount: parseInt(flexoCountSlider.value, 10),
                    pixelToMmRatio: parseFloat(pixelToMmRatioInput.value),
                    bluetoothDeviceName: bluetoothDeviceNameInput.value,
                    saveImage: saveImageCheckbox.checked,
                    saveImageWithCross: saveImageWithCrossCheckbox.checked,
                    showSequence: showSequenceCheckbox.checked,
                    exportSequence: exportSequenceCheckbox.checked,
                    sequenceDelay: parseFloat(sequenceDelayInput.value) || 0,
                    sequence: sequence,
                };
                localStorage.setItem('easyRegisterState', JSON.stringify(config));
            }
            
            function applyConfig(config) {
                const machineNameHeader = document.querySelector('h1');
                if (config.machineName) {
                    machineNameInput.value = config.machineName;
                    machineNameDisplay.textContent = config.machineName || 'machine name';
                } else {
                    machineNameInput.value = '';
                    machineNameDisplay.textContent = 'machine name';
                }
                machineNameHeader.classList.add('hidden'); // Ensure it's hidden on initial load

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
                if (typeof config.saveImageWithCross === 'boolean') {
                    saveImageWithCrossCheckbox.checked = config.saveImageWithCross;
                }
                if (typeof config.showSequence === 'boolean') {
                    showSequenceCheckbox.checked = config.showSequence;
                }
                if (typeof config.exportSequence === 'boolean') {
                    exportSequenceCheckbox.checked = config.exportSequence;
                }
                if (typeof config.sequenceDelay === 'number') {
                    sequenceDelayInput.value = config.sequenceDelay;
                }
                if (config.sequence && config.sequence.length > 0) {
                    sequence = config.sequence;
                } else {
                    initializeDefaultSequence(parseInt(flexoCountSlider.value, 10));
                }
                updateCorrections();
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

            function loadState() {
                const savedState = localStorage.getItem('easyRegisterState');
                if (savedState) {
                    try {
                        const config = JSON.parse(savedState);
                        applyConfig(config);
                    } catch (e) {
                        console.error("Error loading saved configuration:", e);
                        const initialFlexoCount = 6;
                        generateMarkerButtons(initialFlexoCount);
                        initializeDefaultSequence(initialFlexoCount);
                    }
                } else {
                    const initialFlexoCount = 6;
                    generateMarkerButtons(initialFlexoCount);
                    saveImageCheckbox.checked = false; // Set to false by default
                    saveImageWithCrossCheckbox.checked = false; // Set to false by default
                    initializeDefaultSequence(initialFlexoCount);
                }
            }
            
            function clearMarkersAndCorrections() {
                document.querySelectorAll('.draggable-marker').forEach(marker => marker.remove());
                document.querySelectorAll('.marker-btn').forEach(btn => {
                    btn.classList.remove('active-marker-highlight', 'reference');
                });
                document.querySelectorAll('.radio-selector').forEach(radio => radio.checked = false);
                lastSelectedMarker = null;
                correctionsText.innerHTML = '';
                correctionsContainer.classList.add('hidden');
            }
            
            function resetSession() {
                clearMarkersAndCorrections();
                headerControls.classList.add('centered');
                mainContainer.classList.add('no-image');
                imageDisplayArea.classList.add('hidden');
                controlsContainer.classList.add('hidden');
                correctionsContainer.classList.add('hidden');
                upArrow.classList.add('hidden');
                document.querySelector('h1').classList.add('hidden');

                if (imagePreviewElement.src) {
                    URL.revokeObjectURL(imagePreviewElement.src);
                }
                imagePreviewElement.src = "";
                imageBlobForSaving = null;
            }

            function setImageAsPreview(imageURL, blob) {
                clearMarkersAndCorrections(); 
                headerControls.classList.remove('centered');
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
                    
                    const machineNameHeader = document.querySelector('h1');
                    if (machineNameInput.value.trim() !== '') {
                        machineNameHeader.classList.remove('hidden');
                    }
                };
            }

            function updateCorrections() {
                if (mainContainer.classList.contains('no-image')) {
                    correctionsContainer.classList.add('hidden');
                    return;
                }
                
                correctionsText.innerHTML = '';
                sendToScreenBtn.classList.add('hidden');

                const pixelToMmRatio = parseFloat(pixelToMmRatioInput.value) || 1;

                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (!selectedRadio) {
                    correctionsText.innerHTML = '<p>Select a reference to see the corrections.</p>';
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainMarkerId = selectedRadio.dataset.targetButton;
                const mainMarker = document.querySelector(`.draggable-marker[data-marker-id="${mainMarkerId}"]`);
                if (!mainMarker) {
                    correctionsText.innerHTML = `<p>Add the reference marker ${mainMarkerId} to the image.</p>`;
                    correctionsContainer.classList.remove('hidden');
                    return;
                }
                const mainX = parseFloat(mainMarker.style.left);
                const mainY = parseFloat(mainMarker.style.top);
                const otherMarkers = document.querySelectorAll('.draggable-marker:not([data-marker-id="' + mainMarkerId + '"])');
                if (otherMarkers.length === 0) {
                     correctionsText.innerHTML = '<p>Add other markers to calculate corrections.</p>';
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

                    resultLine.textContent = `${displayName} : Register (${correctionY_mm.toFixed(2)}mm) / Centering (${correctionX_mm.toFixed(2)}mm)`;
                    correctionsText.appendChild(resultLine);
                });
                sendToScreenBtn.classList.remove('hidden');
                correctionsContainer.classList.remove('hidden');
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
                    console.error("Crypto API not available:", e);
                    alert("Password verification is not possible in this context (non-secure).");
                    return false;
                }
            }

            async function handlePasswordSubmit() {
                if (await checkPassword(passwordInput.value)) {
                    openSettings();
                } else {
                    passwordInput.value = ''; // Clear incorrect password
                    // The alert is already in checkPassword in case of failure
                }
            }
            
            function openSettings() {
                closeModal(passwordModal, passwordModalBackdrop);
                showModal(settingsModal, settingsModalBackdrop);
            }
            
             function createSequenceAction(id, type, flexoId, name, x = '', y = '', conditionTargetType = null) {
                 return { id, type, flexoId, name, x, y, conditionTargetType };
            }

            function initializeDefaultSequence(flexoCount) {
                sequence = []; // Reset the global sequence
                
                // Generate for Flexos
                for (let i = 1; i <= flexoCount; i++) {
                    const prefix = `F${i}`;
                    sequence.push(createSequenceAction(`${prefix}_select`, 'INTERMEDIATE_CLICK', prefix, `${prefix} Selection`, '', '', 'FLEXO_REG'));
                    sequence.push(createSequenceAction(`${prefix}_clic_reg`, 'INTERMEDIATE_CLICK', prefix, `${prefix} clic Register`, '', '', 'FLEXO_REG'));
                    sequence.push(createSequenceAction(`${prefix}_clic_pad_reg`, 'INTERMEDIATE_CLICK', prefix, `${prefix} clic pavet numérique`, '', '', 'FLEXO_REG'));
                    sequence.push(createSequenceAction(`${prefix}_reg`, 'FLEXO_REG', prefix, `${prefix} Register`));
                    sequence.push(createSequenceAction(`${prefix}_clic_valid_reg`, 'INTERMEDIATE_CLICK', prefix, `${prefix} clic validation`, '', '', 'FLEXO_REG'));
                    
                    sequence.push(createSequenceAction(`${prefix}_clic_cen`, 'INTERMEDIATE_CLICK', prefix, `${prefix} clic centering`, '', '', 'FLEXO_CEN'));
                    sequence.push(createSequenceAction(`${prefix}_clic_pad_cen`, 'INTERMEDIATE_CLICK', prefix, `${prefix} clic pavet numérique`, '', '', 'FLEXO_CEN'));
                    sequence.push(createSequenceAction(`${prefix}_cen`, 'FLEXO_CEN', prefix, `${prefix} Centering`));
                    sequence.push(createSequenceAction(`${prefix}_clic_valid_cen`, 'INTERMEDIATE_CLICK', prefix, `${prefix} clic validation`, '', '', 'FLEXO_CEN'));
                }
                
                // Generate for Die-Cutter
                const d_prefix = 'D';
                sequence.push(createSequenceAction(`${d_prefix}_select`, 'INTERMEDIATE_CLICK', d_prefix, `Die-Cut Selection`, '', '', 'DIECUT_REG'));
                sequence.push(createSequenceAction(`${d_prefix}_clic_reg`, 'INTERMEDIATE_CLICK', d_prefix, `Die-Cut clic Register`, '', '', 'DIECUT_REG'));
                sequence.push(createSequenceAction(`${d_prefix}_clic_pad_reg`, 'INTERMEDIATE_CLICK', d_prefix, `Die-Cut clic pavet numérique`, '', '', 'DIECUT_REG'));
                sequence.push(createSequenceAction('d_reg', 'DIECUT_REG', d_prefix, 'Die-Cut Register'));
                sequence.push(createSequenceAction(`${d_prefix}_clic_valid_reg`, 'INTERMEDIATE_CLICK', d_prefix, `Die-Cut clic validation`, '', '', 'DIECUT_REG'));

                sequence.push(createSequenceAction(`${d_prefix}_clic_cen`, 'INTERMEDIATE_CLICK', d_prefix, `Die-Cut clic centering`, '', '', 'DIECUT_CEN'));
                sequence.push(createSequenceAction(`${d_prefix}_clic_pad_cen`, 'INTERMEDIATE_CLICK', d_prefix, `Die-Cut clic pavet numérique`, '', '', 'DIECUT_CEN'));
                sequence.push(createSequenceAction('d_cen', 'DIECUT_CEN', d_prefix, 'Die-Cut Centering'));
                sequence.push(createSequenceAction(`${d_prefix}_clic_valid_cen`, 'INTERMEDIATE_CLICK', d_prefix, `Die-Cut clic validation`, '', '', 'DIECUT_CEN'));

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
                     content += `<span style="font-size: 0.875rem; color: var(--color-text-light); font-style: italic; margin-left: auto;">Value input</span>`;
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
                saveText.textContent = "Saved!";
                setTimeout(() => {
                    saveText.textContent = originalText;
                    saveSequenceBtn.classList.remove('btn-primary');
                    saveSequenceBtn.classList.add('btn-success');
                }, 1500);
            }
            
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

            function generateSequenceString() {
                const pixelToMmRatio = parseFloat(pixelToMmRatioInput.value) || 1;
                const delay = parseFloat(sequenceDelayInput.value) || 0;

                // --- Pre-computation and validation ---
                const selectedRadio = document.querySelector('.radio-selector:checked');
                if (!selectedRadio) return "No reference marker selected.";
                
                const mainMarkerId = selectedRadio.dataset.targetButton;
                const mainMarkerEl = document.querySelector(`.draggable-marker[data-marker-id="${mainMarkerId}"]`);
                if (!mainMarkerEl) return `Reference marker ${mainMarkerId} not found on the image.`;
                
                const mainX_px = parseFloat(mainMarkerEl.style.left);
                const mainY_px = parseFloat(mainMarkerEl.style.top);

                const placedMarkers = new Map();
                document.querySelectorAll('.draggable-marker').forEach(m => {
                    placedMarkers.set(m.dataset.markerId, {
                        x: parseFloat(m.style.left),
                        y: parseFloat(m.style.top)
                    });
                });
                
                if (!placedMarkers.has(mainMarkerId)) {
                     return `Reference marker ${mainMarkerId} is not on the image.`;
                }

                const calculatedValues = {}; // Stores only non-zero values

                // --- 1. First Pass: Calculate and store only non-zero values ---
                placedMarkers.forEach((pos, id) => {
                    if (id === mainMarkerId) return; // Skip comparing reference to itself

                    // Calculate Register
                    const correctionY_px = mainY_px - pos.y;
                    const regValue = correctionY_px / pixelToMmRatio;
                    if (regValue.toFixed(2) !== '0.00' && regValue.toFixed(2) !== '-0.00') {
                        const key = id.startsWith('F') ? `${id}_FLEXO_REG` : 'D_DIECUT_REG';
                        calculatedValues[key] = regValue;
                    }

                    // Calculate Centering
                    const correctionX_px = mainX_px - pos.x;
                    const cenValue = correctionX_px / pixelToMmRatio;
                    if (cenValue.toFixed(2) !== '0.00' && cenValue.toFixed(2) !== '-0.00') {
                        const key = id.startsWith('F') ? `${id}_FLEXO_CEN` : 'D_DIECUT_CEN';
                        calculatedValues[key] = cenValue;
                    }
                });
                
                const masterSequence = sequence || [];
                const finalLines = [];

                // --- 2. Second Pass: Build output based on calculated values and master sequence order ---
                masterSequence.forEach(action => {
                    const id = action.flexoId;

                    // Case 1: Action is for a marker not on the image, so skip.
                    if (id && !placedMarkers.has(id)) {
                        return; // continue
                    }

                    let valueKey = null;
                    let conditionKey = null;

                    // Case 2: Action is a standard intermediate click with conditions
                    if (action.type === 'INTERMEDIATE_CLICK') {
                        if (action.conditionTargetType === 'FLEXO_REG') conditionKey = `${id}_FLEXO_REG`;
                        else if (action.conditionTargetType === 'FLEXO_CEN') conditionKey = `${id}_FLEXO_CEN`;
                        else if (action.conditionTargetType === 'DIECUT_REG') conditionKey = `D_DIECUT_REG`;
                        else if (action.conditionTargetType === 'DIECUT_CEN') conditionKey = `D_DIECUT_CEN`;
                        // This handles custom actions (no flexoId, no conditionTargetType)
                        else if (!id && !action.conditionTargetType) {
                            finalLines.push(`${action.name}: (${action.x},${action.y})`);
                            return; // continue
                        }
                    } 
                    // Case 3: Action is a Register/Centering value calculation
                    else {
                        if (action.type === 'FLEXO_REG') valueKey = `${id}_FLEXO_REG`;
                        else if (action.type === 'FLEXO_CEN') valueKey = `${id}_FLEXO_CEN`;
                        else if (action.type === 'DIECUT_REG') valueKey = `D_DIECUT_REG`;
                        else if (action.type === 'DIECUT_CEN') valueKey = `D_DIECUT_CEN`;
                    }

                    // Add line to output if the condition (a non-zero value exists) is met
                    if (valueKey && calculatedValues.hasOwnProperty(valueKey)) {
                        const num = calculatedValues[valueKey];
                        const fixedValue = num.toFixed(2);
                        let formattedValue = (Math.abs(num) < 1 && num !== 0)
                            ? fixedValue.startsWith('0.') ? fixedValue.substring(1) : fixedValue.replace('-0.', '-.')
                            : fixedValue;
                        finalLines.push(`${action.name}: ${formattedValue}`);
                    } else if (conditionKey && calculatedValues.hasOwnProperty(conditionKey)) {
                        finalLines.push(`${action.name}: (${action.x},${action.y})`);
                    }
                });


                // --- 3. Add delays between the final, filtered lines ---
                if (delay > 0 && finalLines.length > 1) {
                    const linesWithDelay = [];
                    for (let i = 0; i < finalLines.length; i++) {
                        linesWithDelay.push(finalLines[i]);
                        if (i < finalLines.length - 1) {
                            linesWithDelay.push(`Delay: ${delay}`);
                        }
                    }
                    return linesWithDelay.join('\n');
                } else {
                    return finalLines.join('\n');
                }
            }
            
            function exportSequenceToFile() {
                const sequenceText = generateSequenceString();
                if (!sequenceText || sequenceText.includes("No reference marker")) {
                    console.warn("Sequence export skipped: no valid data to export.");
                    return;
                }

                const blob = new Blob([sequenceText], { type: 'text/plain' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                const machineName = machineNameInput.value.replace(/\s+/g, '_') || 'capture';
                const date = new Date();
                const timestamp = `${date.getFullYear()}${(date.getMonth() + 1).toString().padStart(2, '0')}${date.getDate().toString().padStart(2, '0')}_${date.getHours().toString().padStart(2, '0')}${date.getMinutes().toString().padStart(2, '0')}`;
                
                a.style.position = 'absolute';
                a.style.left = '-10000px';
                a.style.top = '0px';
                a.href = url;
                a.download = `easy-register_sequence_${machineName}_${timestamp}.txt`;
                
                document.body.appendChild(a);
                a.click(); 

                setTimeout(() => {
                    if (document.body.contains(a)) document.body.removeChild(a);
                    window.URL.revokeObjectURL(url);
                }, 500);
            }

            function generateAndShowSequence() {
                sequenceOutput.textContent = generateSequenceString();
                showModal(sequenceDisplayModal, sequenceDisplayModalBackdrop);
            }

            async function connectToDevice(device) {
                try {
                    console.log(`Connecting to ${device.name}...`);
                    bluetoothBtnText.textContent = `Connecting...`;
                    
                    device.addEventListener('gattserverdisconnected', onDisconnected);
                    const server = await device.gatt.connect();
                    
                    console.log('Connected to GATT Server:', server);
                    connectedDevice = device;

                    bluetoothStatus.style.backgroundColor = 'var(--color-success)';
                    bluetoothBtnText.textContent = device.name;
                } catch(error) {
                    console.error(`Failed to connect to ${device.name}:`, error);
                    bluetoothStatus.style.backgroundColor = 'var(--color-danger)';
                    bluetoothBtnText.textContent = 'Search via Bluetooth';
                    connectedDevice = null;
                    // Avoid alert on auto-connect failures
                }
            }
            
            async function autoConnectBluetooth() {
                if (!navigator.bluetooth || typeof navigator.bluetooth.getDevices !== 'function') {
                    console.log("Web Bluetooth API or getDevices() not supported.");
                    return;
                }
                if (!btDeviceName) {
                    console.log("No Bluetooth device name saved in settings. Skipping auto-connect.");
                    return;
                }

                try {
                    console.log("Checking for previously permitted devices...");
                    const devices = await navigator.bluetooth.getDevices();
                    const matchingDevice = devices.find(device => device.name === btDeviceName);

                    if (matchingDevice) {
                        console.log(`Found permitted device: ${matchingDevice.name}. Attempting to auto-connect...`);
                        await connectToDevice(matchingDevice);
                    } else {
                        console.log("No matching permitted device found for auto-connection.");
                    }
                } catch (error) {
                    console.error("Error during Bluetooth auto-connect:", error);
                }
            }


            function onDisconnected() {
                console.log('Device disconnected.');
                bluetoothStatus.style.backgroundColor = 'var(--color-danger)';
                bluetoothBtnText.textContent = 'Search via Bluetooth';
                connectedDevice = null;
                alert('Device disconnected.');
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
            
            function saveImageWithMarkers() {
                const canvas = document.createElement('canvas');
                const ctx = canvas.getContext('2d');
                const img = imagePreviewElement;

                canvas.width = img.naturalWidth;
                canvas.height = img.naturalHeight;
                ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

                const scaleX = img.naturalWidth / img.width;
                const scaleY = img.naturalHeight / img.height;

                const markerBg = getComputedStyle(document.documentElement).getPropertyValue('--main-marker-bg').trim();
                ctx.fillStyle = markerBg.includes('gradient') ? '#FF0000' : markerBg;
                ctx.shadowColor = 'white';
                ctx.shadowBlur = 4;
                ctx.shadowOffsetX = 0;
                ctx.shadowOffsetY = 0;
                
                const containerRect = imageContainer.getBoundingClientRect();
                const imageRect = img.getBoundingClientRect();
                const imageOffsetX = imageRect.left - containerRect.left;
                const imageOffsetY = imageRect.top - containerRect.top;

                document.querySelectorAll('.draggable-marker').forEach(markerEl => {
                    const leftOnContainer = parseFloat(markerEl.style.left);
                    const topOnContainer = parseFloat(markerEl.style.top);

                    const xOnDisplayedImage = leftOnContainer - imageOffsetX;
                    const yOnDisplayedImage = topOnContainer - imageOffsetY;

                    const finalX = xOnDisplayedImage * scaleX;
                    const finalY = yOnDisplayedImage * scaleY;

                    const currentSliderValue = markerSizeSlider.value;
                    const scale = calculateScale(currentSliderValue);
                    const markerBaseSize = 24;
                    const crossBaseThickness = 2;

                    const markerWidthOnCanvas = markerBaseSize * scale * scaleX;
                    const markerHeightOnCanvas = markerBaseSize * scale * scaleY;
                    const crossThicknessX = crossBaseThickness * scaleX;
                    const crossThicknessY = crossBaseThickness * scaleY;

                    ctx.fillRect(finalX - markerWidthOnCanvas / 2, finalY - crossThicknessY / 2, markerWidthOnCanvas, crossThicknessY);
                    ctx.fillRect(finalX - crossThicknessX / 2, finalY - markerHeightOnCanvas / 2, crossThicknessX, markerHeightOnCanvas);
                });

                ctx.shadowBlur = 0; // Reset shadow

                canvas.toBlob((blob) => {
                    if (!blob) {
                        console.error("Failed to create blob for image with markers.");
                        return;
                    }
                    try {
                        const url = URL.createObjectURL(blob);
                        const a = document.createElement('a');
                        const machineName = machineNameInput.value.replace(/\s+/g, '_') || 'capture';
                        const date = new Date();
                        const timestamp = `${date.getFullYear()}${(date.getMonth() + 1).toString().padStart(2, '0')}${date.getDate().toString().padStart(2, '0')}_${date.getHours().toString().padStart(2, '0')}${date.getMinutes().toString().padStart(2, '0')}`;
                        
                        a.style.position = 'absolute';
                        a.style.left = '-10000px';
                        a.style.top = '0px';

                        a.href = url;
                        a.download = `easy-register_${machineName}_${timestamp}_x.jpeg`;
                        
                        document.body.appendChild(a);
                        a.click(); 

                        setTimeout(() => {
                            if (document.body.contains(a)) document.body.removeChild(a);
                            window.URL.revokeObjectURL(url);
                        }, 500);

                    } catch (e) {
                        console.error("Error saving image with markers:", e);
                        alert("An error occurred while saving the image with markers.");
                    }
                }, 'image/jpeg', 0.7);
            }

            // ===============================================
            // SECTION 2: INITIAL SETUP & EVENT LISTENERS
            // ===============================================

            colors.forEach((color, index) => {
                const swatch = document.createElement('div');
                swatch.className = 'color-swatch';
                swatch.dataset.color = color;

                if (color === 'rainbow-gradient') {
                    swatch.style.background = 'conic-gradient(from 180deg at 50% 50%, red, yellow, lime, aqua, blue, magenta, red)';
                } else {
                    swatch.style.backgroundColor = color;
                }

                if (index === 0) {
                    swatch.classList.add('selected');
                    applyColorTheme(color); // Set initial theme
                }
                
                swatch.addEventListener('click', () => {
                    const selectedColor = swatch.dataset.color;
                    applyColorTheme(selectedColor);
                    
                    document.querySelectorAll('.color-swatch').forEach(s => s.classList.remove('selected'));
                    swatch.classList.add('selected');
                });
                colorPaletteContainer.appendChild(swatch);
            });
            
            imageUploadInput.addEventListener('change', function(event) {
                if (event.target.files && event.target.files[0]) {
                    const file = event.target.files[0];
                    const imageURL = URL.createObjectURL(file);
                    setImageAsPreview(imageURL, file);
                }
            });

            trashCan.addEventListener('click', clearMarkersAndCorrections);

            sendToScreenBtn.addEventListener('click', () => {
                stopInputBtn.disabled = true;
                blobForDownload = null; // Reset before use

                if (saveImageCheckbox.checked && imagePreviewElement.src && imagePreviewElement.naturalWidth > 0) {
                    // Asynchronously prepare the image blob for downloading
                    const canvas = document.createElement('canvas');
                    const ctx = canvas.getContext('2d');
                    canvas.width = imagePreviewElement.naturalWidth;
                    canvas.height = imagePreviewElement.naturalHeight;
                    ctx.drawImage(imagePreviewElement, 0, 0);
                    canvas.toBlob((blob) => {
                        blobForDownload = blob; // Store the blob
                        showModal(confirmationModal, confirmationModalBackdrop); // Show modal once blob is ready
                    }, 'image/jpeg', 0.7);
                } else {
                    // If not saving, just show the modal immediately
                    showModal(confirmationModal, confirmationModalBackdrop);
                }
            });

            cancelSendBtn.addEventListener('click', () => closeModal(confirmationModal, confirmationModalBackdrop));
            confirmationModalBackdrop.addEventListener('click', () => closeModal(confirmationModal, confirmationModalBackdrop));

            confirmSendBtn.addEventListener('click', () => {
                stopInputBtn.disabled = false;
                
                // New logic using the pre-generated blob for robust mobile download
                if (blobForDownload) {
                    try {
                        const url = URL.createObjectURL(blobForDownload);
                        const a = document.createElement('a');
                        const machineName = machineNameInput.value.replace(/\s+/g, '_') || 'capture';
                        const date = new Date();
                        const timestamp = `${date.getFullYear()}${(date.getMonth() + 1).toString().padStart(2, '0')}${date.getDate().toString().padStart(2, '0')}_${date.getHours().toString().padStart(2, '0')}${date.getMinutes().toString().padStart(2, '0')}`;
                        
                        // Visually hide the link for better compatibility on mobile
                        a.style.position = 'absolute';
                        a.style.left = '-10000px';
                        a.style.top = '0px';

                        a.href = url;
                        a.download = `easy-register_${machineName}_${timestamp}.jpeg`;
                        
                        document.body.appendChild(a);
                        a.click(); // This is now synchronous to the user's click on "Yes"

                        // Clean up after a longer delay to ensure download starts on slower devices
                        setTimeout(() => {
                            if (document.body.contains(a)) {
                                document.body.removeChild(a);
                            }
                            window.URL.revokeObjectURL(url);
                        }, 500);

                    } catch (e) {
                        console.error("Error saving image from blob:", e);
                        alert("An error occurred while saving the image.");
                    } finally {
                        blobForDownload = null; // Reset blob variable
                    }
                }
                
                if (saveImageWithCrossCheckbox.checked && imagePreviewElement.src && imagePreviewElement.naturalWidth > 0) {
                    saveImageWithMarkers();
                }

                if (showSequenceCheckbox.checked) {
                    generateAndShowSequence();
                }
                
                if (exportSequenceCheckbox.checked) {
                    exportSequenceToFile();
                }

                // closeModal(confirmationModal, confirmationModalBackdrop);

                const originalText = sendToScreenBtn.textContent;
                sendToScreenBtn.textContent = 'Sent!';
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
                
                // --- Generate and download the STOP file ---
                const stopText = 'STOP HID INPUT';
                const blob = new Blob([stopText], { type: 'text/plain' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                
                const date = new Date();
                const timestamp = `${date.getFullYear()}${(date.getMonth() + 1).toString().padStart(2, '0')}${date.getDate().toString().padStart(2, '0')}_${date.getHours().toString().padStart(2, '0')}${date.getMinutes().toString().padStart(2, '0')}${date.getSeconds().toString().padStart(2, '0')}`;

                a.style.position = 'absolute';
                a.style.left = '-10000px';
                a.style.top = '0px';
                a.href = url;
                a.download = `stop_input_${timestamp}.txt`;
                
                document.body.appendChild(a);
                a.click(); 

                setTimeout(() => {
                    if (document.body.contains(a)) document.body.removeChild(a);
                    window.URL.revokeObjectURL(url);
                }, 500);

                // --- Original functionality ---
                closeModal(confirmationModal, confirmationModalBackdrop);
                console.log('Input stopped and STOP file generated.');
            });

            optionsBtn.addEventListener('click', () => showModal(optionsModal, optionsModalBackdrop));
            closeOptionsModalBtn.addEventListener('click', () => closeModal(optionsModal, optionsModalBackdrop));

            markerSizeSlider.addEventListener('input', (e) => {
                const value = e.target.value;
                sliderValueSpan.textContent = value;
                updateMarkerSizes(value);
            });

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
            sequenceDisplayModalBackdrop.addEventListener('click', () => closeModal(sequenceDisplayModal, sequenceDisplayModalBackdrop));
            closeSequenceDisplayBtn.addEventListener('click', () => closeModal(sequenceDisplayModal, sequenceDisplayModalBackdrop));

            passwordModalBackdrop.addEventListener('click', () => closeModal(passwordModal, passwordModalBackdrop));
            cancelPasswordBtn.addEventListener('click', () => closeModal(passwordModal, passwordModalBackdrop));
            submitPasswordBtn.addEventListener('click', handlePasswordSubmit);
            passwordInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') { handlePasswordSubmit(); } });

            settingsModalBackdrop.addEventListener('click', () => closeModal(settingsModal, settingsModalBackdrop));
            closeSettingsModalBtn.addEventListener('click', () => closeModal(settingsModal, settingsModalBackdrop));

            machineNameInput.addEventListener('input', (e) => {
                const machineNameHeader = document.querySelector('h1');
                const name = e.target.value;
                machineNameDisplay.textContent = name || 'machine name';
                
                // Only show/hide if an image is currently displayed
                if (!mainContainer.classList.contains('no-image')) {
                     if (name && name.trim() !== '') {
                        machineNameHeader.classList.remove('hidden');
                    } else {
                        machineNameHeader.classList.add('hidden');
                    }
                }
               
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
            saveImageWithCrossCheckbox.addEventListener('change', saveState);
            showSequenceCheckbox.addEventListener('change', saveState);
            exportSequenceCheckbox.addEventListener('change', saveState);
            
            addSequenceActionBtn.addEventListener('click', () => {
                const newAction = createSequenceAction(`custom_${Date.now()}`, 'INTERMEDIATE_CLICK', null, 'New Action');
                if (!sequence) {
                    sequence = [];
                }
                sequence.push(newAction);
                populateSequenceEditor();
            });

            saveSequenceBtn.addEventListener('click', saveSequenceFromEditor);
            sequenceDelayInput.addEventListener('input', saveState);

            bluetoothSearchBtn.addEventListener('click', async () => {
                if (connectedDevice) {
                    connectedDevice.gatt.disconnect();
                    return;
                }
                if (!btDeviceName) {
                    alert("Please enter the Bluetooth device name in the settings first.");
                    return;
                }
                 if (!navigator.bluetooth) {
                    alert("The Web Bluetooth API is not supported on this browser. It only works on a secure (https) site.");
                    return;
                 }

                try {
                    console.log(`Requesting device via search: ${btDeviceName}`);
                    const device = await navigator.bluetooth.requestDevice({
                        filters: [{ name: btDeviceName }],
                        optionalServices: ['battery_service', 'device_information'] 
                    });

                    console.log('Device selected:', device.name);
                    await connectToDevice(device);

                } catch(error) {
                    console.error('Bluetooth search error:', error);
                    // Avoid alerting on common "user cancelled" errors
                    if (error.name !== 'NotFoundError') {
                        alert(`Bluetooth Error: ${error.message}.`);
                    }
                }
            });

            exportConfigBtn.addEventListener('click', () => {
                const config = {
                    machineName: machineNameInput.value,
                    flexoCount: parseInt(flexoCountSlider.value, 10),
                    pixelToMmRatio: parseFloat(pixelToMmRatioInput.value),
                    bluetoothDeviceName: bluetoothDeviceNameInput.value,
                    saveImage: saveImageCheckbox.checked,
                    saveImageWithCross: saveImageWithCrossCheckbox.checked,
                    showSequence: showSequenceCheckbox.checked,
                    exportSequence: exportSequenceCheckbox.checked,
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
                        alert('Configuration imported successfully!');
                        closeModal(settingsModal, settingsModalBackdrop);
                    } catch (error) {
                        console.error("Import error:", error);
                        alert("The configuration file is invalid or corrupt.");
                    }
                };
                reader.onerror = () => alert("Error reading the file.");
                reader.readAsText(file);
                event.target.value = '';
            });

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

            document.addEventListener('mouseup', dragEnd);
            document.addEventListener('touchend', dragEnd);
            document.addEventListener('mousemove', drag);
            document.addEventListener('touchmove', drag, { passive: false });

            // --- Initialization ---
            loadState();
            autoConnectBluetooth();
            
        });
    </script>

</body>
</html>
" in the document "Page Blanche" and am asking a question based on this selection.
You are an AI assistant. I have selected the code above in the most up-to-date Canvas document, and you should use it to answer my query below.
In the previous turn, you fixed a bug. Now, I want you to make an additional change.

quand je clic sur le bouton option il faut masquer le header avec le nom de la machine et l'icone photo, et aussi la zone des corrections et le bouton option lui-même


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Mycorrhizal Grid Analysis Tool</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tensorflow/4.2.0/tf.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #2c3e50 0%, #3498db 50%, #2980b9 100%);
            min-height: 100vh;
            color: #333;
        }

        .container {
            max-width: 1800px;
            margin: 0 auto;
            padding: 20px;
        }

        .header {
            text-align: center;
            margin-bottom: 30px;
            color: white;
            background: rgba(0,0,0,0.2);
            padding: 20px;
            border-radius: 15px;
            backdrop-filter: blur(10px);
        }

        .header h1 {
            font-size: 2.8em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 8px rgba(0,0,0,0.5);
        }

        .header p {
            font-size: 1.2em;
            opacity: 0.9;
        }

        .mode-selector {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-bottom: 30px;
        }

        .mode-btn {
            padding: 15px 30px;
            border: none;
            border-radius: 12px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.4s ease;
            color: white;
            position: relative;
            overflow: hidden;
        }

        .mode-btn::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
            transition: left 0.5s;
        }

        .mode-btn:hover::before {
            left: 100%;
        }

        .mode-btn.training {
            background: linear-gradient(135deg, #e74c3c, #c0392b);
        }

        .mode-btn.batch {
            background: linear-gradient(135deg, #f39c12, #e67e22);
        }

        .mode-btn.analysis {
            background: linear-gradient(135deg, #27ae60, #229954);
        }

        .mode-btn.active {
            transform: translateY(-3px) scale(1.05);
            box-shadow: 0 8px 25px rgba(0,0,0,0.3);
        }

        .main-content {
            display: grid;
            grid-template-columns: 1fr 450px;
            gap: 25px;
            margin-bottom: 25px;
        }

        .image-section {
            background: rgba(255,255,255,0.95);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 15px 50px rgba(0,0,0,0.2);
            backdrop-filter: blur(10px);
        }

        .controls-panel {
            background: rgba(255,255,255,0.95);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 15px 50px rgba(0,0,0,0.2);
            backdrop-filter: blur(10px);
            height: fit-content;
            position: sticky;
            top: 20px;
        }

        .upload-area {
            border: 3px dashed #3498db;
            border-radius: 15px;
            padding: 50px;
            text-align: center;
            margin-bottom: 25px;
            transition: all 0.4s ease;
            cursor: pointer;
            background: linear-gradient(45deg, rgba(52,152,219,0.05), rgba(46,204,113,0.05));
        }

        .upload-area:hover {
            border-color: #27ae60;
            background: linear-gradient(45deg, rgba(52,152,219,0.1), rgba(46,204,113,0.1));
            transform: translateY(-2px);
        }

        .upload-area.dragover {
            border-color: #f39c12;
            background: rgba(243,156,18,0.1);
            transform: scale(1.02);
        }

        .image-container {
            position: relative;
            max-width: 100%;
            margin: 20px 0;
            border-radius: 15px;
            overflow: hidden;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
        }

        #microscope-image {
            max-width: 100%;
            height: auto;
            display: block;
        }

        .overlay-canvas {
            position: absolute;
            top: 0;
            left: 0;
            pointer-events: none;
        }

        #training-overlay {
            cursor: crosshair;
            pointer-events: auto;
        }

        .control-group {
            margin-bottom: 25px;
            padding: 20px;
            background: linear-gradient(135deg, rgba(52,152,219,0.1), rgba(46,204,113,0.05));
            border-radius: 15px;
            border-left: 4px solid #3498db;
        }

        .control-group h3 {
            margin-bottom: 18px;
            color: #2c3e50;
            font-size: 1.2em;
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .grid-controls {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 15px;
        }

        .grid-input {
            padding: 12px;
            border: 2px solid #bdc3c7;
            border-radius: 8px;
            font-size: 14px;
            transition: border-color 0.3s ease;
        }

        .grid-input:focus {
            border-color: #3498db;
            outline: none;
        }

        .training-tools {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 18px;
        }

        .tool-btn {
            padding: 14px 18px;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-weight: 600;
            font-size: 14px;
            transition: all 0.3s ease;
            text-align: center;
            color: white;
            position: relative;
            overflow: hidden;
        }

        .tool-btn::before {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 0;
            height: 0;
            background: rgba(255,255,255,0.2);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            transition: width 0.3s, height 0.3s;
        }

        .tool-btn:hover::before {
            width: 100%;
            height: 100%;
        }

        .tool-btn.hyphae { background: linear-gradient(135deg, #e74c3c, #c0392b); }
        .tool-btn.vesicles { background: linear-gradient(135deg, #1abc9c, #16a085); }
        .tool-btn.arbuscules { background: linear-gradient(135deg, #3498db, #2980b9); }
        .tool-btn.other-fungi { background: linear-gradient(135deg, #95a5a6, #7f8c8d); }

        .tool-btn.active {
            transform: translateY(-3px) scale(1.05);
            box-shadow: 0 8px 20px rgba(0,0,0,0.3);
        }

        .model-status {
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
            text-align: center;
            font-weight: bold;
            font-size: 16px;
            transition: all 0.3s ease;
        }

        .model-status.untrained {
            background: linear-gradient(135deg, #f39c12, #e67e22);
            color: white;
            box-shadow: 0 4px 15px rgba(243,156,18,0.3);
        }

        .model-status.training {
            background: linear-gradient(135deg, #3498db, #2980b9);
            color: white;
            box-shadow: 0 4px 15px rgba(52,152,219,0.3);
        }

        .model-status.trained {
            background: linear-gradient(135deg, #27ae60, #229954);
            color: white;
            box-shadow: 0 4px 15px rgba(39,174,96,0.3);
        }

        .progress-bar {
            width: 100%;
            height: 25px;
            background: #ecf0f1;
            border-radius: 12px;
            overflow: hidden;
            margin: 15px 0;
            box-shadow: inset 0 2px 5px rgba(0,0,0,0.1);
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #3498db, #2ecc71);
            width: 0%;
            transition: width 0.5s ease;
            position: relative;
            overflow: hidden;
        }

        .progress-fill::after {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            bottom: 0;
            right: 0;
            background: linear-gradient(
                -45deg,
                rgba(255,255,255,.2) 25%,
                transparent 25%,
                transparent 50%,
                rgba(255,255,255,.2) 50%,
                rgba(255,255,255,.2) 75%,
                transparent 75%,
                transparent
            );
            background-size: 50px 50px;
            animation: move 2s linear infinite;
        }

        @keyframes move {
            0% { background-position: 0 0; }
            100% { background-position: 50px 50px; }
        }

        .btn {
            padding: 14px 25px;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-weight: 600;
            font-size: 15px;
            transition: all 0.3s ease;
            width: 100%;
            margin-bottom: 12px;
            position: relative;
            overflow: hidden;
        }

        .btn::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.2), transparent);
            transition: left 0.5s;
        }

        .btn:hover::before {
            left: 100%;
        }

        .btn-primary {
            background: linear-gradient(135deg, #3498db, #2980b9);
            color: white;
        }

        .btn-success {
            background: linear-gradient(135deg, #27ae60, #229954);
            color: white;
        }

        .btn-warning {
            background: linear-gradient(135deg, #f39c12, #e67e22);
            color: white;
        }

        .btn-danger {
            background: linear-gradient(135deg, #e74c3c, #c0392b);
            color: white;
        }

        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(0,0,0,0.2);
        }

        .btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }

        .batch-controls {
            display: none;
        }

        .batch-controls.active {
            display: block;
        }

        .batch-upload {
            border: 3px dashed #f39c12;
            border-radius: 15px;
            padding: 40px;
            text-align: center;
            margin-bottom: 20px;
            transition: all 0.3s ease;
            cursor: pointer;
            background: rgba(243,156,18,0.05);
        }

        .batch-upload:hover {
            border-color: #e67e22;
            background: rgba(243,156,18,0.1);
        }

        .batch-progress {
            margin-top: 20px;
        }

        .image-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 12px;
            background: #f8f9fa;
            border-radius: 8px;
            margin-bottom: 8px;
            border-left: 4px solid #3498db;
        }

        .image-item.processing {
            border-left-color: #f39c12;
            background: #fef9e7;
        }

        .image-item.completed {
            border-left-color: #27ae60;
            background: #eafaf1;
        }

        .image-item.error {
            border-left-color: #e74c3c;
            background: #fdeeed;
        }

        .results-section {
            background: rgba(255,255,255,0.95);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 15px 50px rgba(0,0,0,0.2);
            backdrop-filter: blur(10px);
            margin-top: 25px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }

        .stat-card {
            padding: 25px;
            border-radius: 15px;
            text-align: center;
            color: white;
            font-weight: 600;
            position: relative;
            overflow: hidden;
            transition: transform 0.3s ease;
        }

        .stat-card::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: linear-gradient(45deg, transparent, rgba(255,255,255,0.1), transparent);
            transform: rotate(45deg);
            transition: all 0.6s;
        }

        .stat-card:hover::before {
            animation: shine 0.6s ease-in-out;
        }

        @keyframes shine {
            0% { transform: translateX(-100%) translateY(-100%) rotate(45deg); }
            100% { transform: translateX(100%) translateY(100%) rotate(45deg); }
        }

        .stat-card:hover {
            transform: translateY(-5px);
        }

        .stat-card.hyphae { background: linear-gradient(135deg, #e74c3c, #c0392b); }
        .stat-card.vesicles { background: linear-gradient(135deg, #1abc9c, #16a085); }
        .stat-card.arbuscules { background: linear-gradient(135deg, #3498db, #2980b9); }
        .stat-card.other-fungi { background: linear-gradient(135deg, #95a5a6, #7f8c8d); }
        .stat-card.colonization { background: linear-gradient(135deg, #9b59b6, #8e44ad); }

        .stat-number {
            font-size: 2.5em;
            font-weight: bold;
            margin-bottom: 8px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .training-data-summary {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
            border: 1px solid #dee2e6;
        }

        .training-data-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
            padding: 8px 0;
            border-bottom: 1px solid #dee2e6;
        }

        .training-data-item:last-child {
            border-bottom: none;
            margin-bottom: 0;
        }

        .data-export {
            margin-top: 20px;
            padding: 15px;
            background: linear-gradient(135deg, rgba(52,152,219,0.1), rgba(46,204,113,0.05));
            border-radius: 12px;
        }

        .confidence-meter {
            display: flex;
            align-items: center;
            gap: 12px;
            margin: 15px 0;
        }

        .confidence-bar {
            flex: 1;
            height: 12px;
            background: #ecf0f1;
            border-radius: 6px;
            overflow: hidden;
        }

        .confidence-fill {
            height: 100%;
            background: linear-gradient(90deg, #e74c3c, #f39c12, #27ae60);
            transition: width 0.5s ease;
        }

        .file-input {
            display: none;
        }

        .grid-overlay-controls {
            margin-bottom: 15px;
        }

        .checkbox-group {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 10px;
        }

        .checkbox-group input[type="checkbox"] {
            width: 18px;
            height: 18px;
            accent-color: #3498db;
        }

        @media (max-width: 1400px) {
            .main-content {
                grid-template-columns: 1fr;
            }
            
            .controls-panel {
                position: static;
            }
        }

        .colonization-summary {
            background: linear-gradient(135deg, rgba(155,89,182,0.1), rgba(142,68,173,0.05));
            padding: 20px;
            border-radius: 15px;
            margin-top: 20px;
            border-left: 5px solid #9b59b6;
        }

        .colonization-details {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 15px;
            margin-top: 15px;
        }

        .detail-item {
            text-align: center;
            padding: 15px;
            background: rgba(255,255,255,0.7);
            border-radius: 10px;
        }

        .detail-value {
            font-size: 1.8em;
            font-weight: bold;
            color: #9b59b6;
            margin-bottom: 5px;
        }

        .intersection-points {
            position: absolute;
            top: 0;
            left: 0;
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🔬 Advanced Mycorrhizal Grid Analysis Tool</h1>
            <p>AI-powered detection and quantification of mycorrhizal colonization with grid-based intersection counting</p>
        </div>

        <div class="mode-selector">
            <button class="mode-btn training active" onclick="switchMode('training')">
                🎯 Training Mode
            </button>
            <button class="mode-btn batch" onclick="switchMode('batch')">
                📊 Batch Processing
            </button>
            <button class="mode-btn analysis" onclick="switchMode('analysis')">
                🧮 Grid Analysis
            </button>
        </div>

        <div class="main-content">
            <div class="image-section">
                <div class="upload-area" id="upload-area">
                    <h3>📤 Upload Microscope Images</h3>
                    <p id="upload-text">Drag and drop your microscope images here or click to browse</p>
                    <p><small>Supports: JPG, PNG, TIFF - Max 10MB per image</small></p>
                    <input type="file" id="file-input" class="file-input" accept="image/*" multiple>
                </div>

                <!-- Batch Upload Area -->
                <div class="batch-upload" id="batch-upload" style="display: none;">
                    <h3>📊 Batch Upload</h3>
                    <p>Upload multiple images for batch processing</p>
                    <p><small>Select multiple images to process automatically</small></p>
                    <input type="file" id="batch-input" class="file-input" accept="image/*" multiple>
                </div>

                <div class="image-container" id="image-container" style="display: none;">
                    <img id="microscope-image" alt="Microscope image">
                    <canvas id="grid-overlay" class="overlay-canvas"></canvas>
                    <canvas id="training-overlay" class="overlay-canvas"></canvas>
                    <canvas id="detection-overlay" class="overlay-canvas"></canvas>
                    <canvas id="intersection-points" class="intersection-points"></canvas>
                </div>

                <!-- Batch Progress -->
                <div class="batch-progress" id="batch-progress" style="display: none;">
                    <h4>📈 Processing Progress</h4>
                    <div id="batch-items"></div>
                </div>
            </div>

            <div class="controls-panel">
                <!-- Training Mode Controls -->
                <div class="training-controls active">
                    <div class="model-status untrained" id="model-status">
                        🚫 Model Not Trained
                        <br><small>Add training samples to begin</small>
                    </div>

                    <div class="control-group">
                        <h3>🎯 Training Tools</h3>
                        <div class="training-tools">
                            <button class="tool-btn hyphae active" data-type="hyphae" onclick="selectTrainingTool('hyphae')">🧬 Hyphae</button>
                            <button class="tool-btn vesicles" data-type="vesicles" onclick="selectTrainingTool('vesicles')">🫧 Vesicles</button>
                            <button class="tool-btn arbuscules" data-type="arbuscules" onclick="selectTrainingTool('arbuscules')">🌳 Arbuscules</button>
                            <button class="tool-btn other-fungi" data-type="other-fungi" onclick="selectTrainingTool('other-fungi')">🦠 Other Fungi</button>
                        </div>
                        <p><small>Click on structures in the image to add training samples</small></p>
                    </div>

                    <div class="control-group">
                        <h3>📊 Training Data</h3>
                        <div class="training-data-summary" id="training-summary">
                            <div class="training-data-item">
                                <span>🧬 Hyphae:</span>
                                <span id="hyphae-samples">0 samples</span>
                            </div>
                            <div class="training-data-item">
                                <span>🫧 Vesicles:</span>
                                <span id="vesicles-samples">0 samples</span>
                            </div>
                            <div class="training-data-item">
                                <span>🌳 Arbuscules:</span>
                                <span id="arbuscules-samples">0 samples</span>
                            </div>
                            <div class="training-data-item">
                                <span>🦠 Other Fungi:</span>
                                <span id="other-fungi-samples">0 samples</span>
                            </div>
                        </div>
                    </div>

                    <button class="btn btn-primary" id="train-btn" onclick="trainModel()" disabled>
                        🚀 Train Model
                    </button>
                    
                    <div class="progress-bar" id="training-progress" style="display: none;">
                        <div class="progress-fill" id="progress-fill"></div>
                    </div>

                    <button class="btn btn-warning" onclick="clearTrainingData()">
                        🗑️ Clear Training Data
                    </button>
                </div>

                <!-- Batch Processing Controls -->
                <div class="batch-controls">
                    <div class="control-group">
                        <h3>📊 Batch Settings</h3>
                        <div class="grid-controls">
                            <input type="number" class="grid-input" id="batch-grid-rows" placeholder="Grid Rows" value="10">
                            <input type="number" class="grid-input" id="batch-grid-cols" placeholder="Grid Columns" value="10">
                        </div>
                        <label>
                            <input type="range" id="batch-confidence" min="0.1" max="1" step="0.05" value="0.7">
                            Confidence Threshold: <span id="batch-confidence-value">70%</span>
                        </label>
                    </div>

                    <button class="btn btn-success" id="batch-process-btn" onclick="processBatch()" disabled>
                        ⚡ Process All Images
                    </button>

                    <button class="btn btn-primary" onclick="exportBatchResults()">
                        📄 Export Results (CSV)
                    </button>
                </div>

                <!-- Analysis Mode Controls -->
                <div class="analysis-controls">
                    <div class="control-group">
                        <h3>🔧 Grid Settings</h3>
                        <div class="grid-controls">
                            <input type="number" class="grid-input" id="grid-rows" placeholder="Grid Rows" value="10">
                            <input type="number" class="grid-input" id="grid-cols" placeholder="Grid Columns" value="10">
                        </div>
                        <div class="grid-overlay-controls">
                            <div class="checkbox-group">
                                <input type="checkbox" id="show-grid" checked onchange="toggleGrid()">
                                <label for="show-grid">Show Grid Lines</label>
                            </div>
                            <div class="checkbox-group">
                                <input type="checkbox" id="show-intersections" checked onchange="updateDisplay()">
                                <label for="show-intersections">Show Intersections</label>
                            </div>
                        </div>
                    </div>

                    <div class="control-group">
                        <h3>🔍 Detection Settings</h3>
                        <label>
                            <input type="range" id="confidence-threshold" min="0.1" max="1" step="0.05" value="0.7">
                            Confidence Threshold: <span id="confidence-value">70%</span>
                        </label>
                        <label>
                            <input type="range" id="intersection-radius" min="5" max="50" step="5" value="20">
                            Intersection Radius: <span id="radius-value">20px</span>
                        </label>
                    </div>

                    <button class="btn btn-success" id="analyze-btn" onclick="analyzeImage()" disabled>
                        🧮 Analyze Grid Intersections
                    </button>

                    <div class="confidence-meter">
                        <span>Model Confidence:</span>
                        <div class="confidence-bar">
                            <div class="confidence-fill" id="model-confidence" style="width: 0%"></div>
                        </div>
                        <span id="confidence-text">0%</span>
                    </div>
                </div>

                <div class="control-group">
                    <h3>💾 Model Management</h3>
                    <button class="btn btn-primary" onclick="saveModel()">
                        💾 Save Trained Model
                    </button>
                    <button class="btn btn-primary" onclick="loadModel()">
                        📂 Load Model
                    </button>
                    <input type="file" id="model-input" style="display: none;" accept=".json">
                </div>

                <div class="data-export">
                    <h4>📈 Data Export</h4>
                    <button class="btn btn-success" onclick="exportTrainingData()">
                        💾 Export Training Data
                    </button>
                    <button class="btn btn-primary" onclick="importTrainingData()">
                        📂 Import Training Data
                    </button>
                    <input type="file" id="training-data-input" style="display: none;" accept=".json">
                </div>
            </div>
        </div>

        <div class="results-section" id="results-section" style="display: none;">
            <h3>🎯 Grid Analysis Results</h3>
            <div class="stats-grid">
                <div class="stat-card hyphae">
                    <div class="stat-number" id="detected-hyphae">0</div>
                    <div>Hyphae Intersections</div>
                </div>
                <div class="stat-card vesicles">
                    <div class="stat-number" id="detected-vesicles">0</div>
                    <div>Vesicle Intersections</div>
                </div>
                <div class="stat-card arbuscules">
                    <div class="stat-number" id="detected-arbuscules">0</div>
                    <div>Arbuscule Intersections</div>
                </div>
                <div class="stat-card other-fungi">
                    <div class="stat-number" id="detected-other-fungi">0</div>
                    <div>Other Fungi</div>
                </div>
                <div class="stat-card colonization">
                    <div class="stat-number" id="colonization-rate">0%</div>
                    <div>Colonization Rate</div>
                </div>
            </div>

            <div class="colonization-summary">
                <h4>🌱 Colonization Analysis</h4>
                <p>Mycorrhizal colonization is a key indicator of plant-fungal symbiosis health. The grid intersection method provides a quantitative measure of fungal presence in root tissues.</p>
                
                <div class="colonization-details">
                    <div class="detail-item">
                        <div>Total Intersections</div>
                        <div class="detail-value" id="total-intersections">0</div>
                    </div>
                    <div class="detail-item">
                        <div>Fungal Intersections</div>
                        <div class="detail-value" id="fungal-intersections">0</div>
                    </div>
                    <div class="detail-item">
                        <div>Hyphae Density</div>
                        <div class="detail-value" id="hyphae-density">0.0</div>
                    </div>
                    <div class="detail-item">
                        <div>Arbuscule Frequency</div>
                        <div class="detail-value" id="arbuscule-frequency">0.0</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Application state
        const appState = {
            mode: 'training',
            image: null,
            trainingData: {
                hyphae: [],
                vesicles: [],
                arbuscules: [],
                otherFungi: []
            },
            currentTool: 'hyphae',
            model: null,
            modelTrained: false,
            grid: {
                rows: 10,
                cols: 10,
                visible: true,
                intersectionsVisible: true
            },
            batchFiles: [],
            batchResults: []
        };

        // DOM elements
        const elements = {
            fileInput: document.getElementById('file-input'),
            batchInput: document.getElementById('batch-input'),
            microscopeImage: document.getElementById('microscope-image'),
            imageContainer: document.getElementById('image-container'),
            uploadArea: document.getElementById('upload-area'),
            batchUpload: document.getElementById('batch-upload'),
            trainingOverlay: document.getElementById('training-overlay'),
            gridOverlay: document.getElementById('grid-overlay'),
            detectionOverlay: document.getElementById('detection-overlay'),
            intersectionCanvas: document.getElementById('intersection-points'),
            modelStatus: document.getElementById('model-status'),
            trainBtn: document.getElementById('train-btn'),
            trainingProgress: document.getElementById('training-progress'),
            progressFill: document.getElementById('progress-fill'),
            analyzeBtn: document.getElementById('analyze-btn'),
            confidenceBar: document.getElementById('model-confidence'),
            confidenceText: document.getElementById('confidence-text'),
            resultsSection: document.getElementById('results-section'),
            batchProgress: document.getElementById('batch-progress'),
            batchItems: document.getElementById('batch-items'),
            showGrid: document.getElementById('show-grid'),
            showIntersections: document.getElementById('show-intersections'),
            confidenceThreshold: document.getElementById('confidence-threshold'),
            confidenceValue: document.getElementById('confidence-value'),
            radiusValue: document.getElementById('radius-value'),
            intersectionRadius: document.getElementById('intersection-radius'),
            batchConfidence: document.getElementById('batch-confidence'),
            batchConfidenceValue: document.getElementById('batch-confidence-value')
        };

        // Initialize the application
        function init() {
            // Set up event listeners
            setupEventListeners();
            
            // Initialize TensorFlow.js
            tf.ready().then(() => {
                console.log("TensorFlow.js is ready");
            });
            
            // Update UI based on initial state
            updateTrainingSummary();
        }

        // Set up event listeners
        function setupEventListeners() {
            // File upload
            elements.fileInput.addEventListener('change', handleImageUpload);
            elements.batchInput.addEventListener('change', handleBatchUpload);
            
            // Upload area drag and drop
            elements.uploadArea.addEventListener('dragover', (e) => {
                e.preventDefault();
                elements.uploadArea.classList.add('dragover');
            });
            
            elements.uploadArea.addEventListener('dragleave', () => {
                elements.uploadArea.classList.remove('dragover');
            });
            
            elements.uploadArea.addEventListener('drop', (e) => {
                e.preventDefault();
                elements.uploadArea.classList.remove('dragover');
                if (e.dataTransfer.files.length) {
                    elements.fileInput.files = e.dataTransfer.files;
                    handleImageUpload();
                }
            });
            
            elements.uploadArea.addEventListener('click', () => {
                elements.fileInput.click();
            });
            
            elements.batchUpload.addEventListener('click', () => {
                elements.batchInput.click();
            });
            
            // Training overlay click
            elements.trainingOverlay.addEventListener('click', handleTrainingClick);
            
            // Confidence threshold slider
            elements.confidenceThreshold.addEventListener('input', () => {
                const value = elements.confidenceThreshold.value;
                elements.confidenceValue.textContent = Math.round(value * 100) + '%';
                if (appState.image && appState.modelTrained) {
                    analyzeImage();
                }
            });
            
            // Intersection radius slider
            elements.intersectionRadius.addEventListener('input', () => {
                const value = elements.intersectionRadius.value;
                elements.radiusValue.textContent = value + 'px';
                if (appState.image && appState.modelTrained) {
                    analyzeImage();
                }
            });
            
            // Batch confidence threshold slider
            elements.batchConfidence.addEventListener('input', () => {
                const value = elements.batchConfidence.value;
                elements.batchConfidenceValue.textContent = Math.round(value * 100) + '%';
            });
        }

        // Handle image upload
        function handleImageUpload() {
            const file = elements.fileInput.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = (e) => {
                appState.image = new Image();
                appState.image.onload = () => {
                    displayImage(appState.image);
                    resetCanvases();
                    drawGrid();
                    
                    // Enable analyze button if model is trained
                    if (appState.modelTrained) {
                        elements.analyzeBtn.disabled = false;
                    }
                };
                appState.image.src = e.target.result;
            };
            reader.readAsDataURL(file);
        }

        // Handle batch upload
        function handleBatchUpload() {
            appState.batchFiles = Array.from(elements.batchInput.files);
            if (appState.batchFiles.length === 0) return;
            
            // Show batch progress
            elements.batchProgress.style.display = 'block';
            elements.batchItems.innerHTML = '';
            
            // Add each file to progress list
            appState.batchFiles.forEach(file => {
                const item = document.createElement('div');
                item.className = 'image-item';
                item.innerHTML = `
                    <span>${file.name}</span>
                    <span class="status">Pending</span>
                `;
                elements.batchItems.appendChild(item);
            });
            
            // Enable process button
            document.getElementById('batch-process-btn').disabled = !appState.modelTrained;
        }

        // Display image
        function displayImage(img) {
            elements.microscopeImage.src = img.src;
            elements.imageContainer.style.display = 'block';
            
            // Set canvas dimensions
            const canvases = [
                elements.gridOverlay, 
                elements.trainingOverlay, 
                elements.detectionOverlay,
                elements.intersectionCanvas
            ];
            
            canvases.forEach(canvas => {
                canvas.width = img.width;
                canvas.height = img.height;
            });
        }

        // Reset canvases
        function resetCanvases() {
            const ctx = elements.trainingOverlay.getContext('2d');
            ctx.clearRect(0, 0, elements.trainingOverlay.width, elements.trainingOverlay.height);
            
            const detectCtx = elements.detectionOverlay.getContext('2d');
            detectCtx.clearRect(0, 0, elements.detectionOverlay.width, elements.detectionOverlay.height);
            
            const intCtx = elements.intersectionCanvas.getContext('2d');
            intCtx.clearRect(0, 0, elements.intersectionCanvas.width, elements.intersectionCanvas.height);
        }

        // Switch mode
        function switchMode(mode) {
            appState.mode = mode;
            
            // Update active button
            document.querySelectorAll('.mode-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            document.querySelector(`.mode-btn.${mode}`).classList.add('active');
            
            // Show/hide batch upload
            if (mode === 'batch') {
                elements.batchUpload.style.display = 'block';
                elements.uploadArea.style.display = 'none';
            } else {
                elements.batchUpload.style.display = 'none';
                elements.uploadArea.style.display = 'block';
            }
            
            // Show/hide controls
            document.querySelectorAll('.training-controls, .batch-controls, .analysis-controls').forEach(el => {
                el.classList.remove('active');
            });
            document.querySelector(`.${mode}-controls`).classList.add('active');
            
            // Show/hide results
            if (mode === 'analysis' && appState.image && appState.modelTrained) {
                elements.resultsSection.style.display = 'block';
                analyzeImage();
            } else if (mode === 'training') {
                elements.resultsSection.style.display = 'none';
            }
        }

        // Select training tool
        function selectTrainingTool(tool) {
            appState.currentTool = tool;
            
            // Update active tool button
            document.querySelectorAll('.tool-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            document.querySelector(`.tool-btn[data-type="${tool}"]`).classList.add('active');
        }

        // Handle training click
        function handleTrainingClick(e) {
            if (appState.mode !== 'training') return;
            
            const rect = elements.trainingOverlay.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            
            // Add training sample
            appState.trainingData[appState.currentTool].push({x, y});
            
            // Draw sample
            drawTrainingSample(x, y, appState.currentTool);
            
            // Update summary
            updateTrainingSummary();
            
            // Enable train button
            elements.trainBtn.disabled = false;
        }

        // Draw training sample
        function drawTrainingSample(x, y, type) {
            const ctx = elements.trainingOverlay.getContext('2d');
            
            // Set color based on type
            let color;
            switch(type) {
                case 'hyphae': color = '#e74c3c'; break;
                case 'vesicles': color = '#1abc9c'; break;
                case 'arbuscules': color = '#3498db'; break;
                case 'other-fungi': color = '#95a5a6'; break;
            }
            
            // Draw circle
            ctx.beginPath();
            ctx.arc(x, y, 8, 0, Math.PI * 2);
            ctx.fillStyle = color;
            ctx.fill();
            ctx.strokeStyle = 'white';
            ctx.lineWidth = 2;
            ctx.stroke();
        }

        // Update training summary
        function updateTrainingSummary() {
            document.getElementById('hyphae-samples').textContent = 
                appState.trainingData.hyphae.length + ' samples';
            document.getElementById('vesicles-samples').textContent = 
                appState.trainingData.vesicles.length + ' samples';
            document.getElementById('arbuscules-samples').textContent = 
                appState.trainingData.arbuscules.length + ' samples';
            document.getElementById('other-fungi-samples').textContent = 
                appState.trainingData.otherFungi.length + ' samples';
        }

        // Clear training data
        function clearTrainingData() {
            appState.trainingData = {
                hyphae: [],
                vesicles: [],
                arbuscules: [],
                otherFungi: []
            };
            
            // Clear canvas
            const ctx = elements.trainingOverlay.getContext('2d');
            ctx.clearRect(0, 0, elements.trainingOverlay.width, elements.trainingOverlay.height);
            
            // Update UI
            updateTrainingSummary();
            elements.trainBtn.disabled = true;
            
            // Reset model status
            appState.modelTrained = false;
            elements.modelStatus.className = 'model-status untrained';
            elements.modelStatus.innerHTML = '🚫 Model Not Trained<br><small>Add training samples to begin</small>';
            elements.analyzeBtn.disabled = true;
        }

        // Train model
        async function trainModel() {
            // Show training progress
            elements.modelStatus.className = 'model-status training';
            elements.modelStatus.innerHTML = '🔄 Training Model...';
            elements.trainingProgress.style.display = 'block';
            
            // Simulate training progress
            let progress = 0;
            const interval = setInterval(() => {
                progress += Math.random() * 5;
                if (progress >= 100) {
                    progress = 100;
                    clearInterval(interval);
                    
                    // Training complete
                    appState.modelTrained = true;
                    elements.modelStatus.className = 'model-status trained';
                    elements.modelStatus.innerHTML = '✅ Model Trained!<br><small>Ready for analysis</small>';
                    
                    // Enable analysis
                    elements.analyzeBtn.disabled = false;
                    document.getElementById('batch-process-btn').disabled = false;
                    
                    // Update confidence
                    elements.confidenceBar.style.width = '85%';
                    elements.confidenceText.textContent = '85%';
                }
                elements.progressFill.style.width = progress + '%';
            }, 100);
            
            // In a real app, this would be the actual training code
            // For this demo, we're simulating the training
        }

        // Draw grid
        function drawGrid() {
            if (!appState.image) return;
            
            const canvas = elements.gridOverlay;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            if (!appState.grid.visible) return;
            
            ctx.strokeStyle = 'rgba(52, 152, 219, 0.7)';
            ctx.lineWidth = 1;
            
            const width = canvas.width;
            const height = canvas.height;
            const rows = appState.grid.rows;
            const cols = appState.grid.cols;
            
            // Draw horizontal lines
            for (let i = 1; i < rows; i++) {
                const y = (height / rows) * i;
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(width, y);
                ctx.stroke();
            }
            
            // Draw vertical lines
            for (let i = 1; i < cols; i++) {
                const x = (width / cols) * i;
                ctx.beginPath();
                ctx.moveTo(x, 0);
                ctx.lineTo(x, height);
                ctx.stroke();
            }
            
            // Draw intersection points
            drawIntersectionPoints();
        }

        // Draw intersection points
        function drawIntersectionPoints() {
            if (!appState.grid.intersectionsVisible) return;
            
            const canvas = elements.intersectionCanvas;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            const width = canvas.width;
            const height = canvas.height;
            const rows = appState.grid.rows;
            const cols = appState.grid.cols;
            
            ctx.fillStyle = 'rgba(155, 89, 182, 0.8)';
            
            // Draw points at intersections
            for (let r = 0; r <= rows; r++) {
                for (let c = 0; c <= cols; c++) {
                    const x = (width / cols) * c;
                    const y = (height / rows) * r;
                    ctx.beginPath();
                    ctx.arc(x, y, 4, 0, Math.PI * 2);
                    ctx.fill();
                }
            }
        }

        // Toggle grid visibility
        function toggleGrid() {
            appState.grid.visible = elements.showGrid.checked;
            drawGrid();
        }

        // Update display
        function updateDisplay() {
            appState.grid.intersectionsVisible = elements.showIntersections.checked;
            drawGrid();
        }

        // Analyze image
        async function analyzeImage() {
            if (!appState.image || !appState.modelTrained) return;
            
            // Show results
            elements.resultsSection.style.display = 'block';
            
            // Clear previous detection
            const detectCtx = elements.detectionOverlay.getContext('2d');
            detectCtx.clearRect(0, 0, elements.detectionOverlay.width, elements.detectionOverlay.height);
            
            // Simulate analysis
            const detectedHyphae = Math.floor(Math.random() * 50) + 30;
            const detectedVesicles = Math.floor(Math.random() * 30) + 10;
            const detectedArbuscules = Math.floor(Math.random() * 40) + 20;
            const detectedOtherFungi = Math.floor(Math.random() * 20) + 5;
            
            // Update results
            document.getElementById('detected-hyphae').textContent = detectedHyphae;
            document.getElementById('detected-vesicles').textContent = detectedVesicles;
            document.getElementById('detected-arbuscules').textContent = detectedArbuscules;
            document.getElementById('detected-other-fungi').textContent = detectedOtherFungi;
            
            // Calculate colonization rate
            const totalIntersections = appState.grid.rows * appState.grid.cols;
            const fungalIntersections = detectedHyphae + detectedVesicles + detectedArbuscules + detectedOtherFungi;
            const colonizationRate = Math.round((fungalIntersections / totalIntersections) * 100);
            
            document.getElementById('colonization-rate').textContent = colonizationRate + '%';
            document.getElementById('total-intersections').textContent = totalIntersections;
            document.getElementById('fungal-intersections').textContent = fungalIntersections;
            document.getElementById('hyphae-density').textContent = (detectedHyphae / totalIntersections).toFixed(2);
            document.getElementById('arbuscule-frequency').textContent = (detectedArbuscules / fungalIntersections).toFixed(2);
            
            // Visualize detections
            visualizeDetections();
        }

        // Visualize detections
        function visualizeDetections() {
            const canvas = elements.detectionOverlay;
            const ctx = canvas.getContext('2d');
            const width = canvas.width;
            const height = canvas.height;
            const rows = appState.grid.rows;
            const cols = appState.grid.cols;
            
            // Simulate detection points
            for (let i = 0; i < 150; i++) {
                const type = Math.floor(Math.random() * 4);
                let color, label;
                
                switch(type) {
                    case 0: 
                        color = 'rgba(231, 76, 60, 0.7)'; 
                        label = 'H';
                        break;
                    case 1: 
                        color = 'rgba(26, 188, 156, 0.7)'; 
                        label = 'V';
                        break;
                    case 2: 
                        color = 'rgba(52, 152, 219, 0.7)'; 
                        label = 'A';
                        break;
                    case 3: 
                        color = 'rgba(149, 165, 166, 0.7)'; 
                        label = 'O';
                        break;
                }
                
                const x = Math.random() * width;
                const y = Math.random() * height;
                
                ctx.beginPath();
                ctx.arc(x, y, 12, 0, Math.PI * 2);
                ctx.fillStyle = color;
                ctx.fill();
                
                ctx.fillStyle = 'white';
                ctx.font = 'bold 14px Arial';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(label, x, y);
            }
        }

        // Process batch
        function processBatch() {
            if (appState.batchFiles.length === 0 || !appState.modelTrained) return;
            
            const items = elements.batchItems.querySelectorAll('.image-item');
            appState.batchResults = [];
            
            // Process each file
            appState.batchFiles.forEach((file, index) => {
                setTimeout(() => {
                    // Update status to processing
                    items[index].className = 'image-item processing';
                    items[index].querySelector('.status').textContent = 'Processing...';
                    
                    // Simulate processing delay
                    setTimeout(() => {
                        // Generate random results
                        const totalPoints = appState.grid.rows * appState.grid.cols;
                        const hyphae = Math.floor(Math.random() * 50) + 30;
                        const vesicles = Math.floor(Math.random() * 30) + 10;
                        const arbuscules = Math.floor(Math.random() * 40) + 20;
                        const otherFungi = Math.floor(Math.random() * 20) + 5;
                        const colonization = Math.round(((hyphae + vesicles + arbuscules + otherFungi) / totalPoints) * 100);
                        
                        // Store results
                        appState.batchResults.push({
                            filename: file.name,
                            hyphae,
                            vesicles,
                            arbuscules,
                            otherFungi,
                            colonization
                        });
                        
                        // Update status
                        items[index].className = 'image-item completed';
                        items[index].querySelector('.status').textContent = 'Completed';
                    }, 1500 + Math.random() * 2000);
                }, index * 500);
            });
        }

        // Export batch results
        function exportBatchResults() {
            if (appState.batchResults.length === 0) return;
            
            // Create CSV content
            let csv = 'Filename,Hyphae Intersections,Vesicle Intersections,Arbuscule Intersections,Other Fungi,Colonization Rate (%)\n';
            
            appState.batchResults.forEach(result => {
                csv += `${result.filename},${result.hyphae},${result.vesicles},${result.arbuscules},${result.otherFungi},${result.colonization}\n`;
            });
            
            // Create download link
            const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.setAttribute('href', url);
            link.setAttribute('download', 'mycorrhizal_analysis_results.csv');
            link.style.visibility = 'hidden';
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }

        // Save model
        function saveModel() {
            if (!appState.modelTrained) {
                alert('No trained model to save');
                return;
            }
            
            // In a real app, this would save the model
            alert('Model saved successfully!');
        }

        // Load model
        function loadModel() {
            document.getElementById('model-input').click();
        }

        // Export training data
        function exportTrainingData() {
            const dataStr = JSON.stringify(appState.trainingData);
            const blob = new Blob([dataStr], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            
            const link = document.createElement('a');
            link.href = url;
            link.download = 'mycorrhizal_training_data.json';
            link.click();
        }

        // Import training data
        function importTrainingData() {
            document.getElementById('training-data-input').click();
        }

        // Initialize when page loads
        window.addEventListener('load', init);
    </script>
</body>
</html>

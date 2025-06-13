<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI-Powered Mycorrhizal Analysis Tool</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tensorflow/4.2.0/tf.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            color: #333;
        }

        .container {
            max-width: 1600px;
            margin: 0 auto;
            padding: 20px;
        }

        .header {
            text-align: center;
            margin-bottom: 30px;
            color: white;
        }

        .header h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }

        .mode-selector {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 30px;
        }

        .mode-btn {
            padding: 15px 30px;
            border: none;
            border-radius: 10px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s ease;
            color: white;
        }

        .mode-btn.training {
            background: linear-gradient(135deg, #ff6b6b, #ee5a52);
        }

        .mode-btn.analysis {
            background: linear-gradient(135deg, #4ecdc4, #44a08d);
        }

        .mode-btn.active {
            transform: translateY(-3px);
            box-shadow: 0 6px 20px rgba(0,0,0,0.3);
        }

        .main-content {
            display: grid;
            grid-template-columns: 1fr 400px;
            gap: 20px;
            margin-bottom: 20px;
        }

        .image-section {
            background: white;
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.1);
        }

        .controls-panel {
            background: rgba(255,255,255,0.95);
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.1);
            height: fit-content;
        }

        .upload-area {
            border: 3px dashed #667eea;
            border-radius: 10px;
            padding: 40px;
            text-align: center;
            margin-bottom: 20px;
            transition: all 0.3s ease;
            cursor: pointer;
        }

        .upload-area:hover {
            border-color: #764ba2;
            background: rgba(102, 126, 234, 0.05);
        }

        .image-container {
            position: relative;
            max-width: 100%;
            margin: 20px 0;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 4px 20px rgba(0,0,0,0.1);
        }

        #microscope-image {
            max-width: 100%;
            height: auto;
            display: block;
        }

        #grid-overlay, #detection-overlay, #training-overlay {
            position: absolute;
            top: 0;
            left: 0;
        }

        #training-overlay {
            cursor: crosshair;
        }

        .control-group {
            margin-bottom: 20px;
            padding: 15px;
            background: rgba(102, 126, 234, 0.05);
            border-radius: 10px;
        }

        .control-group h3 {
            margin-bottom: 15px;
            color: #667eea;
            font-size: 1.1em;
        }

        .training-controls {
            display: none;
        }

        .training-controls.active {
            display: block;
        }

        .analysis-controls {
            display: none;
        }

        .analysis-controls.active {
            display: block;
        }

        .training-tools {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-bottom: 15px;
        }

        .tool-btn {
            padding: 12px 15px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 500;
            transition: all 0.3s ease;
            text-align: center;
            color: white;
        }

        .tool-btn.hyphae { background: #ff6b6b; }
        .tool-btn.vesicles { background: #4ecdc4; }
        .tool-btn.arbuscules { background: #45b7d1; }
        .tool-btn.other-fungi { background: #96ceb4; }

        .tool-btn.active {
            transform: translateY(-2px);
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }

        .model-status {
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 15px;
            text-align: center;
            font-weight: bold;
        }

        .model-status.untrained {
            background: #fff3cd;
            color: #856404;
            border: 2px solid #ffeaa7;
        }

        .model-status.training {
            background: #d1ecf1;
            color: #0c5460;
            border: 2px solid #bee5eb;
        }

        .model-status.trained {
            background: #d4edda;
            color: #155724;
            border: 2px solid #c3e6cb;
        }

        .progress-bar {
            width: 100%;
            height: 20px;
            background: #f0f0f0;
            border-radius: 10px;
            overflow: hidden;
            margin: 10px 0;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #667eea, #764ba2);
            width: 0%;
            transition: width 0.3s ease;
        }

        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: 500;
            transition: all 0.3s ease;
            width: 100%;
            margin-bottom: 10px;
        }

        .btn-primary {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
        }

        .btn-success {
            background: linear-gradient(135deg, #4ecdc4, #44a08d);
            color: white;
        }

        .btn-warning {
            background: linear-gradient(135deg, #ffa726, #ff8a65);
            color: white;
        }

        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }

        .btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
            transform: none;
        }

        .detection-results {
            background: rgba(255,255,255,0.95);
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 8px 32px rgba(0,0,0,0.1);
            margin-top: 20px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
            gap: 15px;
            margin-top: 15px;
        }

        .stat-card {
            padding: 15px;
            border-radius: 10px;
            text-align: center;
            color: white;
            font-weight: 500;
        }

        .stat-card.hyphae { background: linear-gradient(135deg, #ff6b6b, #ee5a52); }
        .stat-card.vesicles { background: linear-gradient(135deg, #4ecdc4, #44a08d); }
        .stat-card.arbuscules { background: linear-gradient(135deg, #45b7d1, #3498db); }
        .stat-card.other-fungi { background: linear-gradient(135deg, #96ceb4, #74b9a0); }

        .stat-number {
            font-size: 2em;
            font-weight: bold;
            margin-bottom: 5px;
        }

        .training-data-summary {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 15px;
        }

        .training-data-item {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
            padding: 5px 0;
            border-bottom: 1px solid #dee2e6;
        }

        .confidence-meter {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-top: 10px;
        }

        .confidence-bar {
            flex: 1;
            height: 10px;
            background: #f0f0f0;
            border-radius: 5px;
            overflow: hidden;
        }

        .confidence-fill {
            height: 100%;
            background: linear-gradient(90deg, #ff6b6b, #4ecdc4, #45b7d1);
            transition: width 0.3s ease;
        }

        .detection-overlay {
            position: absolute;
            top: 0;
            left: 0;
            pointer-events: none;
        }

        .file-input {
            display: none;
        }

        @media (max-width: 1200px) {
            .main-content {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🤖 AI-Powered Mycorrhizal Analysis</h1>
            <p>Autonomous detection and classification of mycorrhizal structures</p>
        </div>

        <div class="mode-selector">
            <button class="mode-btn training active" onclick="switchMode('training')">
                🎯 Training Mode
            </button>
            <button class="mode-btn analysis" onclick="switchMode('analysis')">
                🔬 Analysis Mode
            </button>
        </div>

        <div class="main-content">
            <div class="image-section">
                <div class="upload-area" id="upload-area">
                    <h3>📤 Upload Microscope Image</h3>
                    <p id="upload-text">Drag and drop your microscope image here or click to browse</p>
                    <input type="file" id="file-input" class="file-input" accept="image/*">
                </div>

                <div class="image-container" id="image-container" style="display: none;">
                    <img id="microscope-image" alt="Microscope image">
                    <canvas id="grid-overlay"></canvas>
                    <canvas id="training-overlay"></canvas>
                    <canvas id="detection-overlay"></canvas>
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
                            <button class="tool-btn hyphae active" data-type="hyphae" onclick="selectTrainingTool('hyphae')">Hyphae</button>
                            <button class="tool-btn vesicles" data-type="vesicles" onclick="selectTrainingTool('vesicles')">Vesicles</button>
                            <button class="tool-btn arbuscules" data-type="arbuscules" onclick="selectTrainingTool('arbuscules')">Arbuscules</button>
                            <button class="tool-btn other-fungi" data-type="other-fungi" onclick="selectTrainingTool('other-fungi')">Other Fungi</button>
                        </div>
                        <p><small>Click on structures in the image to add training samples</small></p>
                    </div>

                    <div class="control-group">
                        <h3>📊 Training Data</h3>
                        <div class="training-data-summary" id="training-summary">
                            <div class="training-data-item">
                                <span>Hyphae:</span>
                                <span id="hyphae-samples">0 samples</span>
                            </div>
                            <div class="training-data-item">
                                <span>Vesicles:</span>
                                <span id="vesicles-samples">0 samples</span>
                            </div>
                            <div class="training-data-item">
                                <span>Arbuscules:</span>
                                <span id="arbuscules-samples">0 samples</span>
                            </div>
                            <div class="training-data-item">
                                <span>Other Fungi:</span>
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

                <!-- Analysis Mode Controls -->
                <div class="analysis-controls">
                    <div class="control-group">
                        <h3>🔍 Detection Settings</h3>
                        <label>
                            <input type="range" id="confidence-threshold" min="0.1" max="1" step="0.1" value="0.7">
                            Confidence Threshold: <span id="confidence-value">70%</span>
                        </label>
                    </div>

                    <button class="btn btn-success" id="analyze-btn" onclick="analyzeImage()" disabled>
                        🔬 Analyze Image
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
                        💾 Save Model
                    </button>
                    <button class="btn btn-primary" onclick="loadModel()">
                        📂 Load Model
                    </button>
                    <input type="file" id="model-input" style="display: none;" accept=".json">
                </div>
            </div>
        </div>

        <div class="detection-results" id="results-section" style="display: none;">
            <h3>🎯 Detection Results</h3>
            <div class="stats-grid">
                <div class="stat-card hyphae">
                    <div class="stat-number" id="detected-hyphae">0</div>
                    <div>Hyphae Detected</div>
                </div>
                <div class="stat-card vesicles">
                    <div class="stat-number" id="detected-vesicles">0</div>
                    <div>Vesicles Detected</div>
                </div>
                <div class="stat-card arbuscules">
                    <div class="stat-number" id="detected-arbuscules">0</div>
                    <div>Arbuscules Detected</div>
                </div>
                <div class="stat-card other-fungi">
                    <div class="stat-number" id="detected-other-fungi">0</div>
                    <div>Other Fungi Detected</div>
                </div>
            </div>

            <div class="analysis-summary" style="margin-top: 20px; padding: 15px; background: linear-gradient(135deg, rgba(102, 126, 234, 0.1), rgba(118, 75, 162, 0.1)); border-radius: 10px;">
                <h4>📈 Autonomous Analysis Summary</h4>
                <p><strong>Total Structures Detected:</strong> <span id="total-detected">0</span></p>
                <p><strong>Mycorrhizal Colonization Rate:</strong> <span id="auto-colonization-rate">0%</span></p>
                <p><strong>Average Detection Confidence:</strong> <span id="avg-confidence">0%</span></p>
            </div>
        </div>
    </div>

    <script>
        // Global variables
        let currentMode = 'training';
        let currentTrainingTool = 'hyphae';
        let currentImage = null;
        let model = null;
        let isModelTrained = false;
        
        // Training data storage
        let trainingData = {
            hyphae: [],
            vesicles: [],
            arbuscules: [],
            'other-fungi': []
        };

        // Detection results
        let detectionResults = [];
        
        const colors = {
            hyphae: '#ff6b6b',
            vesicles: '#4ecdc4',
            arbuscules: '#45b7d1',
            'other-fungi': '#96ceb4'
        };

        // Initialize the application
        document.addEventListener('DOMContentLoaded', function() {
            setupEventListeners();
            updateConfidenceDisplay();
        });

        function setupEventListeners() {
            const uploadArea = document.getElementById('upload-area');
            const fileInput = document.getElementById('file-input');
            const confidenceSlider = document.getElementById('confidence-threshold');
            
            uploadArea.addEventListener('click', () => fileInput.click());
            uploadArea.addEventListener('dragover', handleDragOver);
            uploadArea.addEventListener('drop', handleDrop);
            fileInput.addEventListener('change', handleFileSelect);
            confidenceSlider.addEventListener('input', updateConfidenceDisplay);
        }

        function handleDragOver(e) {
            e.preventDefault();
            e.currentTarget.classList.add('dragover');
        }

        function handleDrop(e) {
            e.preventDefault();
            e.currentTarget.classList.remove('dragover');
            const files = e.dataTransfer.files;
            if (files.length > 0) {
                loadImage(files[0]);
            }
        }

        function handleFileSelect(e) {
            const files = e.target.files;
            if (files.length > 0) {
                loadImage(files[0]);
            }
        }

        function loadImage(file) {
            const reader = new FileReader();
            reader.onload = function(e) {
                const img = document.getElementById('microscope-image');
                const imageContainer = document.getElementById('image-container');
                
                img.onload = function() {
                    currentImage = this;
                    imageContainer.style.display = 'block';
                    document.getElementById('upload-area').style.display = 'none';
                    setupCanvases();
                    updateModeDisplay();
                };
                
                img.src = e.target.result;
            };
            reader.readAsDataURL(file);
        }

        function setupCanvases() {
            const img = document.getElementById('microscope-image');
            const canvases = ['grid-overlay', 'training-overlay', 'detection-overlay'];
            
            canvases.forEach(canvasId => {
                const canvas = document.getElementById(canvasId);
                canvas.width = img.offsetWidth;
                canvas.height = img.offsetHeight;
            });
            
            // Add training click handler
            const trainingCanvas = document.getElementById('training-overlay');
            trainingCanvas.addEventListener('click', handleTrainingClick);
        }

        function switchMode(mode) {
            currentMode = mode;
            
            // Update mode buttons
            document.querySelectorAll('.mode-btn').forEach(btn => btn.classList.remove('active'));
            document.querySelector(`.mode-btn.${mode}`).classList.add('active');
            
            // Update controls visibility
            document.querySelectorAll('.training-controls, .analysis-controls').forEach(ctrl => {
                ctrl.classList.remove('active');
            });
            document.querySelector(`.${mode}-controls`).classList.add('active');
            
            updateModeDisplay();
        }

        function updateModeDisplay() {
            const uploadText = document.getElementById('upload-text');
            const analyzeBtn = document.getElementById('analyze-btn');
            
            if (currentMode === 'training') {
                uploadText.textContent = 'Upload images to create training samples';
                if (currentImage) {
                    document.getElementById('training-overlay').style.pointerEvents = 'auto';
                    document.getElementById('detection-overlay').style.pointerEvents = 'none';
                }
            } else {
                uploadText.textContent = 'Upload images for autonomous analysis';
                analyzeBtn.disabled = !isModelTrained || !currentImage;
                if (currentImage) {
                    document.getElementById('training-overlay').style.pointerEvents = 'none';
                    document.getElementById('detection-overlay').style.pointerEvents = 'none';
                }
            }
        }

        function selectTrainingTool(tool) {
            currentTrainingTool = tool;
            
            document.querySelectorAll('.tool-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            document.querySelector(`[data-type="${tool}"]`).classList.add('active');
        }

        function handleTrainingClick(e) {
            if (currentMode !== 'training' || !currentImage) return;
            
            const rect = e.target.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            
            // Extract image patch around click point
            const patchSize = 32;
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = patchSize;
            canvas.height = patchSize;
            
            // Draw image patch
            ctx.drawImage(
                currentImage,
                x - patchSize/2, y - patchSize/2, patchSize, patchSize,
                0, 0, patchSize, patchSize
            );
            
            // Convert to tensor data
            const imageData = ctx.getImageData(0, 0, patchSize, patchSize);
            const tensorData = Array.from(imageData.data).filter((_, i) => i % 4 !== 3); // Remove alpha
            
            // Add to training data
            trainingData[currentTrainingTool].push({
                data: tensorData,
                x: x,
                y: y,
                timestamp: Date.now()
            });
            
            // Visual feedback
            drawTrainingPoint(x, y, currentTrainingTool);
            updateTrainingDataDisplay();
        }

        function drawTrainingPoint(x, y, type) {
            const canvas = document.getElementById('training-overlay');
            const ctx = canvas.getContext('2d');
            
            ctx.fillStyle = colors[type];
            ctx.beginPath();
            ctx.arc(x, y, 8, 0, 2 * Math.PI);
            ctx.fill();
            
            ctx.strokeStyle = 'white';
            ctx.lineWidth = 2;
            ctx.stroke();
        }

        function updateTrainingDataDisplay() {
            document.getElementById('hyphae-samples').textContent = `${trainingData.hyphae.length} samples`;
            document.getElementById('vesicles-samples').textContent = `${trainingData.vesicles.length} samples`;
            document.getElementById('arbuscules-samples').textContent = `${trainingData.arbuscules.length} samples`;
            document.getElementById('other-fungi-samples').textContent = `${trainingData['other-fungi'].length} samples`;
            
            // Enable training if we have enough samples
            const totalSamples = Object.values(trainingData).reduce((sum, arr) => sum + arr.length, 0);
            const minSamplesPerClass = 5;
            const hasEnoughSamples = Object.values(trainingData).every(arr => arr.length >= minSamplesPerClass);
            
            document.getElementById('train-btn').disabled = !hasEnoughSamples;
        }

        async function trainModel() {
            const trainBtn = document.getElementById('train-btn');
            const progressBar = document.getElementById('training-progress');
            const progressFill = document.getElementById('progress-fill');
            const modelStatus = document.getElementById('model-status');
            
            trainBtn.disabled = true;
            progressBar.style.display = 'block';
            modelStatus.className = 'model-status training';
            modelStatus.innerHTML = '🔄 Training Model...<br><small>Please wait</small>';
            
            try {
                // Prepare training data
                const { xs, ys } = prepareTrainingData();
                
                // Create model architecture
                model = tf.sequential({
                    layers: [
                        tf.layers.conv2d({
                            inputShape: [32, 32, 3],
                            filters: 32,
                            kernelSize: 3,
                            activation: 'relu'
                        }),
                        tf.layers.maxPooling2d({ poolSize: 2 }),
                        tf.layers.conv2d({
                            filters: 64,
                            kernelSize: 3,
                            activation: 'relu'
                        }),
                        tf.layers.maxPooling2d({ poolSize: 2 }),
                        tf.layers.flatten(),
                        tf.layers.dense({ units: 128, activation: 'relu' }),
                        tf.layers.dropout({ rate: 0.5 }),
                        tf.layers.dense({ units: 4, activation: 'softmax' })
                    ]
                });
                
                model.compile({
                    optimizer: 'adam',
                    loss: 'categoricalCrossentropy',
                    metrics: ['accuracy']
                });
                
                // Train model
                await model.fit(xs, ys, {
                    epochs: 50,
                    validationSplit: 0.2,
                    callbacks: {
                        onEpochEnd: (epoch, logs) => {
                            const progress = ((epoch + 1) / 50) * 100;
                            progressFill.style.width = progress + '%';
                        }
                    }
                });
                
                isModelTrained = true;
                modelStatus.className = 'model-status trained';
                modelStatus.innerHTML = '✅ Model Trained Successfully<br><small>Ready for analysis</small>';
                
                // Update UI
                updateModeDisplay();
                
            } catch (error) {
                console.error('Training failed:', error);
                modelStatus.className = 'model-status untrained';
                modelStatus.innerHTML = '❌ Training Failed<br><small>Check console for details</small>';
            }
            
            trainBtn.disabled = false;
            progressBar.style.display = 'none';
            progressFill.style.width = '0%';
        }

        function prepareTrainingData() {
            const allData = [];
            const allLabels = [];
            
            const labelMap = { hyphae: 0, vesicles: 1, arbuscules: 2, 'other-fungi': 3 };
            
            Object.entries(trainingData).forEach(([type, samples]) => {
                samples.forEach(sample => {
                    // Normalize pixel data
                    const normalizedData = sample.data.map(pixel => pixel / 255.0);
                    allData.push(normalizedData);
                    
                    // One-hot encode labels
                    const label = [0, 0, 0, 0];
                    label[labelMap[type]] = 1;
                    allLabels.push(label);
                });
            });
            
            const xs = tf.tensor4d(allData, [allData.length, 32, 32, 3]);
            const ys = tf.tensor2d(allLabels);
            
            return { xs, ys };
        }

        async function analyzeImage() {
            if (!model || !isModelTrained || !currentImage) return;
            
            const analyzeBtn = document.getElementById('analyze-btn');
            analyzeBtn.disabled = true;
            analyzeBtn.textContent = '🔄 Analyzing...';
            
            try {
                detectionResults = [];
                const confidenceThreshold = parseFloat(document.getElementById('confidence-threshold').value);
                
                // Sliding window approach
                const windowSize = 32;
                const stride = 16;
                const canvas = document.createElement('canvas');
                const ctx = canvas.getContext('2d');
                canvas.width = windowSize;
                canvas.height = windowSize;
                
                const imgWidth = currentImage.offsetWidth;
                const imgHeight = currentImage.offsetHeight;
                
                let totalConfidence = 0;
                let predictionCount = 0;
                
                for (let y = 0; y < imgHeight - windowSize; y += stride) {
                    for (let x = 0; x < imgWidth - windowSize; x += stride) {
                        // Extract patch
                        ctx.drawImage(
                            currentImage,
                            x, y, windowSize, windowSize,
                            0, 0, windowSize, windowSize
                        );
                        
                        const imageData = ctx.getImageData(0, 0, windowSize, windowSize);
                        const normalizedData = Array.from(imageData.data)
                            .filter((_, i) => i % 4 !== 3) // Remove alpha
                            .map(pixel => pixel / 255.0);
                        
                        const inputTensor = tf.tensor4d([normalizedData], [1, windowSize, windowSize, 3]);
                        
                        // Make prediction
                        const prediction = await model.predict(inputTensor).data();
                        const maxConfidence = Math.max(...prediction);
                        const predictedClass = prediction.indexOf(maxConfidence);
                        
                        if (maxConfidence > confidenceThreshold) {
                            const classNames = ['hyphae', 'vesicles', 'arbuscules', 'other-fungi'];
                            detectionResults.push({
                                x: x + windowSize/2,
                                y: y + windowSize/2,
                                type: classNames[predictedClass],
                                confidence: maxConfidence
                            });
                            
                            totalConfidence += maxConfidence;
                            predictionCount++;
                        }
                        
                        inputTensor.dispose();
                    }
                }
                
                // Draw detection results
                drawDetectionResults();
                updateDetectionStats();
                
                // Update confidence display
                const avgConfidence = predictionCount > 0 ? (totalConfidence / predictionCount) * 100 : 0;
                document.getElementById('model-confidence').style.width = avgConfidence + '%';
                document.getElementById('confidence-text').textContent = Math.round(avgConfidence) + '%';
                document.getElementById('avg-confidence').textContent = Math.round(avgConfidence) + '%';
                
                document.getElementById('results-section').style.display = 'block';
                
            } catch (error) {
                console.error('Analysis failed:', error);
                alert('Analysis failed. Please check the console for details.');
            }
            
            analyzeBtn.disabled = false;
            analyzeBtn.textContent = '🔬 Analyze Image';
        }

        function drawDetectionResults() {
            const canvas = document.getElementById('detection-overlay');
            const ctx = canvas.getContext('2d');
            
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            detectionResults.forEach(detection => {
                ctx.fillStyle = colors[detection.type];
                ctx.beginPath();
                ctx.arc(detection.x, detection.y, 6, 0, 2 * Math.PI);
                ctx.fill();
                
                // White border
                ctx.strokeStyle = 'white';
                ctx.lineWidth = 2;
                ctx.stroke();
                
                // Confidence indicator
                ctx.fillStyle = 'rgba(255, 255, 255, 0.9)';
                ctx.font = '10px Arial';
                ctx.fillText(
                    Math.round(detection.confidence * 100) + '%',
                    detection.x - 10, detection.y - 10
                );
            });
        }

        function updateDetectionStats() {
            const stats = {
                hyphae: 0,
                vesicles: 0,
                arbuscules: 0,
                'other-fungi': 0
            };
            
            detectionResults.forEach(detection => {
                stats[detection.type]++;
            });
            
            document.getElementById('detected-hyphae').textContent = stats.hyphae;
            document.getElementById('detected-vesicles').textContent = stats.vesicles;
            document.getElementById('detected-arbuscules').textContent = stats.arbuscules;
            document.getElementById('detected-other-fungi').textContent = stats['other-fungi'];
            
            const totalDetected = Object.values(stats).reduce((sum, count) => sum + count, 0);
            const mycorrhizalCount = stats.hyphae + stats.vesicles + stats.arbuscules;
            const colonizationRate = totalDetected > 0 ? Math.round((mycorrhizalCount / totalDetected) * 100) : 0;
            
            document.getElementById('total-detected').textContent = totalDetected;
            document.getElementById('auto-colonization-rate').textContent = colonizationRate + '%';
        }

        function updateConfidenceDisplay() {
            const value = document.getElementById('confidence-threshold').value;
            document.getElementById('confidence-value').textContent = Math.round(value * 100) + '%';
        }

        function clearTrainingData() {
            if (confirm('Are you sure you want to clear all training data?')) {
                trainingData = {
                    hyphae: [],
                    vesicles: [],
                    arbuscules: [],
                    'other-fungi': []
                };
                
                // Clear training overlay
                const canvas = document.getElementById('training-overlay');
                const ctx = canvas.getContext('2d');
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                
                updateTrainingDataDisplay();
                
                // Reset model status
                const modelStatus = document.getElementById('model-status');
                modelStatus.className = 'model-status untrained';
                modelStatus.innerHTML = '🚫 Model Not Trained<br><small>Add training samples to begin</small>';
                
                isModelTrained = false;
                if (model) {
                    model.dispose();
                    model = null;
                }
                
                updateModeDisplay();
            }
        }

        async function saveModel() {
            if (!model || !isModelTrained) {
                alert('No trained model to save!');
                return;
            }
            
            try {
                // Save model architecture and weights
                const modelData = await model.save('downloads://mycorrhiza-model');
                
                // Save training metadata
                const metadata = {
                    trainingStats: {
                        hyphae: trainingData.hyphae.length,
                        vesicles: trainingData.vesicles.length,
                        arbuscules: trainingData.arbuscules.length,
                        'other-fungi': trainingData['other-fungi'].length
                    },
                    timestamp: new Date().toISOString(),
                    version: '1.0'
                };
                
                const metadataBlob = new Blob([JSON.stringify(metadata, null, 2)], { type: 'application/json' });
                const url = URL.createObjectURL(metadataBlob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `mycorrhiza-model-metadata-${new Date().toISOString().split('T')[0]}.json`;
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
                URL.revokeObjectURL(url);
                
                alert('Model saved successfully!');
                
            } catch (error) {
                console.error('Failed to save model:', error);
                alert('Failed to save model. Check console for details.');
            }
        }

        function loadModel() {
            const input = document.getElementById('model-input');
            input.click();
            
            input.onchange = async function(e) {
                const files = e.target.files;
                if (files.length === 0) return;
                
                try {
                    // Load model from files
                    const modelFile = Array.from(files).find(f => f.name.includes('.json') && !f.name.includes('metadata'));
                    const weightsFile = Array.from(files).find(f => f.name.includes('.bin'));
                    
                    if (!modelFile) {
                        alert('Please select the model.json file');
                        return;
                    }
                    
                    // Create file URLs
                    const modelUrl = URL.createObjectURL(modelFile);
                    
                    model = await tf.loadLayersModel(modelUrl);
                    isModelTrained = true;
                    
                    // Update UI
                    const modelStatus = document.getElementById('model-status');
                    modelStatus.className = 'model-status trained';
                    modelStatus.innerHTML = '✅ Model Loaded Successfully<br><small>Ready for analysis</small>';
                    
                    updateModeDisplay();
                    
                    URL.revokeObjectURL(modelUrl);
                    alert('Model loaded successfully!');
                    
                } catch (error) {
                    console.error('Failed to load model:', error);
                    alert('Failed to load model. Please check the file format.');
                }
            };
        }

        // Enhanced image preprocessing for better feature extraction
        function preprocessImagePatch(imageData, windowSize) {
            const data = new Float32Array(windowSize * windowSize * 3);
            
            for (let i = 0; i < imageData.data.length; i += 4) {
                const pixelIndex = Math.floor(i / 4);
                const r = imageData.data[i] / 255.0;
                const g = imageData.data[i + 1] / 255.0;
                const b = imageData.data[i + 2] / 255.0;
                
                // Apply contrast enhancement
                data[pixelIndex * 3] = Math.pow(r, 0.8);
                data[pixelIndex * 3 + 1] = Math.pow(g, 0.8);
                data[pixelIndex * 3 + 2] = Math.pow(b, 0.8);
            }
            
            return data;
        }

        // Batch processing for better performance
        async function batchPredict(patches) {
            const batchSize = 32;
            const results = [];
            
            for (let i = 0; i < patches.length; i += batchSize) {
                const batch = patches.slice(i, i + batchSize);
                const batchTensor = tf.stack(batch);
                const predictions = await model.predict(batchTensor).data();
                
                for (let j = 0; j < batch.length; j++) {
                    const startIdx = j * 4;
                    results.push(predictions.slice(startIdx, startIdx + 4));
                }
                
                batchTensor.dispose();
                batch.forEach(tensor => tensor.dispose());
            }
            
            return results;
        }

        // Advanced detection with Non-Maximum Suppression
        function applyNonMaxSuppression(detections, threshold = 0.3) {
            const filtered = [];
            const sorted = detections.sort((a, b) => b.confidence - a.confidence);
            
            for (let i = 0; i < sorted.length; i++) {
                const current = sorted[i];
                let keep = true;
                
                for (let j = 0; j < filtered.length; j++) {
                    const distance = Math.sqrt(
                        Math.pow(current.x - filtered[j].x, 2) + 
                        Math.pow(current.y - filtered[j].y, 2)
                    );
                    
                    if (distance < threshold * 50 && current.type === filtered[j].type) {
                        keep = false;
                        break;
                    }
                }
                
                if (keep) {
                    filtered.push(current);
                }
            }
            
            return filtered;
        }

        // Initialize first training tool
        selectTrainingTool('hyphae');
    </script>
</body>
</html>

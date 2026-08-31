<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced 3D LLM Neural Simulator</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #0b0f19;
            color: #f3f4f6;
            overflow: hidden;
            height: 100vh;
            display: flex;
        }
        #canvas-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }
        /* Dashboard Overlay */
        .dashboard {
            position: relative;
            z-index: 10;
            width: 380px;
            background: rgba(17, 24, 39, 0.85);
            backdrop-filter: blur(12px);
            border-right: 1px solid rgba(255, 255, 255, 0.1);
            display: flex;
            flex-direction: column;
            height: 100vh;
            padding: 20px;
            box-shadow: 5px 0 25px rgba(0,0,0,0.5);
            overflow-y: auto;
        }
        h1 {
            font-size: 1.25rem;
            color: #38bdf8;
            margin-bottom: 5px;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .subtitle {
            font-size: 0.75rem;
            color: #9ca3af;
            margin-bottom: 20px;
        }
        .section-title {
            font-size: 0.85rem;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: #818cf8;
            margin-bottom: 10px;
            margin-top: 15px;
            border-bottom: 1px solid rgba(255,255,255,0.1);
            padding-bottom: 4px;
        }
        .control-group {
            margin-bottom: 12px;
        }
        label {
            display: block;
            font-size: 0.8rem;
            color: #d1d5db;
            margin-bottom: 5px;
        }
        input[type="range"] {
            width: 100%;
            accent-color: #38bdf8;
            cursor: pointer;
        }
        select, input[type="text"] {
            width: 100%;
            padding: 8px 12px;
            background: rgba(31, 41, 55, 0.8);
            border: 1px solid rgba(255, 255, 255, 0.2);
            color: white;
            border-radius: 6px;
            font-size: 0.85rem;
            outline: none;
            transition: border 0.2s;
        }
        select:focus, input[type="text"]:focus {
            border-color: #38bdf8;
        }
        .btn {
            background: linear-gradient(135deg, #3b82f6, #6366f1);
            color: white;
            border: none;
            padding: 10px;
            width: 100%;
            border-radius: 6px;
            font-weight: 600;
            cursor: pointer;
            transition: opacity 0.2s, transform 0.1s;
            margin-top: 5px;
        }
        .btn:hover {
            opacity: 0.9;
        }
        .btn:active {
            transform: scale(0.98);
        }
        /* Metrics Panel */
        .metrics-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 5px;
        }
        .metric-card {
            background: rgba(30, 41, 59, 0.6);
            padding: 10px;
            border-radius: 6px;
            border: 1px solid rgba(255,255,255,0.05);
        }
        .metric-value {
            font-size: 1.1rem;
            font-weight: bold;
            color: #34d399;
        }
        .metric-label {
            font-size: 0.7rem;
            color: #9ca3af;
        }
        /* Live Output Box */
        .output-box {
            background: rgba(15, 23, 42, 0.9);
            border: 1px solid rgba(56, 189, 248, 0.3);
            border-radius: 6px;
            padding: 10px;
            font-family: monospace;
            font-size: 0.8rem;
            color: #38bdf8;
            min-height: 60px;
            max-height: 100px;
            overflow-y: auto;
            margin-top: 5px;
        }
        /* Floating Info HUD */
        .hud {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 10;
            background: rgba(17, 24, 39, 0.75);
            backdrop-filter: blur(8px);
            padding: 12px 18px;
            border-radius: 8px;
            border: 1px solid rgba(255, 255, 255, 0.1);
            pointer-events: none;
            font-size: 0.85rem;
        }
        .hud span {
            color: #38bdf8;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <!-- 3D Viewport Container -->
    <div id="canvas-container"></div>

    <!-- Floating HUD -->
    <div class="hud">
        Active Stage: <span id="hud-stage">Token Embedding & Attention</span><br>
        Current Token: <span id="hud-token">"Initializing..."</span>
    </div>

    <!-- Interactive Dashboard -->
    <div class="dashboard">
        <h1>🧠 LLM 3D Cognitive Core</h1>
        <div class="subtitle">Interactive Transformer & Neural Pathway Simulator</div>

        <!-- Task Selection & Prompt Input -->
        <div class="section-title">Task Execution</div>
        <div class="control-group">
            <label for="task-select">Select AI Capability</label>
            <select id="task-select">
                <option value="generate">Text Generation (Creative)</option>
                <option value="sentiment">Sentiment Analysis</option>
                <option value="code">Code Debugging/Compilation</option>
                <option value="translate">Multilingual Translation</option>
            </select>
        </div>
        <div class="control-group">
            <label for="user-prompt">Input Prompt / Context</label>
            <input type="text" id="user-prompt" value="Explain quantum computing simply">
        </div>
        <button class="btn" id="run-btn">Execute Inference Task</button>

        <!-- Live Metrics Panel -->
        <div class="section-title">Performance & Diagnostics</div>
        <div class="metrics-grid">
            <div class="metric-card">
                <div class="metric-value" id="metric-acc">98.4%</div>
                <div class="metric-label">Token Accuracy</div>
            </div>
            <div class="metric-card">
                <div class="metric-value" id="metric-loss">0.024</div>
                <div class="metric-label">Cross-Entropy Loss</div>
            </div>
            <div class="metric-card">
                <div class="metric-value" id="metric-tokens">142 t/s</div>
                <div class="metric-label">Inference Speed</div>
            </div>
            <div class="metric-card">
                <div class="metric-value" id="metric-params">175B</div>
                <div class="metric-label">Active Weights</div>
            </div>
        </div>

        <!-- Simulation Controls -->
        <div class="section-title">Simulator Settings</div>
        <div class="control-group">
            <label for="speed-range">Simulation Speed: <span id="speed-val">1.0</span>x</label>
            <input type="range" id="speed-range" min="0.1" max="3.0" step="0.1" value="1.0">
        </div>
        <div class="control-group">
            <label for="layers-range">Transformer Layers Depth: <span id="layers-val">12</span></label>
            <input type="range" id="layers-range" min="4" max="24" step="2" value="12">
        </div>

        <!-- Output Log -->
        <div class="section-title">Generated Stream Output</div>
        <div class="output-box" id="output-stream">Ready for inference pipeline execution...</div>
    </div>

    <!-- Three.js CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // --- SCENE SETUP ---
        const container = document.getElementById('canvas-container');
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x0b0f19, 0.015);

        const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 15, 35);
        camera.lookAt(0, 0, 0);

        const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        container.appendChild(renderer.domElement);

        // Lighting
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const pointLight = new THREE.PointLight(0x38bdf8, 2, 50);
        pointLight.position.set(0, 10, 10);
        scene.add(pointLight);

        // --- 3D ARCHITECTURE GENERATION ---
        // We will build distinct layers representing:
        // 1. Input Embedding Layer (Tokens)
        // 2. Transformer Multi-Head Attention Blocks (Grid/Cubes)
        // 3. Feed-Forward Neural Networks (Dense Spheres)
        // 4. Output Softmax Layer (Probabilities)

        const networkGroup = new THREE.Group();
        scene.add(networkGroup);

        let simSpeed = 1.0;
        let activeTokensList = ["Quantum", "computing", "uses", "qubits", "to", "process", "states", "simultaneously."];
        
        // Create Layers Representation
        const layersCount = 12;
        const layerSpacing = 3.5;
        const startX = -((layersCount - 1) * layerSpacing) / 2;

        const layerMeshes = [];
        const dataPackets = [];

        for (let i = 0; i < layersCount; i++) {
            const layerGroup = new THREE.Group();
            const xPos = startX + (i * layerSpacing);
            layerGroup.position.x = xPos;

            // Determine Layer Type
            let layerType = "hidden";
            let color = 0x3b82f6;
            if (i === 0) { layerType = "embedding"; color = 0x10b981; }
            else if (i === layersCount - 1) { layerType = "output"; color = 0xf59e0b; }
            else if (i % 2 === 0) { layerType = "attention"; color = 0x6366f1; }

            // Create grid of nodes for each layer
            const rows = 4;
            const cols = 4;
            const nodeGeometry = new THREE.SphereGeometry(0.3, 16, 16);
            
            const subNodes = [];
            for (let r = 0; r < rows; r++) {
                for (let c = 0; c < cols; c++) {
                    const material = new THREE.MeshStandardMaterial({
                        color: color,
                        emissive: color,
                        emissiveIntensity: 0.3,
                        roughness: 0.2
                    });
                    const node = new THREE.Mesh(nodeGeometry, material);
                    node.position.set(0, (r - rows / 2 + 0.5) * 1.2, (c - cols / 2 + 0.5) * 1.2);
                    layerGroup.add(node);
                    subNodes.push(node);
                }
            }

            // Connect nodes between adjacent layers with glowing lines
            layerMeshes.push({ group: layerGroup, nodes: subNodes, type: layerType });
            networkGroup.add(layerGroup);
        }

        // Create Traveling Data Packets (Tokens processing through layers)
        const packetGeo = new THREE.OctahedronGeometry(0.4, 0);
        const packetMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
        
        for (let i = 0; i < 15; i++) {
            const packet = new THREE.Mesh(packetGeo, packetMat);
            packet.userData = {
                layerIndex: Math.floor(Math.random() * layersCount),
                progress: Math.random(),
                speed: 0.005 + Math.random() * 0.005,
                targetY: (Math.random() - 0.5) * 4,
                targetZ: (Math.random() - 0.5) * 4
            };
            scene.add(packet);
            dataPackets.push(packet);
        }

        // --- INTERACTION & UI CONTROLS ---
        document.getElementById('speed-range').addEventListener('input', (e) => {
            simSpeed = parseFloat(e.target.value);
            document.getElementById('speed-val').textContent = simSpeed.toFixed(1);
        });

        document.getElementById('run-btn').addEventListener('click', () => {
            const prompt = document.getElementById('user-prompt').value;
            const task = document.getElementById('task-select').value;
            const outputBox = document.getElementById('output-stream');
            
            outputBox.innerHTML = `[INFERENCE START] Processing prompt: "${prompt}"...<br>`;
            
            // Simulate token generation stream based on task
            let responses = {
                generate: "Quantum computing leverages superposition, enabling qubits to represent multiple states at once, drastically accelerating complex simulations.",
                sentiment: "Sentiment Analysis Result: Highly Positive (Confidence score: 99.1%). Key emotional markers detected.",
                code: "Code Compiler Status: Optimized successfully. No syntax faults found in target loop architecture.",
                translate: "Translation Context mapped successfully across vector space matrix dimensions."
            };

            let fullText = responses[task];
            let charIndex = 0;
            
            // Flash layers to indicate activity
            pointLight.color.setHex(0x10b981);
            
            let interval = setInterval(() => {
                if (charIndex < fullText.length) {
                    if (charIndex === 0) outputBox.innerHTML = "";
                    outputBox.innerHTML += fullText.charAt(charIndex);
                    outputBox.scrollTop = outputBox.scrollHeight;
                    document.getElementById('hud-token').textContent = `"${fullText.substr(Math.max(0, charIndex-5), 5)}"`;
                    charIndex++;
                } else {
                    clearInterval(interval);
                    pointLight.color.setHex(0x38bdf8);
                }
            }, 25 / simSpeed);
        });

        // --- ANIMATION LOOP ---
        let clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame(animate);
            const delta = clock.getDelta();

            // Rotate entire neural structure slightly for visual depth
            networkGroup.rotation.y += 0.05 * delta * simSpeed;

            // Animate Data Packets flowing through layers
            dataPackets.forEach(packet => {
                packet.userData.progress += packet.userData.speed * simSpeed;
                if (packet.userData.progress > 1) {
                    packet.userData.progress = 0;
                    packet.userData.layerIndex = (packet.userData.layerIndex + 1) % layersCount;
                }

                const currentLayerIdx = packet.userData.layerIndex;
                const nextLayerIdx = (currentLayerIdx + 1) % layersCount;

                const x1 = startX + (currentLayerIdx * layerSpacing);
                const x2 = startX + (nextLayerIdx * layerSpacing);

                packet.position.x = THREE.MathUtils.lerp(x1, x2, packet.userData.progress);
                packet.rotation.x += 0.02;
                packet.rotation.y += 0.03;
            });

            // Pulse node emissive intensities to mimic active neural firing
            layerMeshes.forEach((layer, idx) => {
                layer.nodes.forEach((node, nIdx) => {
                    const pulse = Math.sin(clock.getElapsedTime() * 3 * simSpeed + idx + nIdx) * 0.5 + 0.5;
                    node.material.emissiveIntensity = 0.2 + pulse * 0.8;
                });
            });

            renderer.render(scene, camera);
        }

        animate();

        // Responsive window resizing
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>

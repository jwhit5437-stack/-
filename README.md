<!DOCTYPE html>
<html lang="my">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>လယ်သမားဂိမ်း</title>
    <!-- Tailwind CSS အတွက် CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Inter', sans-serif;
            background-color: #87CEEB; /* အပြာရောင်ကောင်းကင်နောက်ခံ */
        }
        canvas {
            display: block;
        }
        #game-ui {
            position: absolute;
            top: 10px;
            left: 10px;
            z-index: 10;
            display: flex; /* အစကတည်းက ပေါ်နေအောင်ထားမည် */
            flex-direction: column;
            gap: 10px;
            padding: 10px;
            background: rgba(255, 255, 255, 0.8);
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        .ui-panel {
            padding: 10px 15px;
            background: #fff;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .ui-panel span {
            font-size: 1.1rem;
            font-weight: 600;
        }
        .ui-button {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 20px;
            cursor: pointer;
            font-size: 1rem;
            font-weight: bold;
            transition: background-color 0.3s, transform 0.2s;
        }
        .ui-button:hover {
            background-color: #45a049;
            transform: translateY(-2px);
        }
        .ui-button:active {
            transform: translateY(0);
        }
        .harvester-active {
            background-color: #FF5722;
        }
        #message-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.75);
            color: white;
            padding: 20px 40px;
            border-radius: 10px;
            font-size: 1.2rem;
            font-weight: bold;
            display: none;
            z-index: 20;
        }
        #crop-menu {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(255, 255, 255, 0.9);
            padding: 15px;
            border-radius: 15px;
            display: none;
            flex-direction: row;
            gap: 15px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
            z-index: 15;
        }
        .crop-button {
            width: 80px;
            height: 80px;
            border: none;
            border-radius: 10px;
            background-color: #f0f0f0;
            cursor: pointer;
            transition: transform 0.2s, background-color 0.3s;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 0.9rem;
            font-weight: 500;
        }
        .crop-button:hover {
            transform: scale(1.05);
            background-color: #e0e0e0;
        }
        .crop-icon {
            font-size: 2rem;
            margin-bottom: 5px;
        }
        .timer-label {
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 5px 10px;
            border-radius: 10px;
            font-size: 1.2rem;
            font-weight: bold;
            white-space: nowrap;
            display: inline-block;
        }
    </style>
    <!-- Three.js Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- OrbitControls for camera movement -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <!-- CSS2DRenderer for text overlays -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/renderers/CSS2DRenderer.js"></script>
    <!-- UI Elements -->
    <div id="game-ui">
        <div class="ui-panel">
            <span id="coin-count">💰 Silver Coins: 100</span>
        </div>
        <div class="ui-panel">
            <span id="user-id-display">User ID: N/A</span>
        </div>
        <div class="flex flex-row gap-2">
            <button id="buy-plot-btn" class="ui-button">မြေကွက်ဝယ်မည် ($50)</button>
            <button id="harvester-btn" class="ui-button">အားလုံးရိတ်သိမ်းမည်</button>
        </div>
    </div>
    <div id="message-box"></div>
    <div id="crop-menu">
        <button class="crop-button" data-crop="rice">
            <span class="crop-icon">🌾</span>
            <span>စပါး</span>
        </button>
        <button class="crop-button" data-crop="corn">
            <span class="crop-icon">🌽</span>
            <span>ပြောင်း</span>
        </button>
    </div>
    <!-- Firebase Libraries and Game Logic -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        window.onload = function() {
            console.log("Window loaded, starting game initialization...");

            // Global Firebase variables
            window.firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
            window.appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
            window.initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

            // Global variables for game state
            let player = {
                coins: 100,
                plots: [],
                username: ""
            };
            let plots = []; // Array to store all plot objects

            // Scene, Camera, and Renderer setup
            let scene, camera, renderer, labelRenderer, controls, raycaster;
            let mouse = new THREE.Vector2();
            let selectedPlot = null;
            let plotPrice = 50;
            let plotSize = 25;
            
            // Growth times for each crop in seconds
            const growthTimes = {
                rice: 5,
                corn: 10
            };
            
            // UI elements
            const gameUI = document.getElementById('game-ui');
            const userIdDisplay = document.getElementById('user-id-display');
            const coinCountUI = document.getElementById('coin-count');
            const buyPlotBtn = document.getElementById('buy-plot-btn');
            const cropMenuUI = document.getElementById('crop-menu');
            const messageBox = document.getElementById('message-box');
            const harvesterBtn = document.getElementById('harvester-btn');

            // Firestore Setup
            let db, gameDocRef;

            /**
             * Creates a 3D label for the timer using CSS2DObject.
             * @param {number} readyTime The timestamp when the crop will be ready.
             * @returns {THREE.Object3D} A CSS2DObject containing the timer text.
             */
            function createTimerLabel(readyTime) {
                const timeRemaining = Math.max(0, Math.floor((readyTime - Date.now()) / 1000));
                const minutes = Math.floor(timeRemaining / 60);
                const seconds = timeRemaining % 60;
                
                const timerDiv = document.createElement('div');
                timerDiv.className = 'timer-label';
                timerDiv.innerText = `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
                
                const timerLabel = new THREE.CSS2DObject(timerDiv);
                timerLabel.position.set(0, 25, 0); // Position above the crop
                return timerLabel;
            }

            // Function to create a Three.js crop mesh with improved visuals.
            function createCropMesh(cropType) {
                const cropGroup = new THREE.Group();
                
                if (cropType === 'rice') {
                    // Rice stalk (a group of thin cylinders) - More realistic cluster
                    const stalkMaterial = new THREE.MeshStandardMaterial({ color: 0x6B8E23 });
                    const leafMaterial = new THREE.MeshStandardMaterial({ color: 0x8FBC8F, side: THREE.DoubleSide });
                    const headMaterial = new THREE.MeshStandardMaterial({ color: 0xFFD700 });
                    const baseHeight = 10;
                    
                    // Create multiple stalks in a cluster
                    for (let i = 0; i < 5; i++) {
                        const stalkGeometry = new THREE.CylinderGeometry(0.3, 0.5, baseHeight + Math.random() * 4, 8);
                        const stalk = new THREE.Mesh(stalkGeometry, stalkMaterial);
                        stalk.position.x = (Math.random() - 0.5) * 6;
                        stalk.position.z = (Math.random() - 0.5) * 6;
                        stalk.position.y = baseHeight / 2;
                        stalk.castShadow = true;
                        cropGroup.add(stalk);
                    }

                    // Add leaves (curved planes)
                    const leafGeometry = new THREE.PlaneGeometry(3, 8);
                    const leafCount = 4;
                    for (let i = 0; i < leafCount; i++) {
                        const leaf = new THREE.Mesh(leafGeometry, leafMaterial);
                        leaf.position.x = (Math.random() - 0.5) * 8;
                        leaf.position.z = (Math.random() - 0.5) * 8;
                        leaf.position.y = baseHeight * 0.7;
                        leaf.rotation.x = Math.PI / 2; // Lie flat first
                        leaf.rotation.z = Math.random() * Math.PI; // Randomize rotation
                        leaf.rotation.y = Math.random() * 0.5; // Slight bend
                        leaf.castShadow = true;
                        cropGroup.add(leaf);
                    }

                    // Rice heads (more detailed panicle shape with small spheres)
                    const headGeometry = new THREE.SphereGeometry(0.5, 8, 8);
                    const headCount = 10 + Math.floor(Math.random() * 5);
                    for (let i = 0; i < headCount; i++) {
                        const head = new THREE.Mesh(headGeometry, headMaterial);
                        head.position.x = (Math.random() - 0.5) * 6;
                        head.position.y = baseHeight + 5 + Math.random() * 5;
                        head.position.z = (Math.random() - 0.5) * 6;
                        head.castShadow = true;
                        cropGroup.add(head);
                    }
                } else if (cropType === 'corn') {
                    // Corn stalk (main cylinder, slightly tapered and segmented)
                    const stalkMaterial = new THREE.MeshStandardMaterial({ color: 0x8B4513 });
                    const stalkHeight = 18;
                    const stalkSegmentCount = 5;
                    for (let i = 0; i < stalkSegmentCount; i++) {
                        const segmentHeight = stalkHeight / stalkSegmentCount;
                        const segmentRadius = 1.5 - i * 0.1; // Tapering effect
                        const segmentGeometry = new THREE.CylinderGeometry(segmentRadius, segmentRadius + 0.1, segmentHeight, 12);
                        const segment = new THREE.Mesh(segmentGeometry, stalkMaterial);
                        segment.position.y = segmentHeight * (i + 0.5);
                        segment.castShadow = true;
                        cropGroup.add(segment);
                    }

                    // Corn cobs (more detailed with two cobs)
                    const cobGeometry = new THREE.CylinderGeometry(1.5, 1.5, 6, 12);
                    const cobMaterial = new THREE.MeshStandardMaterial({ color: 0xFFA500 });
                    
                    const cob1 = new THREE.Mesh(cobGeometry, cobMaterial);
                    cob1.position.set(2.5, 10, 0);
                    cob1.rotation.z = Math.PI / 4;
                    cob1.castShadow = true;
                    cropGroup.add(cob1);

                    const cob2 = new THREE.Mesh(cobGeometry, cobMaterial);
                    cob2.position.set(-3, 8, 0);
                    cob2.rotation.z = -Math.PI / 4;
                    cob2.castShadow = true;
                    cropGroup.add(cob2);

                    // Leaves (multiple curved planes with a more drooping effect)
                    const leafGeometry = new THREE.PlaneGeometry(8, 12);
                    const leafMaterial = new THREE.MeshStandardMaterial({ color: 0x228B22, side: THREE.DoubleSide });
                    
                    const leaf1 = new THREE.Mesh(leafGeometry, leafMaterial);
                    leaf1.position.set(0, 15, 0);
                    leaf1.rotation.x = Math.PI / 2;
                    leaf1.rotation.y = Math.PI / 6;
                    leaf1.rotation.z = 0.5;
                    leaf1.castShadow = true;
                    cropGroup.add(leaf1);
                    
                    const leaf2 = new THREE.Mesh(leafGeometry, leafMaterial);
                    leaf2.position.set(0, 10, 0);
                    leaf2.rotation.x = Math.PI / 2;
                    leaf2.rotation.y = -Math.PI / 6;
                    leaf2.rotation.z = -0.5;
                    leaf2.castShadow = true;
                    cropGroup.add(leaf2);
                }
                
                cropGroup.userData.type = cropType;
                return cropGroup;
            }

            // Function to create a Three.js plot mesh
            function createPlot(x, z, id) {
                const plotGeometry = new THREE.PlaneGeometry(plotSize, plotSize);
                const plotMaterial = new THREE.MeshStandardMaterial({ color: 0x964B00, side: THREE.DoubleSide });
                const plot = new THREE.Mesh(plotGeometry, plotMaterial);
                plot.rotation.x = Math.PI / 2;
                plot.position.set(x, 0.1, z);
                plot.receiveShadow = true;
                plot.userData = { id: id, isPlanted: false, crop: null, readyTime: null, startTime: null, timerLabel: null };
                return plot;
            }

            // Function to initialize the 3D scene
            function initThreeJS() {
                // Scene creation
                scene = new THREE.Scene();
                scene.background = new THREE.Color(0x87CEEB); // Sky blue color

                // Camera setup
                camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
                camera.position.set(0, 100, 100);

                // Renderer setup
                renderer = new THREE.WebGLRenderer({ antialias: true });
                renderer.setSize(window.innerWidth, window.innerHeight);
                renderer.setPixelRatio(window.devicePixelRatio);
                renderer.shadowMap.enabled = true; // Enable shadows
                document.body.appendChild(renderer.domElement);

                // Label Renderer setup for 3D text
                labelRenderer = new THREE.CSS2DRenderer();
                labelRenderer.setSize(window.innerWidth, window.innerHeight);
                labelRenderer.domElement.style.position = 'absolute';
                labelRenderer.domElement.style.top = '0px';
                labelRenderer.domElement.style.pointerEvents = 'none'; // So it doesn't block clicks
                document.body.appendChild(labelRenderer.domElement);

                // OrbitControls for camera movement
                controls = new THREE.OrbitControls(camera, renderer.domElement);
                controls.enableDamping = true; // Smooth camera movement
                controls.dampingFactor = 0.05;
                controls.maxPolarAngle = Math.PI / 2 - 0.1; // Restrict camera from going below the ground

                // Lighting
                const ambientLight = new THREE.AmbientLight(0xffffff, 0.8); // Softer light
                scene.add(ambientLight);

                const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
                directionalLight.position.set(100, 200, 50);
                directionalLight.castShadow = true; // Enable shadows from the light
                directionalLight.shadow.mapSize.width = 2048;
                directionalLight.shadow.mapSize.height = 2048;
                directionalLight.shadow.camera.left = -200;
                directionalLight.shadow.camera.right = 200;
                directionalLight.shadow.camera.top = 200;
                directionalLight.shadow.camera.bottom = -200;
                scene.add(directionalLight);

                // Raycaster for clicking
                raycaster = new THREE.Raycaster();
                
                // Background environment (Ground, Mountains, Trees)
                createEnvironment();

                // Event Listeners
                window.addEventListener('resize', onWindowResize, false);
                renderer.domElement.addEventListener('click', onClick, false);
                buyPlotBtn.addEventListener('click', buyNewPlot);
                cropMenuUI.addEventListener('click', plantCrop);
                harvesterBtn.addEventListener('click', harvestAllReadyCrops);
            }

            // Function to create the ground, mountains, and trees
            function createEnvironment() {
                // Ground
                const groundGeometry = new THREE.PlaneGeometry(500, 500);
                const groundMaterial = new THREE.MeshStandardMaterial({ color: 0x82b740, side: THREE.DoubleSide }); // Green ground color
                const ground = new THREE.Mesh(groundGeometry, groundMaterial);
                ground.rotation.x = Math.PI / 2;
                ground.receiveShadow = true; // Ground receives shadows
                scene.add(ground);

                // Mountains (simple cartoon style)
                const mountainGeometry = new THREE.ConeGeometry(80, 200, 6);
                const mountainMaterial = new THREE.MeshStandardMaterial({ color: 0x888888 });
                
                const mountain1 = new THREE.Mesh(mountainGeometry, mountainMaterial);
                mountain1.position.set(200, 100, -200);
                mountain1.castShadow = true;
                scene.add(mountain1);

                const mountain2 = new THREE.Mesh(mountainGeometry, mountainMaterial);
                mountain2.position.set(-250, 120, -150);
                mountain2.scale.set(1.2, 1.2, 1.2);
                mountain2.castShadow = true;
                scene.add(mountain2);

                // Trees (simple cartoon style)
                const treeTrunkGeometry = new THREE.CylinderGeometry(5, 5, 20);
                const treeTrunkMaterial = new THREE.MeshStandardMaterial({ color: 0x8B4513 });
                
                const treeLeavesGeometry = new THREE.DodecahedronGeometry(15);
                const treeLeavesMaterial = new THREE.MeshStandardMaterial({ color: 0x228B22 });

                function createTree(x, z) {
                    const trunk = new THREE.Mesh(treeTrunkGeometry, treeTrunkMaterial);
                    trunk.position.set(x, 10, z);
                    trunk.castShadow = true;
                    scene.add(trunk);

                    const leaves = new THREE.Mesh(treeLeavesGeometry, treeLeavesMaterial);
                    leaves.position.set(x, 25, z);
                    leaves.castShadow = true;
                    scene.add(leaves);
                }

                createTree(150, 100);
                createTree(-120, -80);
                createTree(-100, 150);
                createTree(80, -180);
            }

            // Function to buy a new farm plot
            function buyNewPlot() {
                if (player.coins >= plotPrice) {
                    player.coins -= plotPrice;
                    
                    const plotX = (plots.length % 5) * (plotSize + 5) - (5 * (plotSize + 5)) / 2;
                    const plotZ = Math.floor(plots.length / 5) * (plotSize + 5) - (5 * (plotSize + 5)) / 2;
                    const plot = createPlot(plotX, plotZ, plots.length);
                    
                    scene.add(plot);
                    plots.push(plot);

                    showMessage(`မြေကွက်အသစ်ဝယ်ယူပြီးပါပြီ!`, 2000);
                    saveGame(); // Save game after buying a plot

                } else {
                    showMessage("ငွေပမာဏ မလုံလောက်ပါ!", 2000);
                }
            }

            // Function to plant a crop on a selected plot
            function plantCrop(event) {
                const button = event.target.closest('.crop-button');
                if (!button) {
                    return;
                }

                if (selectedPlot && !selectedPlot.userData.isPlanted) {
                    const cropType = button.dataset.crop;
                    
                    const cropMesh = createCropMesh(cropType);

                    if (cropMesh) {
                        cropMesh.position.copy(selectedPlot.position);
                        cropMesh.position.y += 5; // Adjust position to be above the plot
                        
                        // Set initial scale to a small value for growth animation
                        cropMesh.scale.set(0.1, 0.1, 0.1); 
                        
                        selectedPlot.userData.crop = cropMesh;
                        selectedPlot.userData.isPlanted = true;
                        // Store the start time and ready time
                        selectedPlot.userData.startTime = Date.now();
                        selectedPlot.userData.readyTime = Date.now() + (growthTimes[cropType] * 1000);
                        scene.add(cropMesh);
                        
                        // Create and attach the timer label
                        const timerLabel = createTimerLabel(selectedPlot.userData.readyTime);
                        selectedPlot.userData.timerLabel = timerLabel;
                        cropMesh.add(timerLabel);
                    }
                    
                    showMessage(`${cropType === 'rice' ? 'စပါး' : 'ပြောင်း'} စိုက်ပျိုးပြီးပါပြီ!`, 2000);
                    cropMenuUI.style.display = 'none';
                    selectedPlot = null;
                    saveGame(); // Save game after planting
                }
            }

            // Function to harvest crops
            function harvestCrop(plot) {
                const cropType = plot.userData.crop.userData.type;
                const reward = 20;
                player.coins += reward;

                // Remove crop from scene and reset plot state
                if (scene && plot.userData.crop) {
                    scene.remove(plot.userData.crop);
                }
                if (plot.userData.timerLabel) {
                    // Dispose of the timer label
                    plot.userData.crop.remove(plot.userData.timerLabel);
                }
                plot.userData.isPlanted = false;
                plot.userData.crop = null;
                plot.userData.readyTime = null;
                plot.userData.startTime = null; // Reset start time
                plot.userData.timerLabel = null;
                
                showMessage(`သီးနှံရိတ်သိမ်းပြီးပါပြီ! ${reward} Coins ရရှိခဲ့သည်!`, 2000);
                saveGame(); // Save game after harvesting
            }

            // Function to handle mouse clicks on the canvas
            function onClick(event) {
                // Calculate mouse position in normalized device coordinates
                mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
                mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

                raycaster.setFromCamera(mouse, camera);
                
                // Check for intersection with plots
                const intersects = raycaster.intersectObjects(plots);

                if (intersects.length > 0) {
                    selectedPlot = intersects[0].object;
                    
                    // Check if the plot has a crop
                    if (selectedPlot.userData.isPlanted) {
                        // Check if the crop is ready to harvest
                        if (Date.now() >= selectedPlot.userData.readyTime) {
                            harvestCrop(selectedPlot);
                        } else {
                            // Show a message that the crop is not ready
                            showMessage("ဒီအပင်က ရိတ်သိမ်းဖို့ အချိန်မပြည့်သေးပါဘူး!", 2000);
                        }
                    } else {
                        // Show crop menu to plant something
                        cropMenuUI.style.display = 'flex';
                    }
                } else {
                    // If no plot is clicked, hide the crop menu
                    cropMenuUI.style.display = 'none';
                }
            }

            // Function to harvest all ready crops with one click
            function harvestAllReadyCrops() {
                let harvestedCount = 0;
                for (const plot of plots) {
                    // Check if the plot has a crop and it's ready to be harvested
                    if (plot.userData.isPlanted && Date.now() >= plot.userData.readyTime) {
                        harvestCrop(plot);
                        harvestedCount++;
                    }
                }
                if (harvestedCount > 0) {
                    showMessage(`ရိတ်သိမ်းဖို့အသင့်ဖြစ်နေတဲ့ သီးနှံ ${harvestedCount} ခုကို ရိတ်သိမ်းပြီးပါပြီ!`, 3000);
                } else {
                    showMessage(`ရိတ်သိမ်းဖို့ အသင့်ဖြစ်နေတဲ့ သီးနှံ မရှိသေးပါ!`, 2000);
                }
            }
            
            // Function to show temporary message
            function showMessage(message, duration) {
                messageBox.innerText = message;
                messageBox.style.display = 'block';
                setTimeout(() => {
                    messageBox.style.display = 'none';
                }, duration);
            }

            // Function to update the UI (coin count)
            function updateUI() {
                coinCountUI.innerText = `💰 Silver Coins: ${player.coins}`;
            }

            // Function to handle window resize
            function onWindowResize() {
                if (camera && renderer && labelRenderer) {
                    camera.aspect = window.innerWidth / window.innerHeight;
                    camera.updateProjectionMatrix();
                    renderer.setSize(window.innerWidth, window.innerHeight);
                    labelRenderer.setSize(window.innerWidth, window.innerHeight);
                }
            }

            // Main animation loop
            function animate() {
                requestAnimationFrame(animate);

                // Update crop growth animation and timers
                plots.forEach(plot => {
                    if (plot.userData.isPlanted && plot.userData.crop) {
                        const now = Date.now();
                        const elapsedTime = now - plot.userData.startTime;
                        const totalTime = plot.userData.readyTime - plot.userData.startTime;
                        const growthProgress = Math.min(elapsedTime / totalTime, 1); // Clamp progress to 1
                        
                        const newScale = 0.1 + (0.9 * growthProgress); // Scale from 0.1 to 1.0
                        plot.userData.crop.scale.set(newScale, newScale, newScale);
                        
                        // Update timer label
                        if (plot.userData.timerLabel) {
                            const timeRemaining = Math.max(0, Math.floor((plot.userData.readyTime - now) / 1000));
                            if (timeRemaining > 0) {
                                const minutes = Math.floor(timeRemaining / 60);
                                const seconds = timeRemaining % 60;
                                plot.userData.timerLabel.element.innerText = `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
                            } else {
                                plot.userData.timerLabel.element.innerText = "ရိတ်သိမ်းရန်အသင့်";
                                plot.userData.timerLabel.element.style.backgroundColor = 'rgba(76, 175, 80, 0.9)'; // Green background when ready
                            }
                        }
                    }
                });

                if (controls) {
                    controls.update(); // Update camera controls
                }

                if (renderer && scene && camera) {
                    renderer.render(scene, camera);
                    labelRenderer.render(scene, camera); // Render the 2D labels
                }
            }

            // Function to initialize the game
            async function startGame() {
                // First, initialize the Three.js scene
                initThreeJS();

                // Firestore ref
                const appId = window.appId;
                const userId = window.userId;
                db = window.db;
                gameDocRef = doc(db, "artifacts", appId, "users", userId, "gameData", "state");

                // Check if user has a saved game
                const docSnap = await getDoc(gameDocRef);

                if (docSnap.exists()) {
                    // Load existing game
                    const savedData = docSnap.data();
                    player.coins = savedData.coins;
                    player.username = savedData.username || 'Anonymous User'; // Use a default name if not found
                    console.log("Loading game for", player.username);

                    const savedPlots = JSON.parse(savedData.plots);
                    savedPlots.forEach(plotData => {
                        const plot = createPlot(plotData.x, plotData.z, plotData.id);
                        plot.userData = { ...plot.userData, ...plotData };
                        if (plotData.isPlanted) {
                            // Recreate crop mesh
                            const cropMesh = createCropMesh(plotData.cropType);
                            if (cropMesh) {
                                cropMesh.position.copy(plot.position);
                                cropMesh.position.y += 5;
                                plot.userData.crop = cropMesh;
                                scene.add(cropMesh);

                                // Recreate timer
                                const timerLabel = createTimerLabel(plot.userData.readyTime);
                                plot.userData.timerLabel = timerLabel;
                                cropMesh.add(timerLabel);
                            }
                        }
                        plots.push(plot);
                        scene.add(plot);
                    });

                } else {
                    // New game
                    player.username = 'Anonymous User';
                    console.log("Creating new game for", player.username);
                    // Create a single initial plot
                    const plot = createPlot(0, 0, 0);
                    plots.push(plot);
                    scene.add(plot);

                    // Save initial state to Firestore
                    saveGame();
                }

                // gameUI.style.display = 'flex';
                userIdDisplay.innerText = `User ID: ${window.userId}`;
                
                // Start the animation loop
                animate();
                
                // Set up real-time listener for game data
                onSnapshot(gameDocRef, (doc) => {
                    if (doc.exists()) {
                        const updatedData = doc.data();
                        player.coins = updatedData.coins;
                        updateUI();
                    }
                });
            }
            
            // Function to save game state to Firestore
            async function saveGame() {
                if (!db || !gameDocRef) return;
                const plotsData = plots.map(plot => ({
                    id: plot.userData.id,
                    x: plot.position.x,
                    z: plot.position.z,
                    isPlanted: plot.userData.isPlanted,
                    readyTime: plot.userData.readyTime,
                    cropType: plot.userData.crop ? plot.userData.crop.userData.type : null,
                    startTime: plot.userData.startTime
                }));

                try {
                    await setDoc(gameDocRef, {
                        username: player.username,
                        coins: player.coins,
                        plots: JSON.stringify(plotsData)
                    });
                    console.log("Game state saved!");
                } catch (e) {
                    console.error("Error saving game state: ", e);
                }
            }
            
            // Initialize Firebase
            if (Object.keys(window.firebaseConfig).length > 0) {
                const app = initializeApp(window.firebaseConfig);
                window.db = getFirestore(app);
                window.auth = getAuth(app);
            } else {
                console.error("Firebase configuration not found. Game data will not be saved.");
                // Proceed without Firebase if config is missing
                window.db = null;
                window.auth = null;
            }

            // Authentication State Listener
            if (window.auth) {
                onAuthStateChanged(window.auth, async (user) => {
                    if (user) {
                        window.userId = user.uid;
                        console.log("User authenticated:", window.userId);
                        // ဂိမ်းကို တန်းစတင်ပါမည်
                        startGame();
                    } else {
                        console.log("User not authenticated, attempting to sign in anonymously...");
                        try {
                            if (window.initialAuthToken) {
                                await signInWithCustomToken(window.auth, window.initialAuthToken);
                            } else {
                                await signInAnonymously(window.auth);
                            }
                        } catch (error) {
                            console.error("Firebase sign-in failed:", error);
                        }
                    }
                });
            }
        };
    </script>
</body>
</html>

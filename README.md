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
            display: flex;
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
            display: flex;
            gap: 15px;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
            display: none;
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
        #login-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 30;
        }
        #login-form {
            background: white;
            padding: 40px;
            border-radius: 15px;
            text-align: center;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
            width: 90%;
            max-width: 400px;
        }
        #login-form h2 {
            font-size: 2rem;
            margin-bottom: 20px;
            color: #333;
        }
        #login-form input {
            width: 100%;
            padding: 12px;
            margin-bottom: 20px;
            border: 2px solid #ddd;
            border-radius: 8px;
            font-size: 1rem;
            box-sizing: border-box;
        }
        #login-form button {
            width: 100%;
            padding: 15px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 1.2rem;
            font-weight: bold;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        #login-form button:hover {
            background-color: #45a049;
        }
        #loading-screen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 2rem;
            z-index: 40;
        }
    </style>
    <!-- Three.js Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- OrbitControls for camera movement -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <!-- Firebase Libraries -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase variables
        window.firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        window.appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        window.initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

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
                    // Now show the login form to get the username
                    document.getElementById('login-screen').style.display = 'flex';
                } else {
                    console.log("User not authenticated, attempting to sign in...");
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
    </script>
</head>
<body>
    <!-- Login/Signup Screen -->
    <div id="login-screen">
        <div id="login-form">
            <h2 class="text-3xl font-bold mb-6 text-gray-800">ဂိမ်းထဲဝင်ရန်</h2>
            <p class="text-gray-600 mb-6">သင့်အမည်ကိုရိုက်ပြီး အကောင့်အသစ်ဖွင့်နိုင်သလို အကောင့်ဟောင်းကိုလည်း ဝင်ရောက်နိုင်ပါတယ်။</p>
            <input type="text" id="username-input" placeholder="အမည်ထည့်ပါ..." class="w-full p-3 border-2 border-gray-300 rounded-lg focus:outline-none focus:border-green-500 transition-colors">
            <button id="login-btn" class="w-full p-4 bg-green-500 text-white font-bold rounded-lg hover:bg-green-600 transition-colors">စတင်ရန်</button>
        </div>
    </div>

    <!-- Loading Screen -->
    <div id="loading-screen" class="hidden">
        <span id="loading-percentage" class="text-4xl font-bold">0%</span>
        <div class="mt-4 text-xl">Loading...</div>
    </div>

    <!-- Main Game UI -->
    <div id="game-ui" class="hidden">
        <div class="ui-panel">
            <span id="coin-count">💰 Silver Coins: 100</span>
        </div>
        <div class="ui-panel">
            <button id="buy-plot-btn" class="ui-button">မြေကွက်ဝယ်ရန် (50 💰)</button>
        </div>
        <div class="ui-panel">
            <button id="harvester-btn" class="ui-button">အလိုအလျောက် ရိတ်သိမ်းရန် 🚜</button>
        </div>
        <div class="ui-panel">
            <span id="user-id-display"></span>
        </div>
    </div>
    
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
    <div id="message-box"></div>

    <script type="module">
        // Import Firestore functions
        import { getFirestore, doc, getDoc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables for game state
        let player = {
            coins: 100,
            plots: [],
            username: ""
        };
        let plots = []; // Array to store all plot objects

        // Scene, Camera, and Renderer setup
        let scene, camera, renderer, controls, raycaster;
        let mouse = new THREE.Vector2();
        let selectedPlot = null;
        let plotPrice = 50;
        let plotSize = 25;
        const cropGrowthTime = 5; // Growth time in seconds
        
        // Harvester variables
        let isHarvesterActive = false;
        let lastHarvestTime = 0;
        const harvestInterval = 1000; // Harvest a crop every 1 second

        // UI elements
        const loginScreen = document.getElementById('login-screen');
        const loginBtn = document.getElementById('login-btn');
        const usernameInput = document.getElementById('username-input');
        const loadingScreen = document.getElementById('loading-screen');
        const loadingPercentageSpan = document.getElementById('loading-percentage');
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
         * Simulates a loading process with a percentage counter.
         * @param {string} username The name of the user.
         */
        function startLoadingAndGame(username) {
            loginScreen.style.display = 'none';
            loadingScreen.style.display = 'flex';
            
            let percentage = 0;
            const loadingInterval = setInterval(() => {
                percentage += 1;
                loadingPercentageSpan.innerText = `${percentage}%`;
                
                if (percentage >= 100) {
                    clearInterval(loadingInterval);
                    player.username = username;
                    startGame();
                }
            }, 20); // Adjust interval for faster/slower loading
        }

        // Function to initialize the game after login
        async function startGame() {
            // First, initialize the Three.js scene
            initThreeJS();

            // Firestore ref
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
            const userId = window.userId;
            db = window.db;
            gameDocRef = doc(db, "artifacts", appId, "users", userId, "gameData", "state");

            // Check if user has a saved game
            const docSnap = await getDoc(gameDocRef);

            if (docSnap.exists()) {
                // Load existing game
                const savedData = docSnap.data();
                player.coins = savedData.coins;
                player.username = savedData.username;
                console.log("Loading game for", player.username);

                const savedPlots = JSON.parse(savedData.plots);
                savedPlots.forEach(plotData => {
                    const plot = createPlot(plotData.x, plotData.z, plotData.id);
                    plot.userData = { ...plot.userData, ...plotData };
                    if (plotData.isPlanted && plotData.readyTime) {
                        // Recreate crop mesh
                        const cropMesh = createCropMesh(plotData.cropType);
                        if (cropMesh) {
                            cropMesh.position.copy(plot.position);
                            cropMesh.position.y += 5;
                            plot.userData.crop = cropMesh;
                            scene.add(cropMesh);
                        }
                    }
                    plots.push(plot);
                    scene.add(plot);
                });

            } else {
                // New game
                console.log("Creating new game for", player.username);
                // Create a single initial plot
                const plot = createPlot(0, 0, 0);
                plots.push(plot);
                scene.add(plot);

                // Save initial state to Firestore
                saveGame();
            }

            loadingScreen.style.display = 'none';
            gameUI.style.display = 'flex';
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
                cropType: plot.userData.crop ? plot.userData.crop.userData.type : null
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

        // Function to create a Three.js plot mesh
        function createPlot(x, z, id) {
            const plotGeometry = new THREE.PlaneGeometry(plotSize, plotSize);
            const plotMaterial = new THREE.MeshStandardMaterial({ color: 0x964B00, side: THREE.DoubleSide });
            const plot = new THREE.Mesh(plotGeometry, plotMaterial);
            plot.rotation.x = Math.PI / 2;
            plot.position.set(x, 0.1, z);
            plot.receiveShadow = true;
            plot.userData = { id: id, isPlanted: false, crop: null, readyTime: null };
            return plot;
        }

        // Function to create a Three.js crop mesh
        function createCropMesh(cropType) {
            let cropMesh;
            if (cropType === 'rice') {
                const riceGeometry = new THREE.BoxGeometry(2, 10, 2);
                const riceMaterial = new THREE.MeshBasicMaterial({ color: 0xFFD700 });
                cropMesh = new THREE.Mesh(riceGeometry, riceMaterial);
            } else if (cropType === 'corn') {
                const cornGeometry = new THREE.CylinderGeometry(2, 2, 15);
                const cornMaterial = new THREE.MeshBasicMaterial({ color: 0xFFA500 });
                cropMesh = new THREE.Mesh(cornGeometry, cornMaterial);
            }
            if (cropMesh) {
                cropMesh.userData.type = cropType;
            }
            return cropMesh;
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
            harvesterBtn.addEventListener('click', toggleHarvester);
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
            if (selectedPlot && !selectedPlot.userData.isPlanted) {
                const cropType = event.target.closest('.crop-button').dataset.crop;
                
                const cropMesh = createCropMesh(cropType);

                if (cropMesh) {
                    cropMesh.position.copy(selectedPlot.position);
                    cropMesh.position.y += 5; // Adjust position to be above the plot
                    selectedPlot.userData.crop = cropMesh;
                    selectedPlot.userData.isPlanted = true;
                    selectedPlot.userData.readyTime = Date.now() + (cropGrowthTime * 1000);
                    scene.add(cropMesh);
                }
                
                showMessage(`${cropType} စိုက်ပျိုးပြီးပါပြီ!`, 2000);
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
            plot.userData.isPlanted = false;
            plot.userData.crop = null;
            plot.userData.readyTime = null;
            
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
                
                // Check if the crop is ready to harvest
                if (selectedPlot.userData.isPlanted && Date.now() >= selectedPlot.userData.readyTime) {
                    harvestCrop(selectedPlot);
                } else if (!selectedPlot.userData.isPlanted) {
                    // Show crop menu to plant something
                    cropMenuUI.style.display = 'flex';
                }
            } else {
                // If no plot is clicked, hide the crop menu
                cropMenuUI.style.display = 'none';
            }
        }

        // Function to toggle the automatic harvester
        function toggleHarvester() {
            isHarvesterActive = !isHarvesterActive;
            if (isHarvesterActive) {
                harvesterBtn.innerText = "အလိုအလျောက် ရိတ်သိမ်းခြင်း: On 🚜";
                harvesterBtn.classList.add('harvester-active');
                showMessage("အလိုအလျောက် ရိတ်သိမ်းခြင်းကို ဖွင့်လိုက်ပါပြီ!", 2000);
            } else {
                harvesterBtn.innerText = "အလိုအလျောက် ရိတ်သိမ်းရန် 🚜";
                harvesterBtn.classList.remove('harvester-active');
                showMessage("အလိုအလျောက် ရိတ်သိမ်းခြင်းကို ပိတ်လိုက်ပါပြီ!", 2000);
            }
        }

        // Function to perform automatic harvesting
        function autoHarvest() {
            const now = Date.now();
            if (now - lastHarvestTime > harvestInterval) {
                for (let i = 0; i < plots.length; i++) {
                    const plot = plots[i];
                    if (plot.userData.isPlanted && now >= plot.userData.readyTime) {
                        harvestCrop(plot);
                        lastHarvestTime = now;
                        break; // Only harvest one at a time to prevent lag
                    }
                }
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
            if (camera && renderer) {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            }
        }

        // Main animation loop
        function animate() {
            requestAnimationFrame(animate);
            if (controls) {
                controls.update(); // Update camera controls
            }

            if (isHarvesterActive) {
                autoHarvest();
            }

            if (renderer && scene && camera) {
                renderer.render(scene, camera);
            }
        }
        
        // Event listener for login button
        loginBtn.addEventListener('click', () => {
            const username = usernameInput.value.trim();
            if (username) {
                startLoadingAndGame(username);
            } else {
                showMessage("ကျေးဇူးပြု၍ သင့်အမည်ကိုထည့်ပါ!", 2000);
            }
        });

        // Start the game logic on window load
        window.onload = function () {
            // No need to call init() or animate() here, they are called after login
        };
    </script>
</body>
</html>

# Watch-Earn
<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D मोटू पतलू रन एंड अर्न गेम</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Arial, sans-serif; }
        body { background: #121212; color: #fff; text-align: center; overflow-x: hidden; }
        
        /* टॉप डैशबोर्ड */
        .header-dashboard { display: flex; justify-content: space-between; align-items: center; background: #1f1f1f; padding: 12px; margin-bottom: 5px; }
        .wallet-box { background: linear-gradient(135deg, #ff9900, #ff5500); padding: 5px 15px; border-radius: 8px; font-weight: bold; }
        
        /* मुख्य लेआउट */
        .main-container { max-width: 600px; margin: 0 auto; padding: 10px; }
        .card { background: #1e1e1e; padding: 15px; border-radius: 12px; margin-bottom: 12px; border: 1px solid #333; }
        h3 { color: #ff9900; margin-bottom: 10px; }
        
        /* 3D गेम स्क्रीन */
        #game-canvas-container { width: 100%; height: 320px; background: #000; border-radius: 10px; position: relative; overflow: hidden; border: 2px solid #ff9900; }
        .game-ui { position: absolute; top: 10px; left: 10px; background: rgba(0,0,0,0.7); padding: 8px; border-radius: 5px; text-align: left; font-size: 14px; pointer-events: none; }
        .game-controls { margin-top: 10px; display: flex; justify-content: space-around; }
        
        /* बटन्स */
        .btn { padding: 10px 20px; border: none; border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer; color: white; }
        .btn-control { background: #ff5500; width: 45%; font-size: 18px; }
        .btn-ads { background: #28a745; width: 100%; margin-top: 5px; animation: pulse 1.5s infinite; }
        .btn-redeem { background: #dc3545; width: 100%; margin-top: 5px; }
        .btn-edit { background: #17a2b8; margin: 5px; font-size: 12px; }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { box-shadow: 0 0 15px #28a745; }
            100% { transform: scale(1); }
        }
    </style>
</head>
<body>

<div class="main-container">

    <div class="header-dashboard">
        <div style="text-align: left; font-size: 13px;">
            <div id="user-id">आईडी: प्लेयर_रमन_२०२६</div>
            <button class="btn" style="background:#4285F4; padding:2px 5px; font-size:10px;" onclick="loginSystem()">लॉगिन</button>
        </div>
        <div class="wallet-box">बैलेंस: ₹<span id="user-balance">0.00</span></div>
    </div>

    <div class="card" style="padding: 5px;">
        <h3>🏃 3D मोटू रनर (लेवल सिस्टम)</h3>
        <div id="game-canvas-container">
            <div class="game-ui">
                <div>दूरी: <span id="game-distance" style="color:#00ffcc; font-weight:bold;">0</span> मीटर</div>
                <div>लेवल: <span id="game-level" style="color:#ff9900; font-weight:bold;">1</span></div>
                <div>समोसे: <span id="game-score" style="color:#ffff00; font-weight:bold;">0</span></div>
            </div>
        </div>
        
        <div class="game-controls">
            <button class="btn btn-control" onclick="moveMotu('left')">◀ बाएँ कूदें</button>
            <button class="btn btn-control" onclick="moveMotu('right')">दाएँ कूदें ▶</button>
        </div>
        <p style="font-size: 11px; color: #aaa; margin-top: 5px;">टिप: कीबोर्ड के Left/Right Arrow कीज़ से भी मोटू को कंट्रोल कर सकते हैं।</p>
    </div>

    <div class="card">
        <h3>💰 वॉच ऐड एंड अर्न (90% आपका, 10% यूज़र का)</h3>
        <button class="btn btn-ads" onclick="watchAd()">📺 वीडियो ऐड देखें (₹0.10 कमाएं)</button>
        <button class="btn btn-redeem" onclick="redeemBalance()">💸 पैसे निकालें (₹10 - ₹500 लिमिट)</button>
    </div>

    <div class="card">
        <h3>🎥 वीडियो एडिटिंग टूल्स</h3>
        <button class="btn btn-edit" onclick="alert('वीडियो कटर चालू हो रहा है...')">✂️ कट/ट्रिम</button>
        <button class="btn btn-edit" onclick="alert('शॉर्ट्स वायरल इफेक्ट्स लोड हो रहे हैं...')">✨ वायरल इफेक्ट्स</button>
    </div>

</div>

<script>
    // --- गेम और वॉलेट वेरिएबल्स ---
    let balance = 0.00;
    let myEarnings90 = 0.00; // आपकी 90% कमाई
    let distance = 0;
    let score = 0;
    let level = 1;
    let gameSpeed = 0.15; // शुरुआत की स्पीड

    // --- 3D GAME SETUP (THREE.JS) ---
    const container = document.getElementById('game-canvas-container');
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x1a1a2e); // रात का आसमान जैसा बैकग्राउंड

    // कैमरा सेटअप
    const camera = new THREE.PerspectiveCamera(60, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.set(0, 3, 6); // मोटू के पीछे और थोड़ा ऊपर कैमरा
    camera.lookAt(0, 1, 0);

    // रेंडरर
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    // लाइट्स (रोशनी)
    const light = new THREE.AmbientLight(0xffffff, 0.8);
    scene.add(light);
    const dirLight = new THREE.DirectionalLight(0xffffff, 0.5);
    dirLight.position.set(0, 5, 5);
    scene.add(dirLight);

    // 3D ट्रैक (सड़क)
    const trackGeometry = new THREE.BoxGeometry(4, 0.1, 100);
    const trackMaterial = new THREE.MeshStandardMaterial({ color: 0x333333 });
    const track = new THREE.Mesh(trackGeometry, trackMaterial);
    track.position.set(0, 0, -40);
    scene.add(track);

    // मोटू का 3D मॉडल (सरल लाल 3D बॉक्स - जिसे मोटू माना गया है)
    const motuGeometry = new THREE.BoxGeometry(0.8, 1.5, 0.8);
    const motuMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 }); // लाल रंग का मोटू
    const motu = new THREE.Mesh(motuGeometry, motuMaterial);
    motu.position.set(0, 0.75, 2);
    scene.add(motu);

    // 3D रुकावटें (Obstacles - पीले रंग के बॉक्स)
    const obstacleGeometry = new THREE.BoxGeometry(0.8, 0.8, 0.8);
    const obstacleMaterial = new THREE.MeshStandardMaterial({ color: 0xffff00 });
    const obstacle = new THREE.Mesh(obstacleGeometry, obstacleMaterial);
    obstacle.position.set(0, 0.4, -20);
    scene.add(obstacle);

    // --- मोटू का मूवमेंट (बाएँ/दाएँ) ---
    function moveMotu(direction) {
        if (direction === 'left' && motu.position.x > -1.2) {
            motu.position.x -= 1.2;
        } else if (direction === 'right' && motu.position.x < 1.2) {
            motu.position.x += 1.2;
        }
    }

    // कीबोर्ड कंट्रोल्स
    window.addEventListener('keydown', (e) => {
        if (e.key === 'ArrowLeft') moveMotu('left');
        if (e.key === 'ArrowRight') moveMotu('right');
    });

    // --- मुख्य 3D गेम लूप (यह लगातार चलता है) ---
    function animate() {
        requestAnimationFrame(animate);

        // रुकावट (Obstacle) को मोटू की तरफ बढ़ाना
        obstacle.position.z += gameSpeed;

        // अगर रुकावट पीछे निकल जाए, तो उसे वापस आगे भेजें (नया चैलेंज)
        if (obstacle.position.z > 5) {
            obstacle.position.z = -40;
            // रैंडम तरीके से बाएँ, बीच में या दाएँ सेट करें
            const lanes = [-1.2, 0, 1.2];
            obstacle.position.x = lanes[Math.floor(Math.random() * lanes.length)];
            score += 1; // एक समोसा मिला
            document.getElementById('game-score').innerText = score;
        }

        // टक्कर की चेकिंग (Collision Detection)
        if (Math.abs(motu.position.z - obstacle.position.z) < 0.8 && 
            Math.abs(motu.position.x - obstacle.position.x) < 0.5) {
            alert("ओह! मोटू टकरा गया। गेम रीस्टार्ट हो रहा है।");
            distance = 0;
            score = 0;
            level = 1;
            gameSpeed = 0.15;
            obstacle.position.z = -40;
            document.getElementById('game-score').innerText = score;
        }

        // --- दूरी (Distance) और लेवल अप लॉजिक (100m, 200m...) ---
        distance += 1;
        document.getElementById('game-distance').innerText = Math.floor(distance / 5);

        let currentMeter = Math.floor(distance / 5);

        // 100 मीटर होने पर लेवल 2
        if (currentMeter >= 100 && currentMeter < 200 && level === 1) {
            level = 2;
            gameSpeed = 0.25; // स्पीड बढ़ गई
            document.getElementById('game-level').innerText = level;
            scene.background = new THREE.Color(0x2b1a1a); // बैकग्राउंड का रंग बदला
        }
        // 200 मीटर होने पर लेवल 3
        else if (currentMeter >= 200 && level === 2) {
            level = 3;
            gameSpeed = 0.35; // स्पीड और तेज़
            document.getElementById('game-level').innerText = level;
            scene.background = new THREE.Color(0x1a2b1a); // बैकग्राउंड फिर बदला
        }

        renderer.render(scene, camera);
    }

    // 3D गेम शुरू करें
    animate();

    // --- अर्निंग और वॉलेट सिस्टम ---
    function loginSystem() {
        let id = prompt("अपनी प्लेयर आईडी डालें:");
        if(id) document.getElementById('user-id').innerText = "आईडी: " + id;
    }

    function watchAd() {
        alert("ऐड लोड हो रहा है... 3 सेकंड देखें।");
        setTimeout(() => {
            let totalRevenue = 1.00; // 1 ऐड का कुल ₹1 मिला
            let userShare = totalRevenue * 0.10; // यूज़र को 10% (₹0.10)
            let myShare = totalRevenue * 0.90;   // आपको 90% (₹0.90)

            balance += userShare;
            myEarnings90 += myShare;

            document.getElementById('user-balance').innerText = balance.toFixed(2);
            alert("बधाई! आपको ₹" + userShare.toFixed(2) + " मिले।");
            console.log("आपकी तिजोरी (90%): ₹" + myEarnings90.toFixed(2));
        }, 1500);
    }

    function redeemBalance() {
        if (balance < 10) {
            alert("न्यूनतम ₹10 होने पर ही रिडीम होगा। और ऐड देखें!");
        } else if (balance > 500) {
            alert("अधिकतम लिमिट ₹500 है।");
        } else {
            let uid = prompt("पैसे भेजने के लिए Free Fire UID या UPI आईडी डालें:");
            if (uid) {
                alert("✅ रिक्वेस्ट सबमिट हो गई! ₹" + balance.toFixed(2) + " आपके अकाउंट में भेज दिए जाएंगे।");
                balance = 0.00;
                document.getElementById('user-balance').innerText = balance.toFixed(2);
            }
        }
    }
</script>
</body>
</html>

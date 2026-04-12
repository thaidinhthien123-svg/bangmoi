<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Delta Robot Remote</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #1e1e1e;
            color: #fff;
            text-align: center;
            padding: 20px;
            margin: 0;
        }

        h1 {
            color: #00ff88;
            margin-bottom: 10px;
        }

        .count {
            font-size: 26px;
            margin: 20px 0;
            background: #333;
            padding: 18px;
            border-radius: 12px;
        }

        button {
            width: 90%;
            max-width: 400px;
            padding: 18px;
            margin: 10px 0;
            font-size: 22px;
            border-radius: 12px;
            border: none;
            font-weight: bold;
            color: white;
        }

        .start {
            background: #00cc00;
        }

        .stop {
            background: #cc0000;
        }

        .home {
            background: #0066ff;
        }

        .manual {
            background: #ff8800;
        }

        .toggle-on {
            background: #00cc00;
        }

        .toggle-off {
            background: #666666;
        }

        .stream {
            background: #00ff88;
            color: #000;
            font-size: 20px;
        }

        .cam-container {
            margin: 30px auto;
            max-width: 640px;
            background: #000;
            border: 5px solid #00ff88;
            border-radius: 12px;
            overflow: hidden;
        }

        #livecam {
            width: 100%;
            display: block;
        }

        .section-title {
            font-size: 20px;
            margin: 25px 0 10px;
            color: #00ffcc;
        }
    </style>
</head>
<body>
    <h1>🔧 DELTA ROBOT CONTROL</h1>

    <div class="count" id="counts">🔴 Red: 0 | ⚫ Black: 0 | ⚪ Silver: 0</div>

    <!-- Điều khiển cơ bản -->
    <button class="start" onclick="sendCommand('start')">▶ START HỆ THỐNG</button>
    <button class="stop" onclick="sendCommand('stop')">⏹ STOP HỆ THỐNG</button>
    <button class="home" onclick="sendCommand('home')">🏠 HOME ROBOT</button>

    <div class="section-title">🖐️ CHẠY TAY (Manual)</div>

    <!-- Cycle vẫn giữ nguyên (không toggle) -->
    <button class="manual" onclick="sendCommand('cycle1')">🔴 Cycle 1 - Red</button>
    <button class="manual" onclick="sendCommand('cycle2')">⚫ Cycle 2 - Black</button>
    <button class="manual" onclick="sendCommand('cycle3')">⚪ Cycle 3 - Silver</button>

    <!-- BĂNG TẢI TOGGLE -->
    <button id="beltBtn" class="toggle-off" onclick="toggleBelt()">
        🏎️ BĂNG TẢI OFF
    </button>

    <!-- BƠM HÚT TOGGLE -->
    <button id="suctionBtn" class="toggle-off" onclick="toggleSuction()">
        🪫 BƠM HÚT OFF
    </button>

    <!-- Stream Camera -->
    <div class="section-title">📹 CAMERA LIVE (từ máy tính)</div>
    <button id="streamBtn" class="stream" onclick="toggleStream()">📡 BẬT STREAM CAMERA</button>

    <div class="cam-container" id="camContainer" style="display:none;">
        <img id="livecam" src="" alt="Live Camera">
    </div>

    <p style="margin-top:30px; color:#aaa;">Trạng thái: <span id="status">Sẵn sàng</span></p>

    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyBfeyfNlvbhLnTciiCYIV6uTIEhzrPCRug",
            authDomain: "xulyanhxin.firebaseapp.com",
            databaseURL: "https://xulyanhxin-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "xulyanhxin"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        // Trạng thái toggle local
        let beltOn = false;
        let suctionOn = false;
        let streamEnabled = false;

        function sendCommand(cmd) {
            db.ref('commands/action').set(cmd);
            document.getElementById('status').innerHTML = `Đã gửi: <b>${cmd.toUpperCase()}</b>`;
            setTimeout(() => document.getElementById('status').innerHTML = 'Sẵn sàng', 2000);
        }

        // Toggle BĂNG TẢI
        function toggleBelt() {
            beltOn = !beltOn;
            const btn = document.getElementById('beltBtn');
            if (beltOn) {
                btn.classList.remove('toggle-off');
                btn.classList.add('toggle-on');
                btn.textContent = "🏎️ BĂNG TẢI ON";
                sendCommand('belt_on');
            } else {
                btn.classList.remove('toggle-on');
                btn.classList.add('toggle-off');
                btn.textContent = "🏎️ BĂNG TẢI OFF";
                sendCommand('belt_off');
            }
        }

        // Toggle BƠM HÚT
        function toggleSuction() {
            suctionOn = !suctionOn;
            const btn = document.getElementById('suctionBtn');
            if (suctionOn) {
                btn.classList.remove('toggle-off');
                btn.classList.add('toggle-on');
                btn.textContent = "🪫 BƠM HÚT ON";
                sendCommand('suction_on');
            } else {
                btn.classList.remove('toggle-on');
                btn.classList.add('toggle-off');
                btn.textContent = "🪫 BƠM HÚT OFF";
                sendCommand('suction_off');
            }
        }

        // Stream Camera (giữ nguyên toggle)
        function toggleStream() {
            streamEnabled = !streamEnabled;
            const btn = document.getElementById('streamBtn');
            const container = document.getElementById('camContainer');

            if (streamEnabled) {
                btn.textContent = "⛔ TẮT STREAM CAMERA";
                btn.style.background = "#ff4444";
                container.style.display = "block";
                sendCommand('stream_on');
                startListeningCamera();
            } else {
                btn.textContent = "📡 BẬT STREAM CAMERA";
                btn.style.background = "#00ff88";
                container.style.display = "none";
                sendCommand('stream_off');
                document.getElementById('livecam').src = "";
            }
        }

        function startListeningCamera() {
            db.ref('/camera/live_frame').on('value', (snap) => {
                const b64 = snap.val();
                if (b64) {
                    document.getElementById('livecam').src = "data:image/jpeg;base64," + b64;
                }
            });
        }

        // Hiển thị số lượng realtime
        db.ref('status/counts').on('value', (snap) => {
            const c = snap.val() || { red: 0, black: 0, silver: 0 };
            document.getElementById('counts').innerHTML =
                `🔴 Red: ${c.red} | ⚫ Black: ${c.black} | ⚪ Silver: ${c.silver}`;
        });
    </script>
</body>
</html>

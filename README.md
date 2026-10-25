<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>視線で選ぶ写真アプリ</title>
    <style>
        * { box-sizing: border-box; user-select: none; margin: 0; padding: 0; }
        body { font-family: "Hiragino Kaku Gothic ProN", "Meiryo", sans-serif; background-color: #1c1917; color: #f5f5f4; height: 100vh; overflow: hidden; }

        .screen { display: none; flex-direction: column; height: 100vh; padding: 20px; position: relative; }
        .screen.active { display: flex; }

        /* --- 設定画面 --- */
        .setup-container { background: #292524; border-radius: 20px; padding: 30px; max-width: 800px; margin: auto; box-shadow: 0 10px 25px rgba(0,0,0,0.8); width: 100%; max-height: 95vh; overflow-y: auto; }
        .setup-header { text-align: center; margin-bottom: 30px; border-bottom: 2px solid #44403c; padding-bottom: 15px; }
        .setup-header h1 { font-size: 24pt; color: #60a5fa; margin-bottom: 10px; }

        .upload-section { display: flex; gap: 20px; margin-bottom: 30px; flex-wrap: wrap; }
        .upload-card { flex: 1; min-width: 250px; background: #44403c; padding: 20px; border-radius: 15px; text-align: center; border: 2px dashed #78716c; }
        .upload-card h3 { color: #fca5a5; margin-bottom: 15px; font-size: 16pt; }
        .file-input { margin-bottom: 15px; width: 100%; color: #d6d3d1; }
        .preview-img { width: 100%; height: 200px; object-fit: cover; border-radius: 10px; display: none; background: #000; }

        .setting-row { display: flex; align-items: center; justify-content: space-between; margin-bottom: 15px; background: #44403c; padding: 15px 20px; border-radius: 10px; }
        .setting-row label { font-weight: bold; font-size: 12pt; }
        .setting-row select { padding: 8px 12px; font-size: 12pt; border-radius: 8px; border: none; background: #1c1917; color: white; }

        .btn { padding: 18px 30px; font-size: 18pt; font-weight: bold; color: white; border: none; border-radius: 12px; cursor: pointer; transition: background 0.2s; text-align: center; width: 100%; }
        .btn-primary { background: linear-gradient(135deg, #2563eb, #7c3aed); }
        .btn-primary:hover { opacity: 0.9; }

        /* --- 選択画面 --- */
        .camera-container { position: absolute; top: 20px; right: 20px; width: 160px; height: 120px; border-radius: 12px; overflow: hidden; border: 3px solid #60a5fa; box-shadow: 0 4px 10px rgba(0,0,0,0.5); z-index: 100; background: #000; }
        #webcam { width: 100%; height: 100%; object-fit: cover; transform: scaleX(-1); }

        .selection-area { display: flex; gap: 20px; height: 100%; flex: 1; align-items: stretch; justify-content: center; padding-top: 20px; }
        
        .choice-card { flex: 1; position: relative; border-radius: 20px; overflow: hidden; border: 6px solid #44403c; transition: all 0.2s ease-out; opacity: 0.7; display: flex; flex-direction: column; background: #000; }
        .choice-card.looking { border-color: #fde047; transform: scale(1.02); opacity: 1; box-shadow: 0 0 30px rgba(253, 224, 71, 0.4); }
        
        .choice-img { width: 100%; height: 100%; object-fit: cover; flex: 1; }
        
        /* ゲージのアニメーション部分 */
        .gauge-container { height: 25px; background: #292524; width: 100%; position: relative; }
        .gauge-bar { height: 100%; width: 0%; background: #fde047; transition: width 0.1s linear; }
        .gauge-text { position: absolute; width: 100%; text-align: center; top: 2px; font-weight: bold; font-size: 12pt; color: #1c1917; text-shadow: 0 0 5px rgba(255,255,255,0.8); }

        /* 目線のカーソル（どこを見ているか可視化する用、非表示にしてもOK） */
        #gaze-cursor { position: absolute; width: 30px; height: 30px; background-color: rgba(239, 68, 68, 0.7); border-radius: 50%; border: 3px solid white; pointer-events: none; z-index: 200; transform: translate(-50%, -50%); display: none; transition: transform 0.1s ease-out; }

        /* --- 結果画面 --- */
        .result-container { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; text-align: center; }
        .result-container h2 { font-size: 32pt; color: #fde047; margin-bottom: 20px; text-shadow: 0 4px 10px rgba(0,0,0,0.8); }
        #result-img { max-width: 80%; max-height: 60vh; border-radius: 20px; border: 8px solid #fde047; box-shadow: 0 10px 40px rgba(253, 224, 71, 0.5); object-fit: contain; margin-bottom: 40px; }
        .btn-secondary { background: #4b5563; width: auto; padding: 15px 40px; }
        
    </style>
</head>
<body>

    <!-- 1. 設定画面 -->
    <div id="setup-screen" class="screen active">
        <div class="setup-container">
            <div class="setup-header">
                <h1>👀 視線で選ぶ写真アプリ</h1>
                <p style="color: #d6d3d1;">写真を入れて、見つめた方を選ぼう！</p>
            </div>

            <div class="upload-section">
                <!-- 左の写真 -->
                <div class="upload-card">
                    <h3>👈 左の写真</h3>
                    <input type="file" id="file-left" class="file-input" accept="image/*">
                    <img id="preview-left" class="preview-img">
                </div>
                <!-- 右の写真 -->
                <div class="upload-card">
                    <h3>👉 右の写真</h3>
                    <input type="file" id="file-right" class="file-input" accept="image/*">
                    <img id="preview-right" class="preview-img">
                </div>
            </div>

            <div class="setting-row">
                <label for="gaze-sensitivity">🚀 視線の感度（動きやすさ）</label>
                <select id="gaze-sensitivity">
                    <option value="3.0">標準（少し動く）</option>
                    <option value="6.0" selected>高感度（おすすめ）</option>
                    <option value="10.0">超高感度</option>
                </select>
            </div>
            
            <div class="setting-row">
                <label for="head-assist">👤 顔の向きアシスト</label>
                <select id="head-assist">
                    <option value="0.5" selected>ON（目線＋顔の向き）</option>
                    <option value="0.0">OFF（目線のみ）</option>
                </select>
            </div>

            <button id="start-btn" class="btn btn-primary" disabled>写真を入れてスタート！</button>
            <p id="loading-msg" style="text-align: center; color: #60a5fa; font-weight: bold; margin-top: 15px; display: none;">AIモデルを準備中...</p>
        </div>
    </div>

    <!-- 2. 選択画面 -->
    <div id="selection-screen" class="screen">
        <div class="camera-container">
            <video id="webcam" autoplay playsinline muted></video>
        </div>
        
        <div id="gaze-cursor"></div>

        <div class="selection-area" id="selection-area">
            <!-- 左の選択肢 -->
            <div class="choice-card" id="choice-0">
                <img id="img-0" class="choice-img">
                <div class="gauge-container">
                    <div class="gauge-bar" id="gauge-0"></div>
                    <div class="gauge-text">2秒見つめると決定</div>
                </div>
            </div>

            <!-- 右の選択肢 -->
            <div class="choice-card" id="choice-1">
                <img id="img-1" class="choice-img">
                <div class="gauge-container">
                    <div class="gauge-bar" id="gauge-1"></div>
                    <div class="gauge-text">2秒見つめると決定</div>
                </div>
            </div>
        </div>
    </div>

    <!-- 3. 結果画面 -->
    <div id="result-screen" class="screen">
        <div class="result-container">
            <h2>✨ これを選びました！ ✨</h2>
            <img id="result-img" src="">
            <button id="retry-btn" class="btn btn-secondary">もう一度選ぶ</button>
            <button id="back-setup-btn" class="btn btn-secondary" style="margin-top: 15px; background: #374151;">設定に戻る</button>
        </div>
    </div>

    <script type="module">
        // 修正ポイント: モジュールの末尾に /vision_bundle.js を追加しています
        import { FaceLandmarker, FilesetResolver } from "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.3/vision_bundle.js";

        // UI要素
        const ui = {
            fileLeft: document.getElementById('file-left'),
            fileRight: document.getElementById('file-right'),
            previewLeft: document.getElementById('preview-left'),
            previewRight: document.getElementById('preview-right'),
            img0: document.getElementById('img-0'),
            img1: document.getElementById('img-1'),
            startBtn: document.getElementById('start-btn'),
            loadingMsg: document.getElementById('loading-msg'),
            setupScreen: document.getElementById('setup-screen'),
            selectionScreen: document.getElementById('selection-screen'),
            resultScreen: document.getElementById('result-screen'),
            video: document.getElementById('webcam'),
            gazeCursor: document.getElementById('gaze-cursor'),
            selectionArea: document.getElementById('selection-area'),
            retryBtn: document.getElementById('retry-btn'),
            backSetupBtn: document.getElementById('back-setup-btn'),
            resultImg: document.getElementById('result-img'),
            sensitivitySelect: document.getElementById('gaze-sensitivity'),
            headAssistSelect: document.getElementById('head-assist')
        };

        // 状態変数
        let faceLandmarker = null;
        let isCameraInit = false;
        let lastVideoTime = -1;
        let loopId = null;
        
        let imgDataLeft = null;
        let imgDataRight = null;

        // 視線計算用
        let currentX = window.innerWidth / 2;
        let baseNoseX = null;
        let sensitivity = 6.0;
        let headAssist = 0.5;

        // 判定用
        let currentLookTarget = -1; // 0: 左, 1: 右, -1: どちらでもない
        let lookStartTime = null;
        const REQUIRED_DWELL_TIME_MS = 2000; // 2秒
        let isDecided = false;

        // --- 1. 写真アップロード処理 ---
        function handleFileUpload(input, previewElement, isLeft) {
            const file = input.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    previewElement.src = e.target.result;
                    previewElement.style.display = 'block';
                    if (isLeft) imgDataLeft = e.target.result;
                    else imgDataRight = e.target.result;
                    checkReady();
                };
                reader.readAsDataURL(file);
            }
        }

        ui.fileLeft.addEventListener('change', () => handleFileUpload(ui.fileLeft, ui.previewLeft, true));
        ui.fileRight.addEventListener('change', () => handleFileUpload(ui.fileRight, ui.previewRight, false));

        function checkReady() {
            if (imgDataLeft && imgDataRight) {
                ui.startBtn.disabled = false;
                ui.startBtn.innerText = 'カメラを起動してスタート！';
            }
        }

        // --- 2. AIとカメラの初期化 ---
        ui.startBtn.addEventListener('click', async () => {
            ui.startBtn.disabled = true;
            ui.loadingMsg.style.display = 'block';
            
            // 画像を選択画面にセット
            ui.img0.src = imgDataLeft;
            ui.img1.src = imgDataRight;
            
            sensitivity = parseFloat(ui.sensitivitySelect.value);
            headAssist = parseFloat(ui.headAssistSelect.value);

            if (!isCameraInit) {
                await initAI();
            } else {
                startSelection();
            }
        });

        async function initAI() {
            try {
                const filesetResolver = await FilesetResolver.forVisionTasks("https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.3/wasm");
                faceLandmarker = await FaceLandmarker.createFromOptions(filesetResolver, {
                    baseOptions: { 
                        modelAssetPath: `https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/1/face_landmarker.task`, 
                        delegate: "CPU" 
                    },
                    outputFaceBlendshapes: true, 
                    runningMode: "VIDEO", 
                    numFaces: 1
                });

                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
                ui.video.srcObject = stream;
                await ui.video.play();

                isCameraInit = true;
                startSelection();
            } catch (err) {
                alert("カメラの読み込みに失敗しました。権限を確認してください。");
                ui.startBtn.disabled = false;
                ui.loadingMsg.style.display = 'none';
            }
        }

        // --- 3. 選択画面の開始 ---
        function startSelection() {
            ui.setupScreen.classList.remove('active');
            ui.resultScreen.classList.remove('active');
            ui.selectionScreen.classList.add('active');
            
            ui.loadingMsg.style.display = 'none';
            ui.startBtn.disabled = false;
            
            isDecided = false;
            resetGauges();
            baseNoseX = null;
            currentX = window.innerWidth / 2;
            ui.gazeCursor.style.display = 'block';

            if (loopId) cancelAnimationFrame(loopId);
            predictWebcam();
        }

        // --- 4. メインループ（視線判定） ---
        function predictWebcam() {
            if (isDecided || !ui.selectionScreen.classList.contains('active')) return;

            let startTimeMs = performance.now();
            
            if (ui.video.readyState >= 2 && lastVideoTime !== ui.video.currentTime) {
                lastVideoTime = ui.video.currentTime;
                const results = faceLandmarker.detectForVideo(ui.video, startTimeMs);

                if (results.faceBlendshapes && results.faceBlendshapes.length > 0) {
                    const shapes = results.faceBlendshapes[0].categories;
                    
                    // 視線の動き（左右のみ抽出）
                    const lookLeft = (shapes.find(s => s.categoryName === "eyeLookOutLeft")?.score || 0) + (shapes.find(s => s.categoryName === "eyeLookInRight")?.score || 0);
                    const lookRight = (shapes.find(s => s.categoryName === "eyeLookInLeft")?.score || 0) + (shapes.find(s => s.categoryName === "eyeLookOutRight")?.score || 0);
                    let gazeDX = (lookRight - lookLeft);

                    // 顔の向きアシスト
                    let headDX = 0;
                    if (results.faceLandmarks && results.faceLandmarks[0]) {
                        const nose = results.faceLandmarks[0][1];
                        if (baseNoseX === null) baseNoseX = nose.x;
                        headDX = (baseNoseX - nose.x) * 4.0; 
                    }

                    // 合成とカーソル移動
                    let totalDX = gazeDX + (headDX * headAssist);
                    const screenWidth = window.innerWidth;
                    const targetX = (screenWidth / 2) + (totalDX * screenWidth * 0.8 * sensitivity);
                    
                    // スムーズな補間
                    currentX += (targetX - currentX) * 0.2;
                    currentX = Math.max(0, Math.min(screenWidth, currentX));

                    // カーソル描画
                    ui.gazeCursor.style.left = `${currentX}px`;
                    ui.gazeCursor.style.top = `50%`;

                    // 左右どちらを見ているか判定
                    let targetZone = -1;
                    if (currentX < screenWidth * 0.45) {
                        targetZone = 0; // 左
                    } else if (currentX > screenWidth * 0.55) {
                        targetZone = 1; // 右
                    }

                    processDwellTime(targetZone, startTimeMs);
                } else {
                    // 顔が見つからない時はリセット
                    processDwellTime(-1, startTimeMs);
                }
            }
            loopId = requestAnimationFrame(predictWebcam);
        }

        // --- 5. 見つめ時間の処理 ---
        function processDwellTime(targetZone, currentTime) {
            // 見ている対象が変わった場合
            if (targetZone !== currentLookTarget) {
                currentLookTarget = targetZone;
                lookStartTime = targetZone !== -1 ? currentTime : null;
                resetGauges();
                updateCardHighlight();
            } 
            // 同じ対象を見続けている場合
            else if (targetZone !== -1 && lookStartTime) {
                const elapsed = currentTime - lookStartTime;
                const progress = Math.min(100, (elapsed / REQUIRED_DWELL_TIME_MS) * 100);
                
                const gauge = document.getElementById(`gauge-${targetZone}`);
                if (gauge) gauge.style.width = `${progress}%`;

                if (elapsed >= REQUIRED_DWELL_TIME_MS) {
                    decideChoice(targetZone);
                }
            }
        }

        function resetGauges() {
            document.getElementById('gauge-0').style.width = '0%';
            document.getElementById('gauge-1').style.width = '0%';
        }

        function updateCardHighlight() {
            document.getElementById('choice-0').classList.toggle('looking', currentLookTarget === 0);
            document.getElementById('choice-1').classList.toggle('looking', currentLookTarget === 1);
        }

        // --- 6. 決定時の処理 ---
        function decideChoice(chosenIndex) {
            isDecided = true;
            ui.gazeCursor.style.display = 'none';
            
            // 少しだけ選ばれた方をハイライトしたまま停止
            setTimeout(() => {
                showResult(chosenIndex);
            }, 500);
        }

        function showResult(chosenIndex) {
            ui.selectionScreen.classList.remove('active');
            ui.resultScreen.classList.add('active');
            
            if (chosenIndex === 0) {
                ui.resultImg.src = imgDataLeft;
            } else {
                ui.resultImg.src = imgDataRight;
            }
        }

        // --- 7. リトライ・戻るボタン ---
        ui.retryBtn.addEventListener('click', () => {
            startSelection();
        });

        ui.backSetupBtn.addEventListener('click', () => {
            ui.resultScreen.classList.remove('active');
            ui.setupScreen.classList.add('active');
            if (loopId) cancelAnimationFrame(loopId);
        });

    </script>
</body>
</html>


<h2> 連結至以下網址 </h2> <br>
https://teachablemachine.withgoogle.com/
點選如下圖的 Get Started <br>
<img src="01.jpg" width=300 height=200 /><br><br>

如下圖，請選擇 Image Project。 <br>

<img src="02.jpg" width=300 height=200 /><br><br>

然後點選 Startard Model <br>

<img src="03.jpg" width=300 height=200 /><br><br>

如下圖建立三個類別分別為 Phone、Power、Pen，請牢記這個個英文單字。

<img src="04.jpg" width=300 height=200 /><br><br>

然後個別類別請使用 WebCam 分別將對應的物品放至 Web Cam 前，建議至少拍攝300張以上照片。

<img src="05.jpg" width=300 height=200 /><br><br>

接下來按下 Training ，讓這三個類別進行訓練。需要等待一點時間。

<img src="06.jpg" width=300 height=200 /><br><br>

訓練完畢之後，可以按下 Export Model匯出模型。

<img src="07.jpg" width=300 height=200 /><br><br>

請切換至 Tensorflow.js 的頁籤，然後按下 Upload my model。

<img src="08.jpg" width=300 height=200 /><br><br>

此時下方有一行 Your shareable link 的網址，可以按下右邊的 Copy按鈕

<img src="09.jpg" width=300 height=200 /><br><br>



準備事項: <br>
1.請建立一個乾淨的資料夾。<br>
2.請下載 tf.min.js teachablemachine-image.min.js 這兩個檔案到資料夾中。<br>
3.請將以下範例貼到記事本，並儲存為 play.html 檔名 (附檔名要一樣，play 可以隨意改檔名)<br>
4.然後請修改兩個地方，第一個地方是修改  // ★★★ 改成你自己的模型網址 ★★★ 以下的網址，同前面的 Copy 按鈕複製的網址。<br>
4.第二個修改地方是修改 // ★★★ 改成分類的類別英文代號 ★★★ ，將你做好的分類取代原來的類別。<br>
5.以瀏覽器開啟 play.html 檔案，按下如下圖中的啟動鏡頭。<br>
<br>
<img src="10.jpg" width=300 height=200 /><br><br>


====================================================<br>
#### 使用 Teachable Machine 的範例網頁程式碼
====================================================<br>
```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>手勢辨識 + 語音播放教學</title>

    <!-- 離線 TensorFlow.js -->
    <script src="tf.min.js"></script>

    <!-- 離線 Teachable Machine Image -->
    <script src="teachablemachine-image.min.js"></script>


    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
        }
        #webcam-container {
            margin-top: 20px;
        }
        .result {
            font-size: 22px;
            margin-top: 15px;
            color: darkblue;
        }
    </style>
</head>

<body>
    <h2>🤖 手勢辨識 + 語音播放（Teachable Machine）</h2>
    <button onclick="init()">啟動鏡頭</button>

    <div id="webcam-container"></div>
    <div id="label-container"></div>
    <div class="result" id="speech-result"></div>

    <script>
        // ★★★ 改成你自己的模型網址 ★★★
        const URL = "https://teachablemachine.withgoogle.com/models/pcLWmaTcY/";


        let model, webcam, labelContainer, maxPredictions;
        let lastSpoken = ""; // 避免重複播放

        async function init() {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";

            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();

	    webcam = new tmImage.Webcam(320, 240, false);
            await webcam.setup();
            await webcam.play();
            window.requestAnimationFrame(loop);

            document.getElementById("webcam-container").appendChild(webcam.canvas);

            labelContainer = document.getElementById("label-container");
            labelContainer.innerHTML = "";
            for (let i = 0; i < maxPredictions; i++) {
                labelContainer.appendChild(document.createElement("div"));
            }
        }

        async function loop() {
            webcam.update();
            await predict();
            window.requestAnimationFrame(loop);
        }

        async function predict() {
            const predictions = await model.predict(webcam.canvas);

            let bestClass = "";
            let bestProb = 0;

            for (let i = 0; i < maxPredictions; i++) {
                const prob = predictions[i].probability;
                const text =
                    predictions[i].className + " : " +
                    (prob * 100).toFixed(1) + "%";
                labelContainer.childNodes[i].innerHTML = text;

                if (prob > bestProb) {
                    bestProb = prob;
                    bestClass = predictions[i].className;
                }
            }

            // 門檻值（避免誤判）
            if (bestProb > 0.90 && bestClass !== lastSpoken) {
                speakGesture(bestClass);
                lastSpoken = bestClass;
            }
        }

        function speakGesture(gesture) {
            let message = "";
            // ★★★ 改成分類的類別英文代號 ★★★
            switch (gesture) {
                case "Phone":
                    message = "這是 一支手機";
                    break;
                case "Power":
                    message = "這是 行動電源";
                    break;
                case "Pen":
                    message = "這是 一支筆";
                    break;
                default:
                    message = gesture;
            }

            document.getElementById("speech-result").innerText = message;

            const utterance = new SpeechSynthesisUtterance(message);
            utterance.lang = "zh-TW";   // 中文語音
            utterance.rate = 1;         // 語速
            utterance.pitch = 1;        // 音調
            speechSynthesis.speak(utterance);
        }
    </script>
</body>
</html>
```

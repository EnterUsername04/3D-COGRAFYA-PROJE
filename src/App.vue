<template>
    <div>
        <div class="min-h-[calc(100vh-120px)]">
            <div id="camera" class="h-[250px]"></div>
            <div class="controllers">
                <select @change="changeVideoSource">
                    <option>
                        {{
                            ctrls.lang === "tr" ?
                            "Kamera seç" :
                            "Choose cam"
                        }}
                    </option>
                    <option v-for="device in videoDevices" :key="device" :value="device.deviceId">
                        {{ device.label.substring(0, 12) }}
                    </option>
                </select>
                <button @click="mirrorVideo">
                    {{
                        ctrls.lang === "tr" ?
                        "Aynala" :
                        "Mirror"
                    }}
                </button>
                <button @click="refresh">
                    {{
                        ctrls.lang === "tr" ?
                        "Yenile" :
                        "Refresh"
                    }}
                </button>
                <button @click="changeLang">{{ ctrls.lang.toUpperCase() }}</button>
            </div>
            <div id="meaning" class="min-h-[250px]">
                {{
                    ctrls.lang === "tr" ?
                    motif["info-tr"] || "Açıklamasını görmek için cismi tarayabilisiniz !" :
                    motif["info-en"] || "You can scan the object to see its description ! " 
                }}
            </div>
        </div>
        <footer>
        <div>
                {{
                    ctrls.lang === "tr" ?
                    "Teknofest Eğitim Teknolojileri Yarışması" :
                    "Teknofest Educational Technologies Competition" 
                }}
     </div>
     <div>Made with 🖤 open source on <a >GitHub</a>.</div>
    </footer>
    </div>
</template>

<script>
import data from "./assets/data.json";
export default {
    data(){
        return{
            ctrls: { lang: "en", mirrored: false, scanned: false },
            videoSource: {},
            videoDevices: [],
            şekil: {}
        }
    },
    methods: {
        getVideoDevices(){
            navigator.mediaDevices.enumerateDevices().then(devices => {
                    this.videoDevices = devices.filter(device => device.kind === "videoinput");
                }
            );
        },
        changeVideoSource(e){
            window.localStorage.setItem("argumareot-turizm-videosource", JSON.stringify({ video: { deviceId: e.target.value } }));
            location.reload();
        },
        mirrorVideo(){
            this.ctrls.mirrored = !this.ctrls.mirrored;
            const canvas = document.querySelector("#camera canvas");
            canvas.classList.toggle("-scale-x-100");
        },
        refresh(){
            location.reload();
        },
        changeLang(){
            this.ctrls.lang = this.ctrls.lang === "tr" ? "en" : "tr";
        }
    },
    mounted(){
        this.getVideoDevices();
        if ("serviceWorker" in navigator) {
            navigator.serviceWorker.register("/service-worker.js");
        }
        const MODEL_URL = "https://storage.googleapis.com/tm-model/_gO2STbel/model.json";
        let classifier, videoInput, outputWidth, outputHeight;
        // video classify with ml5.js
        const classifyVideo = () => {
            classifier.classify(videoInput, (err, result) => {
                if (!err && result[0].confidence * 100 > 95 && result[0].label !== "dark"){
                    this.motif = data.filter(m => m.name === result[0].label)[0];
                    console.log(result);
                    this.ctrls.scanned = true;
                }
                if (!this.ctrls.scanned){
                    classifyVideo();
                }
            });
        }
        // p5.js functions
        const script = p => {
            p.preload = _ => {
                classifier = ml5.imageClassifier(MODEL_URL);
            }
            p.setup = _ => {
                outputWidth = window.innerWidth - 100;
                outputHeight = 250;
                const canvas = p.createCanvas(outputWidth, outputHeight);
                canvas.parent("camera");
                const videoSource = JSON.parse(window.localStorage.getItem("argumareot-turizm-videosource")) || p.VIDEO;
                videoInput = p.createCapture(videoSource);
                videoInput.size(outputWidth, outputHeight);
                videoInput.hide();
                classifyVideo();
            }
            
            p.draw = _ => {
                p.image(videoInput, 0, 0, outputWidth, outputHeight);
            }
        }
        new p5(script);
  // Classifier Variable
                let classifier;
                // Model URL
                let imageModelURL = 'https://teachablemachine.withgoogle.com/models/_gO2STbel/';

                // Video
                let video;
                let flippedVideo;
                // To store the classification
                let label = "";

                // Load the model first
                function preload() {classifier = ml5.imageClassifier(imageModelURL + 'model.json')};
                }

                function setup() {createCanvas(320, 260)};
                // Create the video
                video = createCapture(VIDEO);
                video.size(320, 240);
                video.hide();

                flippedVideo = ml5.flipImage(video);
                // Start classifying
                classifyVideo();
                }

                function draw() {background(0)};
                // Draw the video
                image(flippedVideo, 0, 0);

                // Draw the label
                fill(255);
                textSize(16);
                textAlign(CENTER);
                text(label, width / 2, height - 4);
                }

                // Get a prediction for the current video frame
                function classifyVideo() {flippedVideo = ml5.flipImage(video)}
                classifier.classify(flippedVideo, gotResult);
                flippedVideo.remove();

                }

                // When we get a result
                function gotResult(error, results) {
                    // If there is an error
                }
    // If there is an error
                if (error) {console.error(error)};
                return;
                }
                // The results are in an array ordered by confidence.
                // console.log(results[0]);
                label = results[0].label;
                // Classifiy again!
                classifyVideo();
                }
            </script></>

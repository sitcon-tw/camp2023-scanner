<template>
    <div>
        <div v-if="showOverlay" class="overlay"></div>
        <Fireworks v-show="showFireworks" ref="fireworks" :options="fireworksOptions" :style="fireworksStyle" />
        <audio ref="audioElement">
        <source :src="audioSrc" :type="audioType">這個瀏覽器不支持音頻</audio>
        <div v-if="showCongratulations" class="congratulations-modal">
            <div class="logo-container">
                <img src="/src/images/xiaostore.png" class="logo">
            </div>
            <h2>拉麵靈魂點數</h2>
            <p>哦耶~恭喜你們ww {{ ranktext }}</p>
            <button @click="closeCongratulations">關閉</button>
        </div>
    </div>
</template>

<script>
import { Fireworks } from '@fireworks-js/vue';

export default {
    components: {
        Fireworks,
    },
    data() {
        return {
            showOverlay: false,
            showFireworks: false,
            showCongratulations: false,
            audioSrc: [ '/src/audio/no1.mp3', '/src/audio/no2-3_audio.mp3' ],
            audioType: 'audio/mpeg',
            ranktext: "",
            // 閉幕式-煙火選項
            fireworksOptions: {
                autoresize: true,
                opacity: 0.5,
                acceleration: 1.05,
                friction: 0.97,
                gravity: 1.5,
                particles: 50,
                traceLength: 3,
                traceSpeed: 10,
                explosion: 5,
                intensity: 30,
                flickering: 50,
                lineStyle: 'round',
                hue: {
                    min: 0,
                    max: 360
                },
                delay: {
                    min: 10,
                    max: 10
                },
                rocketsPoint: {
                    min: 0,
                    max: 100
                },
                lineWidth: {
                    explosion: {
                        min: 1,
                        max: 3
                    },
                    trace: {
                        min: 1,
                        max: 2
                    }
                },
                brightness: {
                    min: 50,
                    max: 80
                },
                decay: {
                    min: 0.015,
                    max: 0.03
                },
                mouse: {
                    click: false,
                    move: false,
                    max: 1
                }
            },
            fireworksStyle: {
                position: 'fixed',
                top: 0,
                left: 0,
                width: '100%',
                height: '100%',
                pointerEvents: 'none',
                zIndex: 999
            },
        };
    },
    methods: {
        startFireworks(ranktext) {
            this.showOverlay = true;
            this.showFireworks = true;
            this.$nextTick(() => {
                const fireworks = this.$refs.fireworks;
                if (fireworks) {
                    fireworks.start();
                    this.$refs.audioElement.play();
                    this.ranktext = ranktext;

                    setTimeout(() => {
                        this.showCongratulations = true;
                    }, 1000);
                    // 5秒後停止煙火
                    setTimeout(() => {
                        fireworks.stop();
                        this.showfireworks = false;
                        this.showOverlay = false;
                        this.showCongratulations = false;
                    }, 10000);
                }
            });
        },
        closeCongratulations() {
            this.showCongratulations = false;
            this.showOverlay = false;
            this.showfireworks = false;
            if (this.$refs.fireworks) {
                this.$refs.fireworks.stop();
            }
        },
    },
};

</script>

<style>
    .overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.4);
    z-index: 1000;
  }

  .congratulations-modal {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: white;
    padding: 20px;
    border-radius: 10px;
    z-index: 1001;
    text-align: center;
  }

  .congratulations-modal h2 {
    margin-top: 0;
  }

  .congratulations-modal button {
    margin-top: 10px;
    padding: 10px 10px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  .congratulations-modal button:hover {
    background-color: #45a049;
  }

  .logo-container {
    display: auto;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }

  .logo {
    width: 80px;
    height: auto;
    transform: rotate(-15deg);
  }

</style>
<template>
    <view class="container">
        <view class="header">
            <text class="title">LED Marquee</text>
            <text class="subtitle">Electronic Display Simulator</text>
        </view>

        <view class="display-frame" @click="toggleFullscreen">
            <view class="display-screen" :class="{ fullscreen: isFullscreen }" :style="{ backgroundColor: bgColor }">
                <view class="display-scanline"></view>
                <view
                    class="led-text"
                    :class="{ paused: !isPlaying, fullscreen: isFullscreen }"
                    :style="isFullscreen ? ledTextFullscreenStyle : ledTextStyle"
                >{{ text }}</view>
                <view class="fullscreen-hint" v-if="!isFullscreen">点击全屏</view>
            </view>
        </view>

        <view class="control-panel" v-if="!isFullscreen">
            <view class="control-group">
                <text class="control-label">显示文字</text>
                <input
                    class="text-input"
                    v-model="text"
                    placeholder="输入滚动文字..."
                    @input="onTextChange"
                />
            </view>

            <view class="control-group">
                <text class="control-label">文字颜色</text>
                <view class="color-grid">
                    <view
                        v-for="color in textColors"
                        :key="color"
                        class="color-swatch"
                        :class="{ active: color === textColor }"
                        :style="{ backgroundColor: color }"
                        @click="selectTextColor(color)"
                    ></view>
                </view>
                <view class="custom-color">
                    <input type="color" :value="textColor" @input="onTextColorInput" />
                    <input
                        type="text"
                        class="color-hex"
                        :value="textColor.toUpperCase()"
                        @input="onTextColorHexInput"
                        maxlength="7"
                    />
                </view>
            </view>

            <view class="control-group">
                <text class="control-label">背景颜色</text>
                <view class="color-grid">
                    <view
                        v-for="color in bgColors"
                        :key="color"
                        class="color-swatch"
                        :class="{ active: color === bgColor }"
                        :style="{ backgroundColor: color }"
                        @click="selectBgColor(color)"
                    ></view>
                </view>
                <view class="custom-color">
                    <input type="color" :value="bgColor" @input="onBgColorInput" />
                    <input
                        type="text"
                        class="color-hex"
                        :value="bgColor.toUpperCase()"
                        @input="onBgColorHexInput"
                        maxlength="7"
                    />
                </view>
            </view>

            <view class="control-group">
                <text class="control-label">滚动速度</text>
                <view class="speed-control">
                    <slider
                        class="speed-slider"
                        :min="20"
                        :max="200"
                        :value="speed"
                        :block-size="20"
                        active-color="#00f0ff"
                        background-color="rgba(0,0,0,0.4)"
                        @change="onSpeedChange"
                    />
                    <text class="speed-value">{{ speed }} px/s</text>
                </view>
                <view class="speed-marks">
                    <text>慢</text>
                    <text>快</text>
                </view>
            </view>

            <view class="button-row">
                <button class="btn btn-primary" @click="togglePlay">
                    {{ isPlaying ? '暂停' : '播放' }}
                </button>
                <button class="btn btn-secondary" @click="resetSettings">重置</button>
            </view>
        </view>

        <view class="footer">设置自动保存</view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            text: '欢迎使用LED跑马灯',
            textColor: '#ff0040',
            bgColor: '#000000',
            speed: 80,
            isFullscreen: false,
            isPlaying: true,
            orientation: 0, // 0=portrait, 90=landscape
            textColors: ['#ff0040', '#ffee00', '#00ff40', '#00ffee', '#0088ff', '#8800ff', '#ffffff', '#ff8800'],
            bgColors: ['#000000', '#001833', '#180028', '#002800', '#280000', '#1a1a1a'],
            animationDuration: 0
        }
    },
    computed: {
        ledTextStyle() {
            return {
                color: this.textColor,
                textShadow: '0 0 10px ' + this.textColor + ', 0 0 20px ' + this.textColor + ', 0 0 40px ' + this.textColor + '80, 0 0 80px ' + this.textColor + '40',
                animationDuration: this.animationDuration + 's',
                animationPlayState: this.isPlaying ? 'running' : 'paused'
            }
        },
        ledTextFullscreenStyle() {
            const base = this.ledTextStyle
            if (this.isFullscreen) {
                return {
                    ...base,
                    transform: 'translateY(-50%) rotate(' + this.orientation + 'deg)',
                    animationDuration: (parseFloat(base.animationDuration) * 1.5) + 's'
                }
            }
            return base
        }
    },
    onLoad() {
        this.loadSettings()
    },
    onReady() {
        this.updateAnimationDuration()
    },
    methods: {
        onTextChange(e) {
            this.text = e.detail.value || ' '
            this.saveSettings()
        },
        selectTextColor(color) {
            this.textColor = color
            this.saveSettings()
        },
        onTextColorInput(e) {
            this.textColor = e.detail.value
            this.saveSettings()
        },
        onTextColorHexInput(e) {
            let value = e.detail.value
            if (!value.startsWith('#')) value = '#' + value
            if (/^#[0-9A-Fa-f]{6}$/.test(value)) {
                this.textColor = value
                this.saveSettings()
            }
        },
        selectBgColor(color) {
            this.bgColor = color
            this.saveSettings()
        },
        onBgColorInput(e) {
            this.bgColor = e.detail.value
            this.saveSettings()
        },
        onBgColorHexInput(e) {
            let value = e.detail.value
            if (!value.startsWith('#')) value = '#' + value
            if (/^#[0-9A-Fa-f]{6}$/.test(value)) {
                this.bgColor = value
                this.saveSettings()
            }
        },
        onSpeedChange(e) {
            this.speed = e.detail.value
            this.updateAnimationDuration()
            this.saveSettings()
        },
        updateAnimationDuration() {
            const screenWidth = 375
            this.animationDuration = screenWidth / this.speed
        },
        togglePlay() {
            this.isPlaying = !this.isPlaying
        },
        toggleFullscreen() {
            this.isFullscreen = !this.isFullscreen
            if (this.isFullscreen) {
                this.startAccelerometer()
            } else {
                this.stopAccelerometer()
                this.orientation = 0
            }
        },
        startAccelerometer() {
            uni.onAccelerometerChange((res) => {
                if (res.x > 0.5) {
                    this.orientation = 90
                } else if (res.x < -0.5) {
                    this.orientation = -90
                } else {
                    this.orientation = 0
                }
            })
        },
        stopAccelerometer() {
            // uni.offAccelerometerChange() // not available in all versions
        },
        resetSettings() {
            this.text = '欢迎使用LED跑马灯'
            this.textColor = '#ff0040'
            this.bgColor = '#000000'
            this.speed = 80
            this.isPlaying = true
            this.updateAnimationDuration()
            this.saveSettings()
        },
        saveSettings() {
            const settings = {
                text: this.text,
                textColor: this.textColor,
                bgColor: this.bgColor,
                speed: this.speed,
                isPlaying: this.isPlaying
            }
            try {
                uni.setStorageSync('ledMarqueeSettings', JSON.stringify(settings))
            } catch (e) {
                console.warn('Failed to save settings')
            }
        },
        loadSettings() {
            try {
                const saved = uni.getStorageSync('ledMarqueeSettings')
                if (saved) {
                    const settings = JSON.parse(saved)
                    this.text = settings.text || '欢迎使用LED跑马灯'
                    this.textColor = settings.textColor || '#ff0040'
                    this.bgColor = settings.bgColor || '#000000'
                    this.speed = settings.speed || 80
                    this.isPlaying = settings.isPlaying !== false
                    this.updateAnimationDuration()
                }
            } catch (e) {
                console.warn('Failed to load settings')
            }
        }
    }
}
</script>

<style>
page {
    background: #0a0a0f;
    min-height: 100vh;
}

.container {
    padding: 20px;
    max-width: 480px;
    margin: 0 auto;
}

.header {
    text-align: center;
    margin-bottom: 24px;
    padding: 16px;
    background: linear-gradient(180deg, rgba(0, 240, 255, 0.08) 0%, transparent 100%);
    border-bottom: 1px solid #2a2d35;
}

.title {
    display: block;
    font-size: 24px;
    font-weight: 600;
    letter-spacing: 4px;
    color: #00f0ff;
    text-shadow: 0 0 20px rgba(0, 240, 255, 0.5), 0 0 40px rgba(0, 240, 255, 0.2);
}

.subtitle {
    display: block;
    font-size: 10px;
    letter-spacing: 6px;
    color: #6a6d75;
    margin-top: 4px;
}

.display-frame {
    background: linear-gradient(145deg, #1a1c22 0%, #0d0e12 100%);
    border: 2px solid #2a2d35;
    border-radius: 12px;
    padding: 4px;
    margin-bottom: 24px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
}

.display-screen {
    position: relative;
    height: 160px;
    background: #000;
    border-radius: 8px;
    overflow: hidden;
}

.display-scanline {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: repeating-linear-gradient(
        0deg,
        transparent 0px,
        rgba(0, 0, 0, 0.1) 1px,
        transparent 2px
    );
    pointer-events: none;
    z-index: 10;
}

.led-text {
    position: absolute;
    top: 50%;
    left: 100%;
    transform: translateY(-50%);
    font-size: 48px;
    font-weight: 700;
    white-space: nowrap;
    will-change: transform;
    letter-spacing: 2px;
    animation: marquee linear infinite;
}

.led-text.paused {
    animation-play-state: paused;
}

@keyframes marquee {
    0% {
        transform: translateY(-50%) translateX(0);
    }
    100% {
        transform: translateY(-50%) translateX(-200%);
    }
}

.control-panel {
    background: #12141a;
    border: 1px solid #2a2d35;
    border-radius: 12px;
    padding: 20px;
}

.control-group {
    margin-bottom: 20px;
}

.control-label {
    display: block;
    font-size: 10px;
    letter-spacing: 3px;
    color: #6a6d75;
    margin-bottom: 8px;
    text-transform: uppercase;
}

.text-input {
    width: 100%;
    background: rgba(0, 0, 0, 0.4);
    border: 1px solid #2a2d35;
    border-radius: 8px;
    padding: 14px 16px;
    font-size: 16px;
    color: #e0e0e5;
    outline: none;
}

.text-input:focus {
    border-color: #00f0ff;
}

.color-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    margin-bottom: 10px;
}

.color-swatch {
    width: 36px;
    height: 36px;
    border-radius: 6px;
    cursor: pointer;
    border: 2px solid transparent;
}

.color-swatch.active {
    border-color: #fff;
    box-shadow: 0 0 10px currentColor;
}

.custom-color {
    display: flex;
    gap: 10px;
    align-items: center;
}

.custom-color input[type="color"] {
    width: 44px;
    height: 44px;
    border: none;
    border-radius: 8px;
    padding: 0;
}

.custom-color input[type="color"]::-webkit-color-swatch-wrapper {
    padding: 0;
}

.custom-color input[type="color"]::-webkit-color-swatch {
    border: 2px solid #2a2d35;
    border-radius: 8px;
}

.color-hex {
    flex: 1;
    background: rgba(0, 0, 0, 0.4);
    border: 1px solid #2a2d35;
    border-radius: 8px;
    padding: 10px 14px;
    font-size: 14px;
    color: #e0e0e5;
    font-family: monospace;
    letter-spacing: 1px;
    outline: none;
}

.speed-control {
    display: flex;
    align-items: center;
}

.speed-slider {
    flex: 1;
    margin: 0 10px;
}

.speed-value {
    font-size: 14px;
    font-family: monospace;
    color: #00f0ff;
    min-width: 70px;
    text-align: right;
}

.speed-marks {
    display: flex;
    justify-content: space-between;
    margin-top: 6px;
    font-size: 9px;
    color: #6a6d75;
}

.button-row {
    display: flex;
    gap: 12px;
    margin-top: 24px;
}

.btn {
    flex: 1;
    padding: 14px 20px;
    border: 1px solid #2a2d35;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 600;
    letter-spacing: 2px;
    cursor: pointer;
    text-transform: uppercase;
    background: transparent;
    color: #9a9da5;
}

.btn-primary {
    background: linear-gradient(145deg, rgba(0, 240, 255, 0.15), rgba(0, 240, 255, 0.05));
    color: #00f0ff;
    border-color: rgba(0, 240, 255, 0.3);
}

.btn-secondary:active {
    background: rgba(255, 255, 255, 0.08);
}

.footer {
    text-align: center;
    margin-top: 24px;
    padding: 12px;
    font-size: 10px;
    color: #6a6d75;
    letter-spacing: 2px;
}

.control-panel.hidden {
    display: none;
}


/* Fullscreen styles */
.display-screen.fullscreen {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    height: 100vh;
    width: 100vw;
    z-index: 9999;
    border-radius: 0;
}

.led-text.fullscreen {
    font-size: 72px;
    transition: transform 0.3s ease;
}

.fullscreen-hint {
    position: absolute;
    bottom: 10px;
    right: 10px;
    font-size: 10px;
    color: rgba(255, 255, 255, 0.3);
    z-index: 11;
}
</style>


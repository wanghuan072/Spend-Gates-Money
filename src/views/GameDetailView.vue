<template>
  <main class="game-detail-view" role="main">
    <!-- 头部组件 -->
    <Header />

    <!-- 游戏详情内容 -->
    <div class="game-content" v-if="game">
      <!-- 游戏标题区域 -->
      <section class="game-header" aria-labelledby="game-title">
        <div class="header-content">
          <h1 id="game-title" class="game-title">{{ game.title }}</h1>
          <p class="game-subtitle">Enjoy the gaming experience</p>
        </div>
      </section>

      <!-- 游戏iframe区域 -->
      <section class="game-iframe-section" aria-labelledby="game-iframe-title">
        <div class="iframe-container">
          <h2 id="game-iframe-title" class="iframe-title">Game Area</h2>
          <div class="iframe-wrapper">
            <!-- 蒙版 - 点击后加载iframe -->
            <div v-if="!iframeLoaded" class="iframe-overlay" @click="loadIframe">
              <!-- 背景图片 -->
              <div
                class="background-image"
                :style="{ backgroundImage: `url(${game.imageUrl})` }"
              ></div>
              <!-- 小图片 -->
              <div class="game-preview">
                <img :src="game.imageUrl" :alt="game.imageAlt" class="preview-image" />
              </div>
              <!-- 播放按钮 -->
              <button class="play-button" @click="loadIframe">
                <span class="play-icon">▶</span>
                PLAY
              </button>
            </div>
            <!-- iframe - 点击蒙版后显示 -->
            <iframe
              v-if="iframeLoaded"
              :src="game.iframeUrl"
              :title="game.title"
              class="game-iframe"
              frameborder="0"
              allowfullscreen
              loading="lazy"
            ></iframe>
          </div>
        </div>
      </section>

      <!-- 游戏详情HTML -->
      <section class="game-details-section" aria-labelledby="game-details-title">
        <div class="details-container">
          <h2 id="game-details-title" class="details-title">Game Introduction</h2>
          <div class="details-content v-html-content" v-html="game.detailsHtml"></div>
        </div>
      </section>

      <!-- 热门游戏推荐 -->
      <HotGames :exclude-address-bar="game.addressBar" @select="navigateToGame" />
    </div>

    <!-- 游戏不存在 -->
    <div class="game-not-found" v-else>
      <div class="not-found-content">
        <h1 class="not-found-title">Game Not Found</h1>
        <p class="not-found-description">Sorry, the game you are looking for does not exist</p>
        <button class="back-btn" @click="goBack">Back to Games</button>
      </div>
    </div>

    <!-- 底部Footer -->
    <Footer />
  </main>
</template>

<script setup>
import { computed, ref, watch, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import Header from '../components/Header.vue'
import HotGames from '../components/HotGames.vue'
import Footer from '../components/Footer.vue'
import { games } from '../data/games.js'
import { setGameDetailPageSEO } from '../utils/seo.js'
import { insertStructuredData, generateGameSchema } from '../utils/structuredData.js'

const route = useRoute()
const router = useRouter()

// iframe加载状态
const iframeLoaded = ref(false)

// 根据addressBar获取游戏
const getGameByAddressBar = (addressBar) => {
  return games.find((game) => game.addressBar === addressBar)
}

const game = computed(() => getGameByAddressBar(route.params.addressBar))

// 监听游戏变化，设置动态SEO和结构化数据
watch(
  game,
  (newGame) => {
    if (newGame) {
      // 设置游戏详情页SEO
      setGameDetailPageSEO(newGame)

      // 插入游戏结构化数据
      const gameSchema = generateGameSchema(newGame)
      insertStructuredData(gameSchema)
    }
  },
  { immediate: true }
)

// 组件挂载时设置SEO
onMounted(() => {
  if (game.value) {
    setGameDetailPageSEO(game.value)

    const gameSchema = generateGameSchema(game.value)
    insertStructuredData(gameSchema)
  }
})

// 加载iframe
const loadIframe = () => {
  iframeLoaded.value = true
}

const navigateToGame = (addressBar) => {
  router.push(`/games/${addressBar}`)
}

const goBack = () => {
  router.push('/games')
}
</script>

<style scoped>
@import '../styles/v-html-content.css';
.game-detail-view {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  padding: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  position: relative;
  overflow-x: hidden;
}

.game-detail-view::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at 20% 80%, rgba(120, 119, 198, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 80% 20%, rgba(255, 119, 198, 0.3) 0%, transparent 50%),
    radial-gradient(circle at 40% 40%, rgba(120, 219, 255, 0.3) 0%, transparent 50%);
  pointer-events: none;
}

/* 游戏头部 */
.game-header {
  padding: 40px 24px 20px;
  text-align: center;
  position: relative;
  z-index: 1;
}

.header-content {
  max-width: 800px;
  margin: 0 auto;
}

.game-title {
  font-size: 36px;
  font-weight: 900;
  margin: 0 0 8px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.game-subtitle {
  font-size: 16px;
  color: #4a5568;
  margin: 0;
  font-weight: 500;
  opacity: 0.8;
}

/* 游戏iframe区域 */
.game-iframe-section {
  padding: 20px 24px;
  position: relative;
  z-index: 1;
}

.iframe-container {
  max-width: 1200px;
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  padding: 24px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.iframe-title {
  font-size: 24px;
  font-weight: 700;
  color: #2d3748;
  margin: 0 0 20px;
  text-align: center;
}

.iframe-wrapper {
  position: relative;
  width: 100%;
  height: 600px;
  border-radius: 12px;
  overflow: hidden;
  background: #f7fafc;
  border: 1px solid rgba(160, 174, 192, 0.2);
}

/* iframe蒙版样式 */
.iframe-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  overflow: hidden;
  gap: 24px;
}

/* 背景图片 */
.background-image {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  filter: blur(8px);
  transform: scale(1.1);
}

/* 小图片样式 */
.game-preview {
  position: relative;
  z-index: 2;
}

.preview-image {
  width: 160px;
  height: 160px;
  border-radius: 24px;
  object-fit: cover;
  border: 4px solid rgba(255, 255, 255, 0.4);
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.3);
}

.play-button {
  position: relative;
  z-index: 2;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 16px 32px;
  background: linear-gradient(135deg, #ff6b9d, #ff8e53);
  color: white;
  border: none;
  border-radius: 16px;
  font-weight: 700;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 24px rgba(255, 107, 157, 0.4);
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.play-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 32px rgba(255, 107, 157, 0.5);
  background: linear-gradient(135deg, #ff5a8a, #ff7a3d);
}

.play-icon {
  font-size: 16px;
}

.game-iframe {
  width: 100%;
  height: 100%;
  border: none;
}

/* 游戏详情区域 */
.game-details-section {
  padding: 40px 24px;
  position: relative;
  z-index: 1;
}

.details-container {
  max-width: 800px;
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  padding: 32px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.details-title {
  font-size: 28px;
  font-weight: 800;
  color: #2d3748;
  margin: 0 0 24px;
  text-align: center;
}

/* 游戏未找到 */
.game-not-found {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  position: relative;
  z-index: 1;
}

.not-found-content {
  text-align: center;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  padding: 60px 40px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.not-found-title {
  font-size: 32px;
  font-weight: 800;
  color: #2d3748;
  margin: 0 0 16px;
}

.not-found-description {
  font-size: 18px;
  color: #718096;
  margin: 0 0 32px;
}

.back-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border: none;
  border-radius: 12px;
  font-weight: 600;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.2);
}

.back-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
}

/* 响应式设计 */
@media (max-width: 1024px) {
  .game-header {
    padding: 30px 16px 16px;
  }

  .game-title {
    font-size: 28px;
  }

  .game-subtitle {
    font-size: 14px;
  }

  .game-iframe-section {
    padding: 16px;
  }

  .iframe-container {
    padding: 20px;
  }

  .iframe-title {
    font-size: 20px;
  }

  .iframe-wrapper {
    height: 500px;
  }

  .play-button {
    padding: 14px 28px;
    font-size: 16px;
  }

  .preview-image {
    width: 140px;
    height: 140px;
  }

  .iframe-overlay {
    gap: 20px;
  }

  .game-details-section {
    padding: 30px 16px;
  }

  .details-container {
    padding: 24px;
  }

  .details-title {
    font-size: 24px;
  }
}

@media (max-width: 768px) {
  .game-header {
    padding: 20px 10px 12px;
  }

  .game-title {
    font-size: 20px;
  }

  .game-subtitle {
    font-size: 12px;
  }

  .game-iframe-section {
    padding: 10px;
  }

  .iframe-container {
    padding: 12px;
  }

  .iframe-title {
    font-size: 16px;
    margin-bottom: 12px;
  }

  .iframe-wrapper {
    height: 400px;
  }

  .play-button {
    padding: 12px 24px;
    font-size: 14px;
  }

  .preview-image {
    width: 120px;
    height: 120px;
  }

  .iframe-overlay {
    gap: 16px;
  }

  .game-details-section {
    padding: 20px 10px;
  }

  .details-container {
    padding: 16px;
  }

  .details-title {
    font-size: 20px;
    margin-bottom: 16px;
  }

  .not-found-content {
    padding: 40px 20px;
  }

  .not-found-title {
    font-size: 24px;
  }

  .not-found-description {
    font-size: 14px;
  }

  .back-btn {
    padding: 10px 20px;
    font-size: 14px;
  }
}
</style> 
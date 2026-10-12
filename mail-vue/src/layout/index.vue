<template>
  <div class="apple-shell">

    <!-- ============ 全局导航（纯黑 44px）：徽章 + 站点名在最左 ============ -->
    <header class="global-nav">
      <div class="gn-left">
        <button class="gn-burger" type="button" @click="toggleAside">
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <line x1="3.5" y1="7" x2="20.5" y2="7"></line>
            <line x1="3.5" y1="12" x2="20.5" y2="12"></line>
            <line x1="3.5" y1="17" x2="20.5" y2="17"></line>
          </svg>
        </button>

        <!-- 徽章 + 字标：徽章固定在最左侧 -->
        <div class="gn-mark">
          <img class="gn-badge" :src="BADGE_SRC" alt="badge" draggable="false"/>
          <span v-if="SHOW_WORDMARK" class="gn-wordmark">{{ siteTitle }}</span>
        </div>

        <nav class="gn-quiet">
          <a class="gn-link" :class="{ 'is-active': route.name === 'email' }" @click="go('email')">{{ $t('inbox') }}</a>
          <a v-perm="'email:send'" class="gn-link" :class="{ 'is-active': route.name === 'send' }" @click="go('send')">{{ $t('sent') }}</a>
          <a class="gn-link" :class="{ 'is-active': route.name === 'star' }" @click="go('star')">{{ $t('starred') }}</a>
          <a class="gn-link" :class="{ 'is-active': route.name === 'setting' }" @click="go('setting')">{{ $t('settings') }}</a>
        </nav>
      </div>

      <div class="gn-right">
        <Header/>
      </div>
    </header>

    <!-- ============ 次级导航（毛玻璃 52px）：页面标题 + 写邮件 ============ -->
    <div class="subnav">
      <div class="subnav-title">
        <span>{{ pageTitle }}</span>
      </div>
      <div class="subnav-right">
        <button v-perm="'email:send'" class="btn-pill" type="button" @click="openSend">
          <svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <path d="M4 20h4l10.5-10.5a2.1 2.1 0 0 0-3-3L5 17v3z" stroke-width="1.7" stroke-linecap="round"
                  stroke-linejoin="round"></path>
          </svg>
          {{ $t('writeMail') }}
        </button>
      </div>
    </div>

    <el-container class="layout">
      <el-aside class="aside" :class="{ 'aside-show': uiStore.asideShow }">
        <Aside/>
      </el-aside>
      <div v-if="isNarrow && uiStore.asideShow" class="overlay-show"
           @click="uiStore.asideShow = false"></div>
      <el-container class="main-container">
        <el-main>
          <Main/>
        </el-main>
      </el-container>
    </el-container>
  </div>
  <writer ref="writerRef" />
</template>

<script setup>
import Aside from '@/layout/aside/index.vue'
import Header from '@/layout/header/index.vue'
import Main from '@/layout/main/index.vue'
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import {useUiStore} from "@/store/ui.js";
import {useSettingStore} from "@/store/setting.js";
import {useRoute} from "vue-router";
import router from "@/router";
import {useI18n} from "vue-i18n";
import writer from '@/layout/write/index.vue'

/* ------------------------------------------------------------------
 * 顶栏徽章配置
 * BADGE_SRC      : 徽章文件放在 mail-vue/public/ 下，这里写 /文件名
 *                  · /huizhang-light.svg  深绿提亮版，配纯黑顶栏最清楚（默认）
 *                  · /huizhang.svg        原色徽章
 *                  · /huizhang-dark.svg   白底用（白色字标改成墨色）
 * SHOW_WORDMARK  : 徽章右侧是否再显示站点名（站点名来自后台“个性化设置-网站标题”）
 * ------------------------------------------------------------------ */
const BADGE_SRC = '/huizhang-light.svg'
const SHOW_WORDMARK = true

const uiStore = useUiStore();
const settingStore = useSettingStore();
const route = useRoute();
const { t } = useI18n();
const writerRef = ref({})
const isNarrow = ref(window.innerWidth < 1025)

const siteTitle = computed(() => settingStore.settings.title || 'Yifly Mail')
const pageTitle = computed(() => route.meta.title ? t(route.meta.title) : siteTitle.value)

const handleResize = () => {
  isNarrow.value = window.innerWidth < 1025
  uiStore.asideShow = window.innerWidth > 1024;
}

onMounted(() => {
  uiStore.writerRef = writerRef

  window.addEventListener('resize', handleResize)
  handleResize()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
})

function toggleAside() {
  uiStore.asideShow = !uiStore.asideShow
}

function openSend() {
  uiStore.writerRef?.open()
}

function go(name) {
  if (route.name === name) return
  if (router.hasRoute(name) || ['email', 'star', 'setting'].includes(name)) {
    router.push({ name })
  }
}
</script>

<style lang="scss" scoped>
/* ---------- 外壳：一列三段（导航 / 次级导航 / 主体） ---------- */
.apple-shell {
  position: fixed;
  inset: 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: var(--ym-canvas);
}

/* ---------- 全局导航：纯黑，整站唯一的纯黑面 ---------- */
.global-nav {
  flex: 0 0 auto;
  height: var(--ym-nav-h);
  background: var(--ym-black);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 24px;
  padding: 0 20px;
  position: relative;
  z-index: 60;
}

.gn-left,
.gn-right {
  display: flex;
  align-items: center;
  gap: 18px;
  min-width: 0;
}

.gn-right {
  height: 100%;
}

.gn-burger {
  display: grid;
  place-items: center;
  width: 28px;
  height: 28px;
  border-radius: var(--ym-r-sm);
  color: #fff;
  cursor: pointer;
  transition: background 0.18s ease;

  svg {
    width: 17px;
    height: 17px;
  }

  svg line {
    stroke: currentColor;
    stroke-width: 1.6;
    stroke-linecap: round;
  }

  &:hover {
    background: rgba(255, 255, 255, 0.12);
  }
}

/* 徽章 + 字标 */
.gn-mark {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 0;
}

.gn-badge {
  height: 26px;
  width: auto;
  max-width: 190px;
  object-fit: contain;
  flex: 0 0 auto;
  user-select: none;
}

.gn-wordmark {
  font-family: var(--ym-font-display);
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.02em;
  color: #fff;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 导航链接（只在宽屏显示） */
.gn-quiet {
  display: flex;
  align-items: center;
  gap: 22px;
  padding-left: 18px;
  margin-left: 4px;
  border-left: 1px solid rgba(255, 255, 255, 0.14);
  height: 20px;
}

.gn-link {
  font-size: 12px;
  font-weight: 400;
  line-height: 1;
  letter-spacing: -0.12px;
  color: rgba(255, 255, 255, 0.82);
  cursor: pointer;
  white-space: nowrap;
  transition: color 0.18s ease;

  &:hover,
  &.is-active {
    color: #fff;
  }
}

@media (max-width: 1000px) {
  .gn-quiet {
    display: none;
  }
}

@media (max-width: 640px) {
  .gn-wordmark {
    display: none;
  }

  .gn-badge {
    max-width: 150px;
  }

  .global-nav {
    padding: 0 14px;
  }
}

/* ---------- 次级导航：毛玻璃，标题 + 主行动按钮 ---------- */
.subnav {
  flex: 0 0 auto;
  height: var(--ym-subnav-h);
  background: rgba(245, 245, 247, 0.8);
  -webkit-backdrop-filter: saturate(180%) blur(20px);
  backdrop-filter: saturate(180%) blur(20px);
  border-bottom: 1px solid var(--ym-hairline-soft);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  position: relative;
  z-index: 50;
}

.subnav-title {
  font-family: var(--ym-font-display);
  font-size: 21px;
  font-weight: 600;
  line-height: 1.19;
  letter-spacing: 0.231px;
  color: var(--ym-ink);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.subnav-right {
  display: flex;
  align-items: center;
  gap: 14px;
}

.btn-pill {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  background: var(--ym-primary);
  color: #fff;
  font-size: 15px;
  font-weight: 400;
  letter-spacing: -0.2px;
  border-radius: var(--ym-r-pill);
  padding: 9px 20px;
  white-space: nowrap;
  cursor: pointer;
  transition: transform 0.12s ease, background 0.18s ease;

  svg {
    width: 14px;
    height: 14px;
  }

  svg path {
    stroke: #fff;
  }

  &:hover {
    background: var(--ym-primary-focus);
  }

  &:active {
    transform: scale(0.95);
  }
}

html.dark .subnav {
  background: rgba(29, 29, 31, 0.82);
}

@media (max-width: 640px) {
  .subnav {
    padding: 0 14px;
  }

  .subnav-title {
    font-size: 17px;
    letter-spacing: -0.374px;
  }

  .btn-pill {
    padding: 8px 16px;
    font-size: 14px;
  }
}

/* ---------- 主体：侧栏 + 内容 ---------- */
.layout {
  flex: 1 1 auto;
  min-height: 0;
  width: 100%;
  overflow: hidden;
}

.el-aside {
  width: var(--ym-aside-w);
  flex: 0 0 var(--ym-aside-w);
  height: 100%;
  overflow: hidden;
  background: var(--ym-parchment);
}

.main-container {
  min-width: 0;
  height: 100%;
  background: var(--ym-canvas);
  overflow: hidden;
}

.el-main {
  padding: 0;
  height: 100%;
  overflow: hidden;
}

/* 窄屏：侧栏变抽屉 */
@media (max-width: 1024px) {
  .el-aside {
    position: fixed;
    top: calc(var(--ym-nav-h) + var(--ym-subnav-h));
    bottom: 0;
    left: 0;
    height: auto;
    z-index: 101;
    transform: translateX(-101%);
    transition: transform 0.3s cubic-bezier(0.32, 0.72, 0, 1);
  }

  .el-aside.aside-show {
    transform: none;
    box-shadow: 0 0 40px rgba(0, 0, 0, 0.2);
  }
}

.overlay-show {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
  z-index: 99;
}
</style>

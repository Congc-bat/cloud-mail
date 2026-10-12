<template>
  <!-- 全局导航右侧工具区（主题 / 公告 / 账号） -->
  <div class="toolbar">
    <div v-if="uiStore.dark" class="sun-icon icon-item" @click="openDark($event)">
      <Icon icon="mingcute:sun-fill"/>
    </div>
    <div v-else class="dark-icon icon-item" @click="openDark($event)">
      <Icon icon="solar:moon-linear"/>
    </div>
    <div class="notice icon-item" @click="openNotice">
      <Icon icon="streamline-plump:announcement-megaphone"/>
    </div>
    <el-dropdown ref="userinfoRef" @visible-change="e => userInfoShow = e" :teleported="false"
                 popper-class="detail-dropdown">
      <div class="avatar" @click="userInfoHide">
        <div class="avatar-text">
          <div>{{ formatName(userStore.user.email) }}</div>
        </div>
        <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
      </div>
      <template #dropdown>
        <div class="user-details">
          <div class="details-avatar">
            {{ formatName(userStore.user.email) }}
          </div>
          <div class="user-name">
            {{ userStore.user.name }}
          </div>
          <div class="detail-email" @click="copyEmail(userStore.user.email)">
            {{ userStore.user.email }}
          </div>
          <div class="detail-user-type">
            <el-tag>{{ userStore.user.role.name }}</el-tag>
          </div>
          <div class="action-info">
            <div>
              <span style="margin-right: 10px">{{ $t('sendCount') }}</span>
              <span style="margin-right: 10px">{{ $t('accountCount') }}</span>
            </div>
            <div>
              <div>
                <span v-if="sendCount" style="margin-right: 5px">{{ sendCount }}</span>
                <el-tag v-if="!hasPerm('email:send')">{{ sendType }}</el-tag>
                <el-tag v-else>{{ sendType }}</el-tag>
              </div>
              <div>
                <el-tag v-if="settingStore.settings.manyEmail || settingStore.settings.addEmail">
                  {{ $t('disabled') }}
                </el-tag>
                <span v-else-if="accountCount && hasPerm('account:add')"
                      style="margin-right: 5px">{{ $t('totalUserAccount', {msg: accountCount}) }}</span>
                <el-tag v-else-if="!accountCount && hasPerm('account:add')">{{ $t('unlimited') }}</el-tag>
                <el-tag v-else-if="!hasPerm('account:add')">{{ $t('unauthorized') }}</el-tag>
              </div>
            </div>
          </div>
          <div class="logout">
            <el-button type="primary" :loading="logoutLoading" @click="clickLogout">{{ $t('logOut') }}</el-button>
          </div>
        </div>
      </template>
    </el-dropdown>
  </div>
</template>

<script setup>
import router from "@/router";
import {logout} from "@/request/login.js";
import {Icon} from "@iconify/vue";
import {useUiStore} from "@/store/ui.js";
import {useUserStore} from "@/store/user.js";
import {computed, ref} from "vue";
import {useSettingStore} from "@/store/setting.js";
import {hasPerm} from "@/perm/perm.js"
import {useI18n} from "vue-i18n";

const {t} = useI18n();
const settingStore = useSettingStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const logoutLoading = ref(false)
const userInfoShow = ref(false)
const userinfoRef = ref({})

const accountCount = computed(() => {
  return userStore.user.role.accountCount
})

const sendType = computed(() => {

  if (settingStore.settings.send === 1) {
    return t('disabled')
  }

  if (!hasPerm('email:send')) {
    return t('unauthorized')
  }

  if (userStore.user.role.sendType === 'ban') {
    return t('sendBanned')
  }

  if (userStore.user.role.sendType === 'internal') {
    return t('sendInternal')
  }

  if (!userStore.user.role.sendCount) {
    return t('unlimited')
  }

  if (userStore.user.role.sendType === 'day') {
    return t('daily')
  }

  if (userStore.user.role.sendType === 'count') {
    return t('total')
  }
})

const sendCount = computed(() => {


  if (!hasPerm('email:send')) {
    return null
  }

  if (userStore.user.role.sendType === 'ban') {
    return null
  }

  if (userStore.user.role.sendType === 'internal') {
    return null
  }

  if (!userStore.user.role.sendCount) {
    return null
  }

  if (settingStore.settings.send === 1) {
    return null
  }

  return userStore.user.sendCount + '/' + userStore.user.role.sendCount
})

function userInfoHide(e) {
    if (userInfoShow.value) {
        userinfoRef.value.handleClose()
    } else {
        userinfoRef.value.handleOpen()
    }
}

async function copyEmail(email) {
  try {
    await navigator.clipboard.writeText(email);
    ElMessage({
      message: t('copySuccessMsg'),
      type: 'success',
      plain: true,
    })
  } catch (err) {
    console.error(`${t('copyFailMsg')}:`, err);
    ElMessage({
      message: t('copyFailMsg'),
      type: 'error',
      plain: true,
    })
  }
}

function openNotice() {
  uiStore.showNotice()
}

function openDark(e) {

  const nextIsDark = !uiStore.dark
  const root = document.documentElement

  if (!document.startViewTransition) {
    switchDark(nextIsDark, root);
    return
  }

  const x = e.clientX
  const y = e.clientY

  const maxX = Math.max(x, window.innerWidth - x)
  const maxY = Math.max(y, window.innerHeight - y)
  const endRadius = Math.hypot(maxX, maxY)

  // 标记切换目标，供 CSS 选择器使用
  root.setAttribute('data-theme-to', nextIsDark ? 'dark' : 'light')
  root.style.setProperty('--vt-x', `${x}px`)
  root.style.setProperty('--vt-y', `${y}px`)
  root.style.setProperty('--vt-end-radius', `${endRadius + 10}px`)

  const transition = document.startViewTransition(() => {
    switchDark(nextIsDark, root);
  })

  transition.finished.finally(() => {
    // 清理标记
    root.removeAttribute('data-theme-to')
  })
}

function switchDark(nextIsDark, root) {
  root.setAttribute('class', nextIsDark ? 'dark' : '')
  const metaTag = document.getElementById('theme-color-meta');
  // 顶部导航始终是纯黑，状态栏颜色保持一致
  metaTag.setAttribute('content', '#000000');
  uiStore.dark = nextIsDark
}

function clickLogout() {
  logoutLoading.value = true
  logout().then(() => {
    localStorage.removeItem("token")
    router.replace('/login')
  }).finally(() => {
    logoutLoading.value = false
  })
}

function formatName(email) {
  return email[0]?.toUpperCase() || ''
}

</script>
<style>
.detail-dropdown {
  color: var(--el-text-color-primary) !important;
  border-radius: var(--ym-r-lg);
  padding: 6px;
}
</style>
<style lang="scss" scoped>

:deep(.el-popper.is-pure) {
  border-radius: var(--ym-r-lg);
}

.user-details {
  width: 250px;
  font-size: 14px;
  display: grid;
  grid-template-columns: 1fr;
  justify-items: center;
  color: var(--el-text-color-primary);

  .user-name {
    font-weight: bold;
    margin-top: 10px;
    padding-left: 20px;
    padding-right: 20px;
    width: 250px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    text-align: center;
  }

  .detail-user-type {
    margin-top: 10px;
  }

  .action-info {
    width: 100%;
    display: grid;
    grid-template-columns: auto auto;
    margin-top: 10px;

    > div:first-child {
      display: grid;
      align-items: center;
      gap: 10px;
    }

    > div:last-child {
      display: grid;
      gap: 10px;
      text-align: center;

      > div {
        display: flex;
        align-items: center;
      }
    }
  }

  .detail-email {
    padding-left: 20px;
    padding-right: 20px;
    width: 250px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    text-align: center;
    color: var(--regular-text-color);
    cursor: pointer;
  }

  .logout {
    margin-top: 20px;
    width: 100%;
    padding-left: 10px;
    padding-right: 10px;
    padding-bottom: 10px;

    .el-button {
      border-radius: 6px;
      height: 28px;
      width: 100%;
    }
  }

  .details-avatar {
    margin-top: 20px;
    height: 40px;
    width: 40px;
    background: var(--el-bg-color);
    color: var(--el-text-color-primary);
    border: 1px solid var(--dark-border);
    font-size: 18px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
  }
}


/* ---------- 全局导航右侧工具区 ---------- */
.toolbar {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 8px;
  height: 100%;

  .icon-item {
    align-self: center;
    width: 28px;
    height: 28px;
    border-radius: var(--ym-r-sm);
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    color: rgba(255, 255, 255, 0.85);
    transition: background 0.18s ease, color 0.18s ease;
  }

  .icon-item:hover {
    background: rgba(255, 255, 255, 0.12);
    color: #ffffff;
  }

  .notice {
    font-size: 20px;
  }

  .dark-icon {
    font-size: 18px;
  }

  .sun-icon {
    font-size: 20px;
  }

  .avatar {
    display: flex;
    align-items: center;
    cursor: pointer;
    gap: 1px;
    margin-left: 6px;
    padding-left: 14px;
    border-left: 1px solid rgba(255, 255, 255, 0.14);

    .avatar-text {
      background: linear-gradient(0deg, #3a3a3d, #4a4a4e);
      color: #ffffff;
      height: 26px;
      width: 26px;
      font-size: 11px;
      font-weight: 600;
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 50%;
      user-select: none;
    }

    .setting-icon {
      position: relative;
      color: rgba(255, 255, 255, 0.7);
    }
  }

}

.el-tooltip__trigger:first-child:focus-visible {
  outline: unset;
}
</style>

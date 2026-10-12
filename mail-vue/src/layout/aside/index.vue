<template>
  <el-scrollbar class="scroll">
    <div class="sidebar">

      <!-- 邮箱 -->
      <div class="sb-group">
        <div class="sb-item" :class="{'is-active': route.meta.name === 'email'}" @click="go('email')">
          <Icon class="sb-ico" icon="hugeicons:mailbox-01" width="18" height="18"/>
          <span class="sb-name">{{ $t('inbox') }}</span>
        </div>
        <div v-perm="'email:send'" class="sb-item" :class="{'is-active': route.meta.name === 'send'}"
             @click="go('send')">
          <Icon class="sb-ico" icon="cil:send" width="17" height="17"/>
          <span class="sb-name">{{ $t('sent') }}</span>
        </div>
        <div v-perm="'email:send'" class="sb-item" :class="{'is-active': route.meta.name === 'draft'}"
             @click="go('draft')">
          <Icon class="sb-ico" icon="ep:document" width="17" height="17"/>
          <span class="sb-name">{{ $t('drafts') }}</span>
        </div>
        <div class="sb-item" :class="{'is-active': route.meta.name === 'star'}" @click="go('star')">
          <Icon class="sb-ico" icon="solar:star-line-duotone" width="17" height="17"/>
          <span class="sb-name">{{ $t('starred') }}</span>
        </div>
        <div class="sb-item" :class="{'is-active': route.meta.name === 'setting'}" @click="go('setting')">
          <Icon class="sb-ico" icon="fluent:settings-48-regular" width="17" height="17"/>
          <span class="sb-name">{{ $t('settings') }}</span>
        </div>
      </div>

      <!-- 管理 -->
      <div class="sb-group"
           v-perm="['all-email:query','user:query','role:query','setting:query','analysis:query','reg-key:query']">
        <div class="sb-label">{{ $t('manage') }}</div>
        <div v-perm="'analysis:query'" class="sb-item" :class="{'is-active': route.meta.name === 'analysis'}"
             @click="go('analysis')">
          <Icon class="sb-ico" icon="fluent:data-pie-20-regular" width="18" height="18"/>
          <span class="sb-name">{{ $t('analytics') }}</span>
        </div>
        <div v-perm="'user:query'" class="sb-item" :class="{'is-active': route.meta.name === 'user'}"
             @click="go('user')">
          <Icon class="sb-ico" icon="si:user-alt-2-line" width="17" height="17"/>
          <span class="sb-name">{{ $t('allUsers') }}</span>
        </div>
        <div v-perm="'all-email:query'" class="sb-item" :class="{'is-active': route.meta.name === 'all-email'}"
             @click="go('all-email')">
          <Icon class="sb-ico" icon="fluent:mail-list-28-regular" width="18" height="18"/>
          <span class="sb-name">{{ $t('allMail') }}</span>
        </div>
        <div v-perm="'role:query'" class="sb-item" :class="{'is-active': route.meta.name === 'role'}"
             @click="go('role')">
          <Icon class="sb-ico" icon="fluent:lock-closed-16-regular" width="17" height="17"/>
          <span class="sb-name">{{ $t('permissions') }}</span>
        </div>
        <div v-perm="'reg-key:query'" class="sb-item" :class="{'is-active': route.meta.name === 'reg-key'}"
             @click="go('reg-key')">
          <Icon class="sb-ico" icon="fluent:fingerprint-20-filled" width="18" height="18"/>
          <span class="sb-name">{{ $t('inviteCode') }}</span>
        </div>
        <div v-perm="'setting:query'" class="sb-item" :class="{'is-active': route.meta.name === 'sys-setting'}"
             @click="go('sys-setting')">
          <Icon class="sb-ico" icon="eos-icons:system-ok-outlined" width="16" height="16"/>
          <span class="sb-name">{{ $t('SystemSettings') }}</span>
        </div>
      </div>

      <!-- 当前账户卡片 -->
      <div class="sb-card" v-if="userStore.user?.email">
        <div class="sb-card-head">
          <span class="sb-card-title">{{ $t('account') }}</span>
          <span class="sb-card-role">{{ userStore.user.role?.name }}</span>
        </div>
        <div class="sb-card-mail" :title="userStore.user.email">{{ userStore.user.email }}</div>
        <div class="sb-card-note" @click="copyEmail">{{ $t('copy') }}</div>
      </div>
    </div>
  </el-scrollbar>
</template>

<script setup>
import router from "@/router/index.js";
import { useRoute } from "vue-router";
import {Icon} from "@iconify/vue";
import {useUserStore} from "@/store/user.js";
import {useI18n} from "vue-i18n";

const route = useRoute();
const userStore = useUserStore();
const { t } = useI18n();

function go(name) {
  if (route.name === name) return
  router.push({ name })
}

async function copyEmail() {
  try {
    await navigator.clipboard.writeText(userStore.user.email);
    ElMessage({message: t('copySuccessMsg'), type: 'success', plain: true})
  } catch (e) {
    ElMessage({message: t('copyFailMsg'), type: 'error', plain: true})
  }
}
</script>

<style lang="scss" scoped>

/* ---------- 侧栏容器：羊皮纸面 ---------- */
.scroll {
  height: 100%;
  background: var(--ym-parchment);
}

:deep(.el-scrollbar__wrap--hidden-default),
:deep(.el-scrollbar__view) {
  background: transparent;
}

/* 让内容撑满可视高度，底部账户卡片才能靠下 */
:deep(.el-scrollbar__view) {
  height: 100%;
}

.sidebar {
  min-height: 100%;
  display: flex;
  flex-direction: column;
  gap: 22px;
  padding: 24px 14px;
  background: var(--ym-parchment);
}

.sb-group {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sb-label {
  font-size: 12px;
  font-weight: 600;
  letter-spacing: -0.12px;
  color: var(--ym-ink-48);
  padding: 0 10px 8px;
  user-select: none;
}

/* ---------- 菜单项 ---------- */
.sb-item {
  display: flex;
  align-items: center;
  gap: 11px;
  width: 100%;
  padding: 9px 10px;
  border-radius: var(--ym-r-sm);
  font-size: 14px;
  line-height: 1.3;
  letter-spacing: -0.2px;
  color: var(--ym-ink);
  cursor: pointer;
  user-select: none;
  transition: background 0.16s ease, color 0.16s ease;

  &:hover {
    background: rgba(0, 0, 0, 0.045);
  }

  &.is-active {
    background: var(--aside-menu-active-background);
    color: var(--ym-primary);
    font-weight: 600;
  }
}

html.dark .sb-item {
  &:hover {
    background: rgba(255, 255, 255, 0.06);
  }
}

.sb-ico {
  flex: 0 0 auto;
  color: var(--ym-ink-80);
  transition: color 0.16s ease;
}

.sb-item.is-active .sb-ico {
  color: var(--ym-primary);
}

.sb-name {
  flex: 1;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

/* ---------- 底部账户卡片 ---------- */
.sb-card {
  margin-top: auto;
  background: var(--ym-canvas);
  border: 1px solid var(--ym-hairline);
  border-radius: var(--ym-r-lg);
  padding: 16px 17px;
}

.sb-card-head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 8px;
  margin-bottom: 10px;
}

.sb-card-title {
  font-size: 13px;
  font-weight: 600;
  letter-spacing: -0.2px;
  color: var(--ym-ink);
}

.sb-card-role {
  font-size: 11px;
  letter-spacing: -0.08px;
  color: var(--ym-primary);
  background: rgba(0, 102, 204, 0.08);
  border: 1px solid rgba(0, 102, 204, 0.16);
  border-radius: var(--ym-r-pill);
  padding: 2px 8px;
  white-space: nowrap;
  max-width: 96px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sb-card-mail {
  font-size: 13px;
  letter-spacing: -0.16px;
  color: var(--ym-ink-80);
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  user-select: text;
}

.sb-card-note {
  margin-top: 10px;
  font-size: 12px;
  letter-spacing: -0.12px;
  color: var(--ym-primary);
  cursor: pointer;
  width: fit-content;
}

.sb-card-note:hover {
  color: var(--ym-primary-focus);
}
</style>

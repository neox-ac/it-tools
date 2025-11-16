<script lang="ts" setup>
import { NIcon, useThemeVars } from 'naive-ui';

import { RouterLink } from 'vue-router';
import { Heart, Home2, Menu2 } from '@vicons/tabler';

import { storeToRefs } from 'pinia';
import MenuLayout from '../components/MenuLayout.vue';
import NavbarButtons from '../components/NavbarButtons.vue';
import { useStyleStore } from '@/stores/style.store';
import { config } from '@/config';
import type { ToolCategory } from '@/tools/tools.types';
import { useToolStore } from '@/tools/tools.store';
import { useTracker } from '@/modules/tracker/tracker.services';
import CollapsibleToolMenu from '@/components/CollapsibleToolMenu.vue';

const themeVars = useThemeVars();
const styleStore = useStyleStore();
const version = config.app.version;
const commitSha = config.app.lastCommitSha.slice(0, 7);

const { tracker } = useTracker();
const { t } = useI18n();

const toolStore = useToolStore();
const { favoriteTools, toolsByCategory } = storeToRefs(toolStore);

const tools = computed<ToolCategory[]>(() => [
  ...(favoriteTools.value.length > 0 ? [{ name: t('tools.categories.favorite-tools'), components: favoriteTools.value }] : []),
  ...toolsByCategory.value,
]);
</script>

<template>
  <MenuLayout class="menu-layout" :class="{ isSmallScreen: styleStore.isSmallScreen }">
    <template #sider>
      <div class="sider-content">
        <div class="menu-and-selector-wrapper">
          <div v-if="styleStore.isSmallScreen" flex flex-col items-center>
            <locale-selector w="90%" />
            <div flex justify-center>
              <NavbarButtons />
            </div>
          </div>

          <CollapsibleToolMenu :tools-by-category="tools" />
        </div>

        <div class="footer">
          <div>
            IT-Tools

            <c-link target="_blank" rel="noopener" :href="`https://github.com/CorentinTh/it-tools/tree/v${version}`">
              v{{ version }}
            </c-link>

            <template v-if="commitSha && commitSha.length > 0">
              -
              <c-link
                target="_blank"
                rel="noopener"
                type="primary"
                :href="`https://github.com/CorentinTh/it-tools/tree/${commitSha}`"
              >
                {{ commitSha }}
              </c-link>
            </template>
          </div>
          <div>
            © {{ new Date().getFullYear() }}
            <c-link target="_blank" rel="noopener" href="https://corentin.tech?utm_source=it-tools&utm_medium=footer">
              Corentin Thomasset
            </c-link>
          </div>
        </div>
      </div>
    </template>

    <template #content>
      <div class="content-wrapper">
        <div class="navbar-wrapper">
          <div flex items-center justify-center gap-2>
            <c-button
              circle
              variant="text"
              :aria-label="$t('home.toggleMenu')"
              @click="styleStore.isMenuCollapsed = !styleStore.isMenuCollapsed"
            >
              <NIcon size="25" :component="Menu2" />
            </c-button>

            <c-tooltip :tooltip="$t('home.home')" position="bottom">
              <c-button to="/" circle variant="text" :aria-label="$t('home.home')">
                <NIcon size="25" :component="Home2" />
              </c-button>
            </c-tooltip>

            <c-tooltip :tooltip="$t('home.uiLib')" position="bottom">
              <c-button v-if="config.app.env === 'development'" to="/c-lib" circle variant="text" :aria-label="$t('home.uiLib')">
                <icon-mdi:brush-variant text-20px />
              </c-button>
            </c-tooltip>

            <command-palette />

            <locale-selector v-if="!styleStore.isSmallScreen" />

            <div>
              <NavbarButtons v-if="!styleStore.isSmallScreen" />
            </div>

            <c-tooltip position="bottom" :tooltip="$t('home.support')">
              <c-button
                round
                href="https://www.buymeacoffee.com/cthmsst"
                rel="noopener"
                target="_blank"
                class="support-button"
                :bordered="false"
                @click="() => tracker.trackEvent({ eventName: 'Support button clicked' })"
              >
                {{ $t('home.buyMeACoffee') }}
                <NIcon v-if="!styleStore.isSmallScreen" :component="Heart" ml-2 />
              </c-button>
            </c-tooltip>
          </div>
        </div>

        <div class="main-content">
          <slot />
        </div>
      </div>
    </template>
  </MenuLayout>
</template>

<style lang="less" scoped>
/* ===== 新增的全屏和内部滚动样式 ===== */

.menu-layout {
  height: 100vh;
  overflow: hidden;
}

.content-wrapper {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.navbar-wrapper {
  flex-shrink: 0;
  padding: 12px;
  border-bottom: 1px solid v-bind('themeVars.dividerColor');
}

.main-content {
  flex-grow: 1;
  overflow-y: auto;
  padding: 16px;
}

/* ===== 侧边栏紧凑布局样式 ===== */

.sider-content {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding-top: 20px;
}

.menu-and-selector-wrapper {
  flex-grow: 1;
  overflow-y: auto;
  padding-bottom: 20px;
}

.footer {
  flex-shrink: 0;
  text-align: center;
  color: #838587;
  padding: 20px 0;
}

/* ===== 其他样式 ===== */

.support-button {
  background: rgb(37, 99, 108);
  background: linear-gradient(48deg, rgba(37, 99, 108, 1) 0%, rgba(59, 149, 111, 1) 60%, rgba(20, 160, 88, 1) 100%);
  color: #fff !important;
  transition: padding ease 0.2s !important;

  &:hover {
    color: #fff;
    padding-left: 30px;
    padding-right: 30px;
  }
}
</style>

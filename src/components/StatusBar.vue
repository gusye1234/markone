<script>
import UpdateStatusItem from './UpdateStatusItem.vue';
import { LANGUAGES } from '../editor/languages.js';

const LANGUAGE_MAP = Object.fromEntries(LANGUAGES.map((l) => [l.token, l]));
const LANGUAGE_NAMES = Object.fromEntries(
  LANGUAGES.map((l) => [l.token, l.name])
);

export default {
  props: [
    'line',
    'column',
    'selectionSize',
    'language',
    'languageAuto',
    'theme',
    'systemTheme',
    'allowBetaVersions',
  ],

  components: {
    UpdateStatusItem,
  },

  mounted() {},

  computed: {
    languageName() {
      return LANGUAGE_NAMES[this.language] || this.language;
    },

    className() {
      return `status`;
    },

    supportsFormat() {
      const lang = LANGUAGE_MAP[this.language];
      return !!lang ? lang.supportsFormat : false;
    },

    cmdKey() {
      return window.heynote.platform.isMac ? '⌘' : 'Ctrl';
    },

    formatBlockTitle() {
      return `Format Block Content (Alt + Shift + F)`;
    },

    changeLanguageTitle() {
      return `Change language for current block (${this.cmdKey} + L)`;
    },
  },
};
</script>

<template>
  <div class="status">
    <div class="status-block line-number">
      Ln <span class="num">{{ line }}</span> Col
      <span class="num">{{ column }}</span>
      <template v-if="selectionSize > 0">
        Sel <span class="num">{{ selectionSize }}</span>
      </template>
    </div>
    <div class="spacer"></div>
    <div
      @click="$emit('openLanguageSelector')"
      class="status-block lang clickable"
      :title="changeLanguageTitle"
    >
      {{ languageName }}
      <span v-if="languageAuto" class="auto">(auto)</span>
    </div>
    <div
      v-if="supportsFormat"
      @click="$emit('formatCurrentBlock')"
      class="status-block format clickable"
      :title="formatBlockTitle"
    >
      <span class="icon icon-format"></span>
    </div>
    <UpdateStatusItem :allowBetaVersions="allowBetaVersions" />
    <div
      class="status-block theme clickable"
      @click="$emit('toggleTheme')"
      title="Toggle dark/light mode"
    >
      <span :class="'icon ' + systemTheme"></span>
    </div>
  </div>
</template>

<style scoped lang="sass">
.status
    box-sizing: border-box
    height: 22px
    width: 100%
    background: #48b57e
    color: #fff
    font-family: "Open Sans"
    font-size: 12px
    padding-left: 0px
    padding-right: 0px
    display: flex
    flex-direction: row
    align-items: center
    user-select: none

    +dark-mode
        background: #0e1217
        color: rgba(255, 255, 255, 0.75)

    .spacer
        flex-grow: 2

    .status-block
        box-sizing: border-box
        height: 22px
        padding: 1px 10px
        cursor: default
        &:first-child
            padding-left: 12px
        &:last-child
            padding-right: 12px
        &.clickable
            cursor: pointer
            &:hover
                background-color: rgba(255,255,255, 0.1)
    .line-number
        color: rgba(255, 255, 255, 0.7)
        .num
            color: rgba(255, 255, 255, 1.0)
        +dark-mode
            color: rgba(255, 255, 255, 0.55)
            .num
                color: rgba(255, 255, 255, 0.75)
    .lang .auto
        color: rgba(255, 255, 255, 0.7)
        +dark-mode
            color: rgba(255, 255, 255, 0.55)
    .theme
        padding-top: 0
        padding-bottom: 0
        .icon
            display: block
            width: 14px
            height: 22px
            background-size: 14px
            background-repeat: no-repeat
            background-position: center center
            +dark-mode
                opacity: 0.9
            &.dark
                background-image: url("icons/dark-mode.png")
            &.light
                background-image: url("icons/light-mode.png")
            &.system
                background-image: url("icons/both-mode.png")

    .format
        padding-top: 0
        padding-bottom: 0
        .icon
            display: block
            width: 14px
            height: 22px
            +dark-mode
                opacity: 0.9
            background-size: 16px
            background-repeat: no-repeat
            background-position: center center
            background-image: url("icons/format.svg")
</style>

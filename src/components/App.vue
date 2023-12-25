<script>
import StatusBar from './StatusBar.vue';
import Editor from './Editor.vue';
import LanguageSelector from './LanguageSelector.vue';
import Settings from './settings/Settings.vue';

export default {
  components: {
    Editor,
    StatusBar,
    LanguageSelector,
    Settings,
  },

  data() {
    return {
      line: 1,
      column: 1,
      selectionSize: 0,
      language: 'plaintext',
      languageAuto: true,
      theme: window.darkMode.initial,
      initialTheme: window.darkMode.initial,
      systemTheme: 'system',
      development: window.location.href.indexOf('dev=1') !== -1,
      showLanguageSelector: false,
      showSettings: false,
      settings: window.heynote.settings,
    };
  },

  mounted() {
    window.darkMode.get().then((mode) => {
      this.theme = mode.computed;
      this.systemTheme = mode.theme;
    });
    window.darkMode.onChange((theme) => {
      this.theme = theme;
    });
    window.heynote.onSettingsChange((settings) => {
      this.settings = settings;
    });
    window.heynote.onOpenSettings(() => {
      this.showSettings = true;
    });
  },

  beforeUnmount() {
    window.darkMode.removeListener();
  },

  methods: {
    openSettings() {
      this.showSettings = true;
    },
    closeSettings() {
      this.showSettings = false;
      this.$refs.editor.focus();
    },

    toggleTheme() {
      let newTheme;
      // when the "system" theme is used, make sure that the first click always results in an actual theme change
      if (this.systemTheme === 'system') {
        newTheme = this.initialTheme === 'light' ? 'dark' : 'light';
      } else {
        newTheme = this.systemTheme === 'light' ? 'dark' : 'light';
      }
      window.darkMode.set(newTheme);
      this.systemTheme = newTheme;
      this.$refs.editor.focus();
    },

    onCursorChange(e) {
      this.line = e.cursorLine.line;
      this.column = e.cursorLine.col;
      this.selectionSize = e.selectionSize;
      this.language = e.language;
      this.languageAuto = e.languageAuto;
    },

    openLanguageSelector() {
      this.showLanguageSelector = true;
    },

    closeLanguageSelector() {
      this.showLanguageSelector = false;
      this.$refs.editor.focus();
    },

    onSelectLanguage(language) {
      this.showLanguageSelector = false;
      this.$refs.editor.setLanguage(language);
    },

    formatCurrentBlock() {
      this.$refs.editor.formatCurrentBlock();
    },
  },
};
</script>

<template>
  <div
    class="w-screen h-screen relative flex flex-col justify-start items-center gap-2"
  >
    <div
      class="absolute h-[28px] top-0 w-full text-center text-zinc-500 flex flex-row justify-center items-center"
      id="drag-region"
    >
      <p class="font-semibold">Markone</p>
    </div>
    <Editor
      @cursorChange="onCursorChange"
      :theme="theme"
      :development="development"
      :debugSyntaxTree="false"
      :keymap="settings.keymap"
      :showLineNumberGutter="settings.showLineNumberGutter"
      :showFoldGutter="settings.showFoldGutter"
      class="w-full grow overflow-y-scroll pt-[28px] pb-[16px] no-scrollbar"
      ref="editor"
      @openLanguageSelector="openLanguageSelector"
    />
    <StatusBar
      :line="line"
      :column="column"
      :selectionSize="selectionSize"
      :language="language"
      :languageAuto="languageAuto"
      :theme="theme"
      :systemTheme="systemTheme"
      :allowBetaVersions="settings.allowBetaVersions"
      @toggleTheme="toggleTheme"
      @openLanguageSelector="openLanguageSelector"
      @formatCurrentBlock="formatCurrentBlock"
      class="absolute bottom-0 w-full left-0"
    />
    <div class="overlay">
      <LanguageSelector
        v-if="showLanguageSelector"
        @selectLanguage="onSelectLanguage"
        @close="closeLanguageSelector"
      />
      <Settings
        v-if="showSettings"
        :initialSettings="settings"
        @closeSettings="closeSettings"
      />
    </div>
  </div>
</template>

<template>
  <div id="app" :class="{ 'user-select-none': userSelectNone }">
    <Scrollbar v-show="!showLyrics" ref="scrollbar" />
    <Navbar
      v-show="showNavbar"
      ref="navbar"
      :sidebar-open="isSidebarOpen"
      @toggle-sidebar="toggleSidebar"
    />
    <transition :name="sidebarTransitionName">
      <div
        v-if="isSidebarOpen"
        class="app-sidebar-wrapper"
        :class="sidebarWrapperClasses"
        role="dialog"
        aria-modal="true"
      >
        <Sidebar class="app-sidebar-panel" />
      </div>
    </transition>
    <div
      v-if="isSidebarOpen && isPhoneViewport"
      class="app-sidebar-backdrop"
      @click="closeSidebar"
    ></div>
    <main
      ref="main"
      :style="{ overflow: enableScrolling ? 'auto' : 'hidden' }"
      @scroll="handleScroll"
    >
      <keep-alive>
        <router-view v-if="$route.meta.keepAlive"></router-view>
      </keep-alive>
      <router-view v-if="!$route.meta.keepAlive"></router-view>
    </main>
    <transition name="slide-up">
      <Player v-if="enablePlayer" v-show="showPlayer" ref="player" />
    </transition>
    <Toast />
    <ModalAddTrackToPlaylist v-if="isAccountLoggedIn" />
    <ModalNewPlaylist v-if="isAccountLoggedIn" />
    <transition v-if="enablePlayer" name="slide-up">
      <Lyrics v-show="showLyrics" />
    </transition>
  </div>
</template>

<script>
import ModalAddTrackToPlaylist from './components/ModalAddTrackToPlaylist.vue';
import ModalNewPlaylist from './components/ModalNewPlaylist.vue';
import Navbar from './components/Navbar.vue';
import Scrollbar from './components/Scrollbar.vue';
import Player from './components/Player.vue';
import Toast from './components/Toast.vue';
import Sidebar from './components/Sidebar.vue';
import { ipcRenderer } from './electron/ipcRenderer';
import { isAccountLoggedIn, isLooseLoggedIn } from '@/utils/auth';
import Lyrics from './views/lyrics.vue';
import { mapState } from 'vuex';

export default {
  name: 'App',
  components: {
    Navbar,
    Player,
    Toast,
    ModalAddTrackToPlaylist,
    ModalNewPlaylist,
    Lyrics,
    Scrollbar,
    Sidebar,
  },
  data() {
    return {
      isElectron: process.env.IS_ELECTRON, // true || undefined
      userSelectNone: false,
      isSidebarOpen: false,
      isPhoneViewport: false,
      sidebarMediaQuery: null,
      sidebarMediaHandler: null,
      htmlOverflowBeforeSidebar: null,
    };
  },
  computed: {
    ...mapState(['showLyrics', 'settings', 'player', 'enableScrolling']),
    isAccountLoggedIn() {
      return isAccountLoggedIn();
    },
    showPlayer() {
      return (
        [
          'mv',
          'loginUsername',
          'login',
          'loginAccount',
          'lastfmCallback',
        ].includes(this.$route.name) === false
      );
    },
    enablePlayer() {
      return this.player.enabled && this.$route.name !== 'lastfmCallback';
    },
    showNavbar() {
      return this.$route.name !== 'lastfmCallback';
    },
    sidebarTransitionName() {
      return this.isPhoneViewport ? 'sheet-up' : 'sidebar-slide';
    },
    sidebarWrapperClasses() {
      return {
        'app-sidebar-wrapper--sheet': this.isPhoneViewport,
        'app-sidebar-wrapper--inline': !this.isPhoneViewport,
      };
    },
  },
  created() {
    if (this.isElectron) ipcRenderer(this);
    window.addEventListener('keydown', this.handleKeydown);
    this.fetchData();
  },
  mounted() {
    this.cacheHtmlOverflowValue();
    this.setupSidebarMediaWatcher();
    this.initializeSidebarState();
  },
  beforeDestroy() {
    this.removeSidebarMediaWatcher();
    this.restoreHtmlOverflowValue();
    this.updateRootSidebarClass(false);
  },
  methods: {
    handleKeydown(e) {
      if (e.code === 'Space') {
        if (e.target.tagName === 'INPUT') return false;
        if (this.$route.name === 'mv') return false;
        e.preventDefault();
        this.player.playOrPause();
      }
    },
    fetchData() {
      if (!isLooseLoggedIn()) return;
      this.$store.dispatch('fetchLikedSongs');
      this.$store.dispatch('fetchLikedSongsWithDetails');
      this.$store.dispatch('fetchLikedPlaylist');
      if (isAccountLoggedIn()) {
        this.$store.dispatch('fetchLikedAlbums');
        this.$store.dispatch('fetchLikedArtists');
        this.$store.dispatch('fetchLikedMVs');
        this.$store.dispatch('fetchCloudDisk');
      }
    },
    handleScroll() {
      this.$refs.scrollbar.handleScroll();
    },
    toggleSidebar(forceState) {
      const nextState =
        typeof forceState === 'boolean' ? forceState : !this.isSidebarOpen;
      this.isSidebarOpen = nextState;
    },
    closeSidebar() {
      if (this.isSidebarOpen) {
        this.toggleSidebar(false);
      }
    },
    cacheHtmlOverflowValue() {
      if (typeof window === 'undefined' || typeof document === 'undefined')
        return;
      const rootStyle = getComputedStyle(document.documentElement);
      this.htmlOverflowBeforeSidebar =
        rootStyle.getPropertyValue('--html-overflow-y').trim() || 'overlay';
    },
    restoreHtmlOverflowValue() {
      if (typeof document === 'undefined') return;
      if (this.htmlOverflowBeforeSidebar) {
        document.documentElement.style.setProperty(
          '--html-overflow-y',
          this.htmlOverflowBeforeSidebar
        );
      }
    },
    setupSidebarMediaWatcher() {
      if (typeof window === 'undefined' || !window.matchMedia) return;
      this.sidebarMediaQuery = window.matchMedia('(max-width: 760px)');
      this.isPhoneViewport = this.sidebarMediaQuery.matches;
      this.sidebarMediaHandler = event => {
        this.isPhoneViewport = event.matches;
      };
      if (this.sidebarMediaQuery.addEventListener) {
        this.sidebarMediaQuery.addEventListener(
          'change',
          this.sidebarMediaHandler
        );
      } else if (this.sidebarMediaQuery.addListener) {
        this.sidebarMediaQuery.addListener(this.sidebarMediaHandler);
      }
    },
    removeSidebarMediaWatcher() {
      if (!this.sidebarMediaQuery || !this.sidebarMediaHandler) return;
      if (this.sidebarMediaQuery.removeEventListener) {
        this.sidebarMediaQuery.removeEventListener(
          'change',
          this.sidebarMediaHandler
        );
      } else if (this.sidebarMediaQuery.removeListener) {
        this.sidebarMediaQuery.removeListener(this.sidebarMediaHandler);
      }
      this.sidebarMediaQuery = null;
      this.sidebarMediaHandler = null;
    },
    initializeSidebarState() {
      if (this.isPhoneViewport) {
        this.isSidebarOpen = false;
      } else {
        this.isSidebarOpen = true;
      }
    },
    updateRootSidebarClass(isOpen) {
      if (typeof document === 'undefined') return;
      const root = document.documentElement;
      if (!root) return;
      if (isOpen && !this.isPhoneViewport) {
        root.classList.add('sidebar-open');
      } else {
        root.classList.remove('sidebar-open');
      }
    },
  },
  watch: {
    $route() {
      if (this.isPhoneViewport) this.closeSidebar();
    },
    isSidebarOpen(isOpen) {
      if (typeof document === 'undefined') return;
      if (!this.htmlOverflowBeforeSidebar) {
        this.cacheHtmlOverflowValue();
      }
      document.documentElement.style.setProperty(
        '--html-overflow-y',
        isOpen && this.isPhoneViewport
          ? 'hidden'
          : this.htmlOverflowBeforeSidebar
      );
      this.updateRootSidebarClass(isOpen);
    },
    isPhoneViewport(isPhone) {
      if (isPhone && this.isSidebarOpen) {
        this.isSidebarOpen = false;
      } else if (!isPhone && !this.isSidebarOpen) {
        this.isSidebarOpen = true;
      } else {
        this.updateRootSidebarClass(this.isSidebarOpen);
      }
    },
  },
};
</script>

<style lang="scss">
#app {
  width: 100%;
  transition: all 0.4s;
}

main {
  position: fixed;
  top: 0;
  bottom: 0;
  right: var(--app-sidebar-offset, 0);
  left: 0;
  overflow: auto;
  // add extra top padding to clear the floating navbar (64px height + 40px gap)
  padding: 104px 10vw 96px 10vw;
  box-sizing: border-box;
  scrollbar-width: none; // firefox
  transition: right 0.35s ease;
}

@media (max-width: 1336px) {
  main {
    padding: 104px 5vw 96px 5vw;
  }
}

// On small screens where the navbar becomes full-width and sticks to the top,
// revert to the original top padding equal to the navbar height.
@media (max-width: 760px) {
  main {
    padding-top: 64px;
  }
}

main::-webkit-scrollbar {
  width: 0px;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.4s;
}
.slide-up-enter,
.slide-up-leave-to {
  transform: translateY(100%);
}

.app-sidebar-wrapper {
  position: fixed;
  top: 0;
  bottom: 0;
  right: 0;
  width: var(--app-sidebar-width);
  background-color: var(--color-body-bg);
  z-index: 500;
  display: flex;
  flex-direction: column;
  padding: 104px 24px 96px;
  border-left: 2px solid rgba(140, 140, 140, 0.2);
}

.app-sidebar-wrapper--sheet {
  top: auto;
  left: 0;
  right: 0;
  bottom: 0;
  width: auto;
  height: min(80vh, 560px);
  border-radius: 32px 32px 0 0;
  box-shadow: 0 -20px 50px rgba(0, 0, 0, 0.35);
  padding: 32px 24px;
}

.app-sidebar-panel {
  flex: 1;
}

.app-sidebar-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.3);
  z-index: 1100;
  animation: backdrop-fade 0.4s;
}

@keyframes backdrop-fade {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.sidebar-slide-enter-active,
.sidebar-slide-leave-active {
  transition: transform 0.35s ease, opacity 0.35s ease;
}
.sidebar-slide-enter,
.sidebar-slide-leave-to {
  transform: translateX(120%);
  opacity: 0;
}

.sheet-up-enter-active,
.sheet-up-leave-active {
  transition: transform 0.35s ease;
}
.sheet-up-enter,
.sheet-up-leave-to {
  transform: translateY(100%);
}
</style>

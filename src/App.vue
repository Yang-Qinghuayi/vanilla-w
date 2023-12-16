<template>
  <div id="app" :class="{ 'user-select-none': userSelectNone }">
    <Scrollbar v-show="!showLyrics" ref="scrollbar"/>
    <div style="display: grid; grid-template-columns: 1fr 18fr; ">
      <Navbar style="width: 10px" v-show="false" ref="navbar"/>
      <div class="nvbar">

        <div class="nav-item" @click="go('library')">我的</div>
        <div class="nav-item" @click="go('home')">推荐</div>
        <div class="nav-item" @click="go('explore')">发现</div>
        <div class="nav-item" @click="go('settings')">设置</div>

      </div>
      <div>
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
      </div>
    </div>
    <transition name="slide-up">
      <Player v-if="enablePlayer" v-show="showPlayer" ref="player"/>
    </transition>

    <Toast/>
    <ModalAddTrackToPlaylist v-if="isAccountLoggedIn"/>
    <ModalNewPlaylist v-if="isAccountLoggedIn"/>
    <transition v-if="enablePlayer" name="slide-up">
      <Lyrics v-show="showLyrics"/>
    </transition>
  </div>
</template>

<script>
import ModalAddTrackToPlaylist from './components/ModalAddTrackToPlaylist.vue';
import ModalNewPlaylist from './components/ModalNewPlaylist.vue';
import Scrollbar from './components/Scrollbar.vue';
import Navbar from './components/Navbar.vue';
import Player from './components/Player.vue';
import Toast from './components/Toast.vue';
import {ipcRenderer} from './electron/ipcRenderer';
import {isAccountLoggedIn, isLooseLoggedIn} from '@/utils/auth';
import Lyrics from './views/lyrics.vue';
import {mapState} from 'vuex';
import {fastKey} from "core-js/internals/internal-metadata";

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
  },
  data() {
    return {
      isElectron: process.env.IS_ELECTRON, // true || undefined
      userSelectNone: false,
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
  },
  created() {
    if (this.isElectron) ipcRenderer(this);
    window.addEventListener('keydown', this.handleKeydown);
    this.fetchData();
  },
  methods: {

    go(where) {
      this.$router.push({path: where});
    },

    fastKey,
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
  },
};
</script>

<style lang="scss">
#app {
  width: 100%;
  transition: all 0.4s;
}


.active {
  background: var(--color-primary-bg-for-transparent);

  input,
  .svg-icon {
    opacity: 1;
    color: var(--color-primary);
  }
}

main {
  //position: fixed;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
  overflow: auto;
  margin-bottom: 0;
  padding: 20px 2vw 11px 12vw;
  box-sizing: border-box;
  scrollbar-width: none; // firefox
  margin-left: 140px;
  width: 90vw;
//  边框阴影
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  background: #fdfdf5;
}

@media (max-width: 1736px) {
  main {
    padding: 4px 2vw 106px 2vw;
  }
}

main::-webkit-scrollbar {
  width: 0;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.4s;
}

.slide-up-enter,
.slide-up-leave-to {
  transform: translateY(100%);
}

.nvbar {
  position: fixed;
  height: 900px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.nav-item {
  align-content: center;
  display: flex;
  font-size: 20px;
  font-weight: bold;
  justify-content: center;
  width: 100px;
  /* 条目宽度占满容器 */
  height: 30px;
  line-height: 30px;
  padding: 10px;
  background-color: #f1f1f1;

  /* 可以根据需要调整内边距 */
  margin: 5px;
  /* 设置边框 */
  border: 0 solid #ccc;
  border-radius: 5px;
}
</style>

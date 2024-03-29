<template>
  <div class="player" @click="toggleLyrics">
    <div class="controls">
      <!-- 歌曲头像以及名称作者红心部分 -->
      <div class="playing">
        <div class="container" @click.stop>
          <!-- cover -->
          <img
            :src="currentTrack.al && currentTrack.al.picUrl | resizeImage(224)"
            loading="lazy"
            @click="goToAlbum"
          />
          <!-- name -->
          <div class="track-info" :title="audioSource">
            <!-- song name -->
            <div
              :class="['name', { 'has-list': hasList() }]"
              @click="hasList() && goToList()"
            >
              {{ currentTrack.name }}
            </div>
            <!-- artists name -->
            <div class="artist">
              <span
                v-for="(ar, index) in currentTrack.ar"
                :key="ar.id"
                @click="ar.id && goToArtist(ar.id)"
              >
                <span :class="{ ar: ar.id }"> {{ ar.name }} </span
                ><span v-if="index !== currentTrack.ar.length - 1">, </span>
              </span>
            </div>
            <!-- end -->
          </div>

          <!-- like -->
          <div class="like-button">
            <button-icon
              :title="
                player.isCurrentTrackLiked
                  ? $t('player.unlike')
                  : $t('player.like')
              "
              @click.native="likeATrack(player.currentTrack.id)"
            >
              <svg-icon
                v-show="!player.isCurrentTrackLiked"
                icon-class="heart"
              ></svg-icon>
              <svg-icon
                v-show="player.isCurrentTrackLiked"
                icon-class="heart-solid"
              ></svg-icon>
            </button-icon>
          </div>
        </div>
        <div class="blank"></div>
      </div>

      <!-- 中间的三个按钮 -->
      <div class="middle-control-buttons">
        <!-- 用于留出空白 -->
        <div class="blank"></div>
        <div class="container" @click.stop>
          <button-icon
            class="previous"
            v-show="!player.isPersonalFM"
            :title="$t('player.previous')"
            @click.native="playPrevTrack"
          >
            <svg-icon icon-class="previous" />
          </button-icon>
          <button-icon
            v-show="player.isPersonalFM"
            title="不喜欢"
            @click.native="moveToFMTrash"
          >
            <svg-icon icon-class="thumbs-down" />
          </button-icon>
          <button-icon
            class="play"
            :title="$t(player.playing ? 'player.pause' : 'player.play')"
            @click.native="playOrPause"
          >
            <svg-icon :icon-class="player.playing ? 'pause' : 'play'" />
          </button-icon>
          <button-icon
            class="previous"
            :title="$t('player.next')"
            @click.native="playNextTrack"
          >
            <svg-icon icon-class="next" />
          </button-icon>
        </div>
        <div class="blank"></div>
      </div>
      <!-- right -->
      <div class="right-control-buttons">
        <div class="blank"></div>
        <div class="container" @click.stop>
          <button-icon
            :title="$t('player.nextUp')"
            :class="{
              active: $route.name === 'next',
              disabled: player.isPersonalFM,
            }"
            @click.native="goToNextTracksPage"
          >
            <svg-icon icon-class="list" />
          </button-icon>
          <button-icon
            :class="{
              active: player.repeatMode !== 'off',
              disabled: player.isPersonalFM,
            }"
            :title="
              player.repeatMode === 'one'
                ? $t('player.repeatTrack')
                : $t('player.repeat')
            "
            @click.native="switchRepeatMode"
          >
            <svg-icon
              v-show="player.repeatMode !== 'one'"
              icon-class="repeat"
            />
            <svg-icon
              v-show="player.repeatMode === 'one'"
              icon-class="repeat-1"
            />
          </button-icon>
          <button-icon
            :class="{ active: player.shuffle, disabled: player.isPersonalFM }"
            :title="$t('player.shuffle')"
            @click.native="switchShuffle"
          >
            <svg-icon icon-class="shuffle" />
          </button-icon>
          <button-icon
            v-if="settings.enableReversedMode"
            :class="{ active: player.reversed, disabled: player.isPersonalFM }"
            :title="$t('player.reversed')"
            @click.native="switchReversed"
          >
            <svg-icon icon-class="sort-up" />
          </button-icon>
          <!-- volume -->
          <div class="volume-control">
            <button-icon :title="$t('player.mute')" @click.native="mute">
              <svg-icon v-show="volume > 0.5" icon-class="volume" />
              <svg-icon v-show="volume === 0" icon-class="volume-mute" />
              <svg-icon
                v-show="volume <= 0.5 && volume !== 0"
                icon-class="volume-half"
              />
            </button-icon>
            <div class="volume-bar">
              <vue-slider
                v-model="volume"
                :min="0"
                :max="1"
                :interval="0.01"
                :drag-on-click="true"
                :duration="0"
                tooltip="none"
                :dot-size="12"
              ></vue-slider>
            </div>
          </div>

          <button-icon
            class="lyrics-button"
            title="歌词"
            style="margin-left: 12px"
            @click.native="toggleLyrics"
          >
            <svg-icon icon-class="arrow-up" />
          </button-icon>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapMutations, mapActions } from 'vuex';
import '@/assets/css/slider.css';

import ButtonIcon from '@/components/ButtonIcon.vue';
import VueSlider from 'vue-slider-component';
import { goToListSource, hasListSource } from '@/utils/playList';
import { formatTrackTime } from '@/utils/common';

export default {
  name: 'Player',
  components: {
    ButtonIcon,
    VueSlider,
  },
  computed: {
    ...mapState(['player', 'settings', 'data']),
    currentTrack() {
      return this.player.currentTrack;
    },
    volume: {
      get() {
        return this.player.volume;
      },
      set(value) {
        this.player.volume = value;
      },
    },
    playing() {
      return this.player.playing;
    },
    audioSource() {
      return this.player._howler?._src.includes('kuwo.cn')
        ? '音源来自酷我音乐'
        : '';
    },
  },
  methods: {
    ...mapMutations(['toggleLyrics']),
    ...mapActions(['showToast', 'likeATrack']),
    playPrevTrack() {
      this.player.playPrevTrack();
    },
    playOrPause() {
      this.player.playOrPause();
    },
    playNextTrack() {
      if (this.player.isPersonalFM) {
        this.player.playNextFMTrack();
      } else {
        this.player.playNextTrack();
      }
    },
    goToNextTracksPage() {
      if (this.player.isPersonalFM) return;
      this.$route.name === 'next'
        ? this.$router.go(-1)
        : this.$router.push({ name: 'next' });
    },
    formatTrackTime(value) {
      return formatTrackTime(value);
    },
    hasList() {
      return hasListSource();
    },
    goToList() {
      goToListSource();
    },
    goToAlbum() {
      if (this.player.currentTrack.al.id === 0) return;
      this.$router.push({ path: '/album/' + this.player.currentTrack.al.id });
    },
    goToArtist(id) {
      this.$router.push({ path: '/artist/' + id });
    },
    moveToFMTrash() {
      this.player.moveToFMTrash();
    },
    switchRepeatMode() {
      this.player.switchRepeatMode();
    },
    switchShuffle() {
      this.player.switchShuffle();
    },
    switchReversed() {
      this.player.switchReversed();
    },
    mute() {
      this.player.mute();
    },
  },
};
</script>

<style lang="scss" scoped>
.player {
  position: fixed;
  bottom: 0;
  right: 0;
  left: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  height: 68px;
  width: 58vw;
  // color: #dfebc9;
  margin-left: 22vw;
  //margin-bottom: 10px;
  border-radius: 10px;
  background-color: #e8f0fe;
  z-index: 100;
}

@supports (-moz-appearance: none) {
  .player {
    background-color: #444746;
  }
}

.progress-bar {
  margin-top: -6px;
  margin-bottom: -6px;
  width: 67.8vw;
  margin-left: 0.2vw;
  //height: 10px;
}

.controls {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  height: 100%;

  padding: {
    right: 2vw;
    left: 2vw;
  }
}

@media (max-width: 1336px) {
  .controls {
    padding: 0 5vw;
  }
}

.blank {
  flex-grow: 1;
}

.playing {
  display: flex;
}

.playing .container {
  display: flex;
  align-items: center;

  .svg-icon {
    color: #b7a64a;
    height: 24px;
    width: 24px;
  }

  img {
    height: 46px;
    border-radius: 5px;
    box-shadow: 0 6px 8px -2px rgba(203, 184, 184, 0.16);
    cursor: pointer;
    user-select: none;
  }

  .track-info {
    // color: #eee;
    height: 46px;
    margin-left: 12px;
    display: flex;
    flex-direction: column;
    justify-content: center;

    .name {
      font-weight: 600;
      font-size: 16px;
      opacity: 0.88;
      color: #b7a64a;
      margin-bottom: 4px;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 1;
      overflow: hidden;
      word-break: break-all;
    }

    .has-list {
      cursor: pointer;

      &:hover {
        text-decoration: underline;
      }
    }

    .artist {
      font-size: 12px;
      opacity: 0.58;
      color: #444746;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 1;
      overflow: hidden;
      word-break: break-all;

      span.ar {
        cursor: pointer;

        &:hover {
          text-decoration: underline;
        }
      }
    }
  }
}

.middle-control-buttons {
  display: flex;
}

.middle-control-buttons .container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0 8px;

  .button-icon {
    margin: 0 8px;
  }

  .play {
    height: 42px;
    width: 42px;

    .svg-icon {
      color: #8e6b12;
      width: 24px;
      height: 24px;
    }
  }
}

.right-control-buttons {
  display: flex;
}

.previous {
  .svg-icon {
    color: #8ba076;
    height: 20px;
    width: 24px;
  }
}

.right-control-buttons .container {
  display: flex;
  justify-content: flex-end;
  align-items: center;

  .expand {
    margin-left: 24px;
  }

  .svg-icon {
    color: #b7a64a;
  }

  .active .svg-icon {
    color: #8e6b12;
  }

  .volume-control {
    margin-left: 4px;
    display: flex;
    align-items: center;

    .volume-bar {
      // height: 20px;
      width: 84px;
    }
  }
}

.like-button {
  margin-left: 16px;

  .svg-icon {
    color: #b7a64a;
    height: 24px;
    width: 24px;
  }
}

.button-icon.disabled {
  cursor: default;
  opacity: 0.38;

  &:hover {
    background: none;
  }

  &:active {
    transform: unset;
  }
}
</style>

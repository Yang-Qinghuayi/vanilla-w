<template>
  <div class="cover-row" :style="rowStyles">
    <div
      v-for="item in items"
      :key="item.id"
      class="item"
      :class="{ artist: type === 'artist' }"
    >
      <Cover
        :id="item.id"
        :image-url="getImageUrl(item)"
        :type="type"
        :play-button-size="type === 'artist' ? 16 : playButtonSize"
      />

      <div
        style="
          display: flex;
          flex-direction: row;
          align-items: center;
          justify-content: space-between;
        "
      >
        <div class="text">
          <router-link :to="getTitleLink(item)">{{ item.name }}</router-link>
        </div>

        <button @click.stop="play(item)">
          <svg-icon
            style="height: 10px; width: 10px; margin-right: 5px"
            icon-class="play"
          />
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import Cover from '@/components/Cover.vue';

export default {
  name: 'CoverRow',
  components: {
    Cover,
  },
  props: {
    items: { type: Array, required: true },
    type: { type: String, required: true },
    subText: { type: String, default: 'null' },
    subTextFontSize: { type: String, default: '16px' },
    showPlayCount: { type: Boolean, default: false },
    columnNumber: { type: Number, default: 6 },
    gap: { type: String, default: '44px 24px' },
    playButtonSize: { type: Number, default: 16 },
  },
  computed: {
    playButtonStyles() {
      let styles = {};
      styles.width = this.playButtonSize + '%';
      styles.height = this.playButtonSize + '%';
      return styles;
    },

    rowStyles() {
      return {
        'grid-template-columns': `repeat(${this.columnNumber}, 1fr)`,
        gap: this.gap,
      };
    },
  },
  methods: {
    play(item) {
      const player = this.$store.state.player;
      const playActions = {
        album: player.playAlbumByID,
        playlist: player.playPlaylistByID,
        artist: player.playArtistByID,
      };
      playActions[this.type].bind(player)(item.id);
    },

    getSubText(item) {
      if (this.subText === 'copywriter') return item.copywriter;
      if (this.subText === 'description') return item.description;
      if (this.subText === 'updateFrequency') return item.updateFrequency;
      if (this.subText === 'creator') return 'by ' + item.creator.nickname;
      if (this.subText === 'releaseYear')
        return new Date(item.publishTime).getFullYear();
      if (this.subText === 'artist') {
        if (item.artist !== undefined)
          return `<a href="/#/artist/${item.artist.id}">${item.artist.name}</a>`;
        if (item.artists !== undefined)
          return `<a href="/#/artist/${item.artists[0].id}">${item.artists[0].name}</a>`;
      }
      if (this.subText === 'albumType+releaseYear') {
        let albumType = item.type;
        if (item.type === 'EP/Single') {
          albumType = item.size === 1 ? 'Single' : 'EP';
        } else if (item.type === 'Single') {
          albumType = 'Single';
        } else if (item.type === '专辑') {
          albumType = 'Album';
        }
        return `${albumType} · ${new Date(item.publishTime).getFullYear()}`;
      }
      if (this.subText === 'appleMusic') return 'by Apple Music';
    },
    isPrivacy(item) {
      return this.type === 'playlist' && item.privacy === 10;
    },
    isExplicit(item) {
      return this.type === 'album' && item.mark === 1056768;
    },
    getTitleLink(item) {
      return `/${this.type}/${item.id}`;
    },
    getImageUrl(item) {
      if (item.img1v1Url) {
        let img1v1ID = item.img1v1Url.split('/');
        img1v1ID = img1v1ID[img1v1ID.length - 1];
        if (img1v1ID === '5639395138885805.jpg') {
          // 没有头像的歌手，网易云返回的img1v1Url并不是正方形的 😅😅😅
          return 'https://p2.music.126.net/VnZiScyynLG7atLIZ2YPkw==/18686200114669622.jpg?param=512y512';
        }
      }
      let img = item.img1v1Url || item.picUrl || item.coverImgUrl;
      return `${img?.replace('http://', 'https://')}?param=512y512`;
    },
  },
};
</script>

<style lang="scss" scoped>
.cover-row {
  display: grid;
}

.item {
  background: #f5f0f0;
  height: 272px;
  border-radius: 0.75em;

  .text {
    padding-top: 10px;
    margin-left: 10px;
    margin-bottom: 10px;
    color: #666;
    font-size: 14px;
    font-weight: 560;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
    word-break: break-all;
  }
}

.item.artist {
  display: flex;
  flex-direction: column;
  text-align: center;

  .cover {
    display: flex;
  }

  .title {
    //margin-top: 4px;
  }
}

@media (max-width: 834px) {
  .item .text .title {
    font-size: 14px;
  }
}

.explicit-symbol {
  opacity: 0.28;
  color: var(--color-text);
  float: right;

  .svg-icon {
    margin-bottom: -3px;
  }
}

.lock-icon {
  opacity: 0.28;
  color: var(--color-text);
  margin-right: 4px;
  // float: right;
  .svg-icon {
    height: 12px;
    width: 12px;
  }
}

.play-count {
  font-weight: 600;
  opacity: 0.58;
  color: var(--color-text);
  font-size: 12px;

  .svg-icon {
    margin-right: 3px;
    height: 8px;
    width: 8px;
  }
}
</style>

<template>
  <div>
    kiss
  </div>
</template>

<script>
import { getTrackDetail } from '@/api/track';
import locale from '@/locale';
import { camelCase } from 'change-case';


export default {
  name: 'Search',
  components: {
  },
  data() {
    return { show: false, result: [], hasMore: true };
  },
  computed: {
    keywords() {
      return this.$route.params.keywords;
    },
    type() {
      return camelCase(this.$route.params.type);
    },
    typeNameTable() {
      return {
        musicVideos: locale.t('search.mv'),
        tracks: locale.t('search.song'),
        albums: locale.t('search.album'),
        artists: locale.t('search.artist'),
        playlists: locale.t('search.playlist'),
      };
    },
  },
  created() {
    this.fetchData();
  },
  methods: {
    getTracksDetail() {
      const trackIDs = this.result.map(t => t.id);
      if (trackIDs.length === 0) return;
      getTrackDetail(trackIDs.join(',')).then(result => {
        this.result = result.songs;
      });
    },
  },
};
</script>

<style lang="scss" scoped>
h1 {
  margin-top: 32px;
  margin-bottom: 28px;
  color: var(--color-text);

  span {
    opacity: 0.58;
  }
}

.load-more {
  display: flex;
  justify-content: center;
  margin-top: 32px;
}

.button.more {
  .svg-icon {
    height: 24px;
    width: 24px;
  }
}
</style>

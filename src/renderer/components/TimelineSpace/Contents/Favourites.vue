<template>
  <div id="favourites">
    <div v-shortkey="{linux: ['ctrl', 'r'], mac: ['meta', 'r']}" @shortkey="reload()">
    </div>
    <div class="fav" v-for="message in favourites" v-bind:key="message.id">
      <toot :message="message" v-on:update="updateToot" v-on:delete="deleteToot"></toot>
    </div>
    <div class="loading-card" v-loading="lazyLoading" :element-loading-background="backgroundColor">
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import Toot from './Cards/Toot'

export default {
  name: 'favourites',
  components: { Toot },
  computed: {
    ...mapState({
      account: state => state.TimelineSpace.account,
      favourites: state => state.TimelineSpace.Contents.Favourites.favourites,
      lazyLoading: state => state.TimelineSpace.Contents.Favourites.lazyLoading,
      backgroundColor: state => state.App.theme.background_color,
      startReload: state => state.TimelineSpace.HeaderMenu.reload
    })
  },
  created () {
    this.$store.commit('TimelineSpace/changeLoading', true)
    this.$store.dispatch('TimelineSpace/Contents/Favourites/fetchFavourites', this.account)
      .catch(() => {
        this.$message({
          message: 'Could not fetch favourites',
          type: 'error'
        })
      })
      .finally(() => {
        this.$store.commit('TimelineSpace/changeLoading', false)
      })
  },
  mounted () {
    document.getElementById('scrollable').addEventListener('scroll', this.onScroll)
  },
  destroyed () {
    this.$store.commit('TimelineSpace/Contents/Favourites/updateFavourites', [])
    if (document.getElementById('scrollable') !== undefined && document.getElementById('scrollable') !== null) {
      document.getElementById('scrollable').removeEventListener('scroll', this.onScroll)
      document.getElementById('scrollable').scrollTop = 0
    }
  },
  watch: {
    startReload: function (newState, oldState) {
      if (!oldState && newState) {
        this.reload()
          .finally(() => {
            this.$store.commit('TimelineSpace/HeaderMenu/changeReload', false)
          })
      }
    }
  },
  methods: {
    updateToot (message) {
      this.$store.commit('TimelineSpace/Contents/Favourites/updateToot', message)
    },
    deleteToot (message) {
      this.$store.commit('TimelineSpace/Contents/Favourites/deleteToot', message)
    },
    onScroll (event) {
      if (((event.target.clientHeight + event.target.scrollTop) >= document.getElementById('favourites').clientHeight - 10) && !this.lazyloading) {
        this.$store.dispatch('TimelineSpace/Contents/Favourites/lazyFetchFavourites', this.favourites[this.favourites.length - 1])
          .catch(() => {
            this.$message({
              message: 'Could not fetch favourites',
              type: 'error'
            })
          })
      }
    },
    async reload () {
      this.$store.commit('TimelineSpace/changeLoading', true)
      try {
        const account = await this.$store.dispatch('TimelineSpace/localAccount', this.$route.params.id).catch((err) => {
          this.$message({
            message: 'Could not find account',
            type: 'error'
          })
          throw err
        })

        await this.$store.dispatch('TimelineSpace/stopUserStreaming')
        await this.$store.dispatch('TimelineSpace/stopLocalStreaming')

        await this.$store.dispatch('TimelineSpace/Contents/Home/fetchTimeline', account)
        await this.$store.dispatch('TimelineSpace/Contents/Local/fetchLocalTimeline', account)
        await this.$store.dispatch('TimelineSpace/Contents/Favourites/fetchFavourites', account)
          .catch(() => {
            this.$message({
              message: 'Could not fetch favourites',
              type: 'error'
            })
          })

        this.$store.dispatch('TimelineSpace/startUserStreaming', account)
        this.$store.dispatch('TimelineSpace/startLocalStreaming', account)
      } finally {
        this.$store.commit('TimelineSpace/changeLoading', false)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.loading-card {
  height: 60px;
}

.loading-card:empty {
  height: 0;
}
</style>

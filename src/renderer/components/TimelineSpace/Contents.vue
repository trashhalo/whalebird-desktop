<template>
<div id="contents">
  <div id="scrollable" :class="openSideBar ? 'timeline-wrapper-with-side-bar' : 'timeline-wrapper'">
    <router-view></router-view>
  </div>
  <side-bar></side-bar>
</div>
</template>

<script>
import { mapState } from 'vuex'
import SideBar from './Contents/SideBar'

export default {
  name: 'contents',
  components: {
    SideBar
  },
  computed: {
    ...mapState({
      openSideBar: state => state.TimelineSpace.Contents.SideBar.openSideBar
    })
  }
}
</script>

<style lang="scss" scoped>
#contents {
  padding-top: 48px;
  height: 100%;
  box-sizing: border-box;

  .timeline-wrapper {
    height: 100%;
    width: 100%;
    overflow: auto;
    transition: all 0.5s;
  }

  .timeline-wrapper-with-side-bar {
    height: 100%;
    width: -webkit-calc(100% - 320px);
    overflow: auto;
    transition: all 0.5s;
  }
}
</style>

<template>
  <div id="app">
    <app-titlebar/>
    <transition name="slide-left" mode="out-in">
      <app-drawer v-if="drawerOpen"/>
    </transition>
    <div class="container">
      <div class="half">
        <app-timer />
      </div>
      <div class="half">
        <app-tasks />
      </div>
    </div>
  </div>
</template>

<script>
import appDrawer from '@/components/drawer/Drawer'
import appTimer from '@/components/timer/Timer'
import appTitlebar from '@/components/Titlebar'
import appTasks from '@/components/Tasks'

export default {
  name: 'pomotroid',
  components: {
    appDrawer,
    appTimer,
    appTitlebar,
    appTasks
  },

  computed: {
    // store getters
    drawerOpen () {
      return this.$store.getters.drawerOpen
    }
  }
}
</script>

<style lang="scss">
#app {
  animation: fade-in 0.5s ease forwards;
  position: relative;
  overflow: hidden;
  height: 100vh;

  .container {
    display: flex;
    
    .half {
      width: 50%;
    }
  }
}
</style>

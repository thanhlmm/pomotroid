<template>
  <div class="Drawer">
    <app-settings-timer/>
  </div>
</template>

<script>
import appSettingsTimer from '@/components/drawer/Settings-timer'

export default {
  components: {
    appSettingsTimer
  }
}
</script>

<style lang="scss" scoped>
.Drawer {
  background-color: $colorLightNavy;
  position: absolute;
  width: 50%;
  height: 100%;
  z-index: 1;
  -webkit-app-region: no-drag;
}
</style>

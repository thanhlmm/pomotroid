<template>
  <div>
    <sweet-modal-tab title="Tab 1" id="tab1">Contents of Tab 1</sweet-modal-tab>
    <sweet-modal-tab title="Tab 2" id="tab2">Contents of Tab 2</sweet-modal-tab>
    <sweet-modal-tab title="Tab 3" id="tab3" disabled>Tab 3 is disabled</sweet-modal-tab>
  </div>
</template>

<script>
import { SweetModalTab } from 'sweet-modal-vue'

export default {
  name: 'tasks',
  components: {
    SweetModalTab
  }
}
</script>


<template>
  <div class="screen-element-video"
    :style="{
      top: elementInfo.top + 'px',
      left: elementInfo.left + 'px',
      width: elementInfo.width + 'px',
      height: elementInfo.height + 'px',
    }"
  >
    <div
      class="rotate-wrapper"
      :style="{ transform: `rotate(${elementInfo.rotate}deg)` }"
    >
      <div class="element-content">
        <VideoPlayer
          :width="elementInfo.width"
          :height="elementInfo.height"
          :src="elementInfo.src" 
          :poster="elementInfo.poster"  
          :scale="scale" 
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, inject, PropType, Ref, ref } from 'vue'
import { PPTVideoElement } from '@/types/slides'

import VideoPlayer from './VideoPlayer/index.vue'

export default defineComponent({
  name: 'screen-element-video',
  components: {
    VideoPlayer,
  },
  props: {
    elementInfo: {
      type: Object as PropType<PPTVideoElement>,
      required: true,
    },
  },
  setup() {
    const scale: Ref<number> = inject('slideScale') || ref(1)

    return {
      scale,
    }
  },
})
</script>

<style lang="scss" scoped>
.screen-element-video {
  position: absolute;
}
.rotate-wrapper {
  width: 100%;
  height: 100%;
}
.element-content {
  width: 100%;
  height: 100%;
}
</style>

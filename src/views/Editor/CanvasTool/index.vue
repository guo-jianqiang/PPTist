<template>
  <div class="canvas-tool">
    <div class="left-handler">
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="撤销">
        <IconBack class="handler-item" :class="{ 'disable': !canUndo }" @click="undo()" />
      </Tooltip>
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="重做">
        <IconNext class="handler-item" :class="{ 'disable': !canRedo }" @click="redo()" />
      </Tooltip>
    </div>

    <div class="add-element-handler">
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入文字">
        <IconFontSize class="handler-item" @click="drawText()" />
      </Tooltip>
      <FileInput @change="files => insertImageElement(files)">
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入图片">
          <IconPicture class="handler-item" />
        </Tooltip>
      </FileInput>
      <Popover trigger="click" v-model:visible="shapePoolVisible">
        <template #content>
          <ShapePool @select="shape => drawShape(shape)" />
        </template>
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入形状">
          <IconGraphicDesign class="handler-item" />
        </Tooltip>
      </Popover>
      <Popover trigger="click" v-model:visible="linePoolVisible">
        <template #content>
          <LinePool @select="line => drawLine(line)" />
        </template>
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入线条">
          <IconConnection class="handler-item" />
        </Tooltip>
      </Popover>
      <Popover trigger="click" v-model:visible="chartPoolVisible">
        <template #content>
          <ChartPool @select="chart => { createChartElement(chart); chartPoolVisible = false }" />
        </template>
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入图表">
          <IconChartProportion class="handler-item" />
        </Tooltip>
      </Popover>
      <Popover trigger="click" v-model:visible="tableGeneratorVisible">
        <template #content>
          <TableGenerator
            @close="tableGeneratorVisible = false"
            @insert="({ row, col }) => { createTableElement(row, col); tableGeneratorVisible = false }"
          />
        </template>
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入表格">
          <IconInsertTable class="handler-item" />
        </Tooltip>
      </Popover>
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入公式">
        <IconFormula class="handler-item" @click="latexEditorVisible = true" />
      </Tooltip>
      <Popover trigger="click" v-model:visible="videoInputVisible">
        <template #content>
          <VideoInput 
            @close="videoInputVisible = false"
            @insert="src => { createVideoElement(src); videoInputVisible = false }"
          />
        </template>
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="插入视频">
          <IconVideoTwo class="handler-item" />
        </Tooltip>
      </Popover>
    </div>

    <div class="right-handler">
      <IconMinus class="handler-item viewport-size" @click="scaleCanvas('-')" />
      <span class="text">{{canvasScalePercentage}}</span>
      <IconPlus class="handler-item viewport-size" @click="scaleCanvas('+')" />
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="适配屏幕">
        <IconFullScreen class="handler-item viewport-size-adaptation" @click="setCanvasPercentage(90)" />
      </Tooltip>
    </div>

    <Modal
      v-model:visible="latexEditorVisible" 
      :footer="null" 
      centered
      :width="880"
      destroyOnClose
    >
      <LaTeXEditor 
        @close="latexEditorVisible = false"
        @update="data => { createLatexElement(data); latexEditorVisible = false }"
      />
    </Modal>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, ref } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore, useSnapshotStore } from '@/store'
import { getImageDataURL } from '@/utils/image'
import { ShapePoolItem } from '@/configs/shapes'
import { LinePoolItem } from '@/configs/lines'
import useScaleCanvas from '@/hooks/useScaleCanvas'
import useHistorySnapshot from '@/hooks/useHistorySnapshot'
import useCreateElement from '@/hooks/useCreateElement'

import ShapePool from './ShapePool.vue'
import LinePool from './LinePool.vue'
import ChartPool from './ChartPool.vue'
import TableGenerator from './TableGenerator.vue'
import VideoInput from './VideoInput.vue'
import LaTeXEditor from '@/components/LaTeXEditor/index.vue'

export default defineComponent({
  name: 'canvas-tool',
  components: {
    ShapePool,
    LinePool,
    ChartPool,
    TableGenerator,
    VideoInput,
    LaTeXEditor,
  },
  setup() {
    const mainStore = useMainStore()
    const { canvasScale } = storeToRefs(mainStore)
    const { canUndo, canRedo } = storeToRefs(useSnapshotStore())

    const canvasScalePercentage = computed(() => parseInt(canvasScale.value * 100 + '') + '%')

    const { scaleCanvas, setCanvasPercentage } = useScaleCanvas()
    const { redo, undo } = useHistorySnapshot()

    const { createImageElement, createChartElement, createTableElement, createLatexElement, createVideoElement } = useCreateElement()

    const insertImageElement = (files: File[]) => {
      const imageFile = files[0]
      if (!imageFile) return
      getImageDataURL(imageFile).then(dataURL => createImageElement(dataURL))
    }

    const shapePoolVisible = ref(false)
    const linePoolVisible = ref(false)
    const chartPoolVisible = ref(false)
    const tableGeneratorVisible = ref(false)
    const videoInputVisible = ref(false)
    const latexEditorVisible = ref(false)

    // 绘制文字范围
    const drawText = () => {
      mainStore.setCreatingElement({
        type: 'text',
      })
    }

    // 绘制形状范围
    const drawShape = (shape: ShapePoolItem) => {
      mainStore.setCreatingElement({
        type: 'shape',
        data: shape,
      })
      shapePoolVisible.value = false
    }

    // 绘制线条路径
    const drawLine = (line: LinePoolItem) => {
      mainStore.setCreatingElement({
        type: 'line',
        data: line,
      })
      linePoolVisible.value = false
    }

    return {
      scaleCanvas,
      setCanvasPercentage,
      canvasScalePercentage,
      canUndo,
      canRedo,
      redo,
      undo,
      insertImageElement,
      shapePoolVisible,
      linePoolVisible,
      chartPoolVisible,
      tableGeneratorVisible,
      videoInputVisible,
      latexEditorVisible,
      drawText,
      drawShape,
      drawLine,
      createChartElement,
      createTableElement,
      createLatexElement,
      createVideoElement,
    }
  },
})
</script>

<style lang="scss" scoped>
.canvas-tool {
  position: relative;
  border-bottom: 1px solid $borderColor;
  background-color: #fff;
  display: flex;
  justify-content: space-between;
  padding: 0 10px;
  font-size: 13px;
  user-select: none;
}
.left-handler {
  display: flex;
  align-items: center;
}
.add-element-handler {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
}
.handler-item {
  margin: 0 10px;
  font-size: 14px;
  cursor: pointer;

  &.disable {
    opacity: .5;
  }
}
.right-handler {
  display: flex;
  align-items: center;

  .text {
    width: 40px;
    text-align: center;
  }

  .viewport-size {
    font-size: 13px;
  }
}
</style>
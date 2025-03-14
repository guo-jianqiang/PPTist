<template>
  <div class="chart-style-panel">
    <Button class="full-width-btn" @click="chartDataEditorVisible = true">
      <IconEdit class="btn-icon" /> 编辑图表数据
    </Button>

    <Divider />

    <template v-if="handleElement.chartType === 'line'">
      <div class="row">
        <Checkbox 
          @change="e => updateOptions({ showArea: e.target.checked })"
          :checked="showArea" 
          style="flex: 1;"
        >面积图样式</Checkbox>
        <Checkbox 
          @change="e => updateOptions({ showLine: !e.target.checked })"
          :checked="!showLine" 
          style="flex: 1;"
        >散点图样式</Checkbox>
      </div>
      <div class="row">
        <Checkbox 
          @change="e => updateOptions({ lineSmooth: e.target.checked })" 
          :checked="lineSmooth"
        >使用平滑曲线</Checkbox>
      </div>
    </template>
    <div class="row" v-if="handleElement.chartType === 'bar'">
      <Checkbox 
        @change="e => updateOptions({ horizontalBars: e.target.checked })" 
        :checked="horizontalBars"
      >条形图样式</Checkbox>
    </div>
    <div class="row" v-if="handleElement.chartType === 'pie'">
      <Checkbox 
        @change="e => updateOptions({ donut: e.target.checked })" 
        :checked="donut"
      >环形图样式</Checkbox>
    </div>

    <Divider />

    <div class="row">
      <div style="flex: 2;">图例：</div>
      <Select style="flex: 3;" :value="legend" @change="value => updateLegend(value)">
        <SelectOption value="">不显示</SelectOption>
        <SelectOption value="top">显示在上方</SelectOption>
        <SelectOption value="bottom">显示在下方</SelectOption>
      </Select>
    </div>

    <Divider />

    <div class="row">
      <div style="flex: 2;">背景填充：</div>
      <Popover trigger="click">
        <template #content>
          <ColorPicker
            :modelValue="fill"
            @update:modelValue="value => updateFill(value)"
          />
        </template>
        <ColorButton :color="fill" style="flex: 3;" />
      </Popover>
    </div>
    <div class="row">
      <div style="flex: 2;">网格颜色：</div>
      <Popover trigger="click">
        <template #content>
          <ColorPicker
            :modelValue="gridColor"
            @update:modelValue="value => updateGridColor(value)"
          />
        </template>
        <ColorButton :color="gridColor" style="flex: 3;" />
      </Popover>
    </div>

    <Divider />

    <div class="row" v-for="(color, index) in themeColor" :key="index">
      <div style="flex: 2;">{{index === 0 ? '主题配色：' : ''}}</div>
      <Popover trigger="click">
        <template #content>
          <ColorPicker
            :modelValue="color"
            @update:modelValue="value => updateTheme(value, index)"
          />
        </template>
        <div class="color-btn-wrap" style="flex: 3;">
          <ColorButton :color="color" style="width: 100%;" />
          <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="删除">
            <div class="delete-color-btn" @click.stop="deleteThemeColor(index)" v-if="index !== 0"><IconCloseSmall /></div>
          </Tooltip>
        </div>
      </Popover>
    </div>
    <div class="row" v-if="themeColor.length < 10">
      <div style="flex: 2;"></div>
      <Button class="add-color-btn" style="flex: 3;" @click="addThemeColor()">
        <IconPlus />
      </Button>  
    </div>

    <Divider />

    <ElementOutline />

    <Modal
      v-model:visible="chartDataEditorVisible" 
      :footer="null" 
      centered
      :closable="false"
      :width="648"
      destroyOnClose
    >
      <ChartDataEditor 
        :data="handleElement.data"
        @close="chartDataEditorVisible = false"
        @save="value => updateData(value)"
      />
    </Modal>
  </div>
</template>

<script lang="ts">
import { defineComponent, onUnmounted, ref, watch } from 'vue'
import { IBarChartOptions, ILineChartOptions, IPieChartOptions } from 'chartist'
import { storeToRefs } from 'pinia'
import { useMainStore, useSlidesStore } from '@/store'
import { ChartData, PPTChartElement } from '@/types/slides'
import emitter, { EmitterEvents } from '@/utils/emitter'
import useHistorySnapshot from '@/hooks/useHistorySnapshot'

import ElementOutline from '../../common/ElementOutline.vue'
import ColorButton from '../../common/ColorButton.vue'
import ChartDataEditor from './ChartDataEditor.vue'

export default defineComponent({
  name: 'chart-style-panel',
  components: {
    ElementOutline,
    ChartDataEditor,
    ColorButton,
  },
  setup() {
    const mainStore = useMainStore()
    const slidesStore = useSlidesStore()
    const { handleElement, handleElementId } = storeToRefs(mainStore)
    const { theme } = storeToRefs(slidesStore)

    const chartDataEditorVisible = ref(false)

    const { addHistorySnapshot } = useHistorySnapshot()

    const fill = ref<string>()

    const themeColor = ref<string[]>([])
    const gridColor = ref('')
    const legend = ref('')

    const lineSmooth = ref(true)
    const showLine = ref(true)
    const showArea = ref(false)
    const horizontalBars = ref(false)
    const donut = ref(false)

    watch(handleElement, () => {
      if (!handleElement.value || handleElement.value.type !== 'chart') return
      fill.value = handleElement.value.fill || '#000'

      if (handleElement.value.options) {
        const {
          lineSmooth: _lineSmooth,
          showLine: _showLine,
          showArea: _showArea,
          horizontalBars: _horizontalBars,
          donut: _donut,
        } = handleElement.value.options

        if (_lineSmooth !== undefined) lineSmooth.value = _lineSmooth as boolean
        if (_showLine !== undefined) showLine.value = _showLine
        if (_showArea !== undefined) showArea.value = _showArea
        if (_horizontalBars !== undefined) horizontalBars.value = _horizontalBars
        if (_donut !== undefined) donut.value = _donut
      }

      themeColor.value = handleElement.value.themeColor
      gridColor.value = handleElement.value.gridColor || 'rgba(0, 0, 0, 0.4)'
      legend.value = handleElement.value.legend || ''
    }, { deep: true, immediate: true })

    const updateElement = (props: Partial<PPTChartElement>) => {
      slidesStore.updateElement({ id: handleElementId.value, props })
      addHistorySnapshot()
    }

    // 设置图表数据
    const updateData = (data: ChartData) => {
      chartDataEditorVisible.value = false
      updateElement({ data })
    }

    // 设置填充色
    const updateFill = (value: string) => {
      updateElement({ fill: value })
    }

    // 设置其他选项：柱状图转条形图、折线图转面积图、折线图转散点图、饼图转环形图、折线图开关平滑曲线
    const updateOptions = (optionProps: ILineChartOptions & IBarChartOptions & IPieChartOptions) => {
      const _handleElement = handleElement.value as PPTChartElement

      const newOptions = { ..._handleElement.options, ...optionProps }
      updateElement({ options: newOptions })
    }

    // 设置主题色
    const updateTheme = (color: string, index: number) => {
      const props = {
        themeColor: themeColor.value.map((c, i) => i === index ? color : c),
      }
      updateElement(props)
    }

    // 添加主题色
    const addThemeColor = () => {
      const props = {
        themeColor: [...themeColor.value, theme.value.themeColor],
      }
      updateElement(props)
    }

    // 删除主题色
    const deleteThemeColor = (index: number) => {
      const props = {
        themeColor: themeColor.value.filter((c, i) => i !== index),
      }
      updateElement(props)
    }

    // 设置网格颜色
    const updateGridColor = (gridColor: string) => {
      updateElement({ gridColor })
    }

    // 设置图例位置/不显示
    const updateLegend = (legend: '' | 'top' | 'bottom') => {
      updateElement({ legend })
    }

    const openDataEditor = () => chartDataEditorVisible.value = true

    emitter.on(EmitterEvents.OPEN_CHART_DATA_EDITOR, openDataEditor)
    onUnmounted(() => {
      emitter.off(EmitterEvents.OPEN_CHART_DATA_EDITOR, openDataEditor)
    })

    return {
      chartDataEditorVisible,
      handleElement,
      updateData,
      fill,
      updateFill,
      lineSmooth,
      showLine,
      showArea,
      horizontalBars,
      donut,
      updateOptions,
      themeColor,
      gridColor,
      legend,
      updateTheme,
      addThemeColor,
      deleteThemeColor,
      updateGridColor,
      updateLegend,
    }
  },
})
</script>

<style lang="scss" scoped>
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.full-width-btn {
  width: 100%;
}
.btn-icon {
  margin-right: 3px;
}

.add-color-btn {
  padding: 0 !important;
}
.color-btn-wrap {
  position: relative;
}
.delete-color-btn {
  position: absolute;
  width: 30px;
  right: 2px;
  top: 2px;
  bottom: 2px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #fff;
  cursor: pointer;
}
</style>
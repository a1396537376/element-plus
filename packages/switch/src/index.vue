<template>
  <div
    class="el-switch"
    :class="{ 'is-disabled': switchDisabled, 'is-checked': checked }"
    role="switch"
    :aria-checked="checked"
    :aria-disabled="switchDisabled"
    @click.prevent="switchValue"
  >
    <input
      :id="id"
      ref="input"
      class="el-switch__input"
      type="checkbox"
      :name="name"
      :true-value="activeValue"
      :false-value="inactiveValue"
      :disabled="switchDisabled"
      @change="handleChange"
      @keydown.enter="switchValue"
    >
    <span
      v-if="inactiveIconClass || inactiveText"
      :class="['el-switch__label', 'el-switch__label--left', !checked ? 'is-active' : '']"
    >
      <i v-if="inactiveIconClass" :class="[inactiveIconClass]"></i>
      <span v-if="!inactiveIconClass && inactiveText" :aria-hidden="checked">{{ inactiveText }}</span>
    </span>
    <span ref="core" class="el-switch__core" :style="{ 'width': coreWidth + 'px' }">
      <div class="el-switch__action">
        <i v-if="loading" class="el-icon-loading"></i>
      </div>
    </span>
    <span
      v-if="activeIconClass || activeText"
      :class="['el-switch__label', 'el-switch__label--right', checked ? 'is-active' : '']"
    >
      <i v-if="activeIconClass" :class="[activeIconClass]"></i>
      <span v-if="!activeIconClass && activeText" :aria-hidden="!checked">{{ activeText }}</span>
    </span>
  </div>
</template>
<script lang='ts'>
import { defineComponent, computed, onMounted, ref, inject, nextTick, watch } from 'vue'
import { elFormKey, elFormItemKey } from '@element-plus/form'

import type { ElFormContext, ElFormItemContext } from '@element-plus/form'


type ValueType = boolean | string | number;

interface ISwitchProps {
  modelValue: ValueType
  value: ValueType
  disabled: boolean
  width: number
  activeIconClass: string
  inactiveIconClass: string
  activeText: string
  inactiveText: string
  activeColor: string
  inactiveColor: string
  activeValue: ValueType
  inactiveValue: ValueType
  name: string
  validateEvent: boolean
  id: string
  loading:boolean
}

export default defineComponent({
  name: 'ElSwitch',
  props: {
    modelValue: {
      type: [Boolean, String, Number],
      default: false,
    },
    value: {
      type: [Boolean, String, Number],
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    width: {
      type: Number,
      default: 40,
    },
    activeIconClass: {
      type: String,
      default: '',
    },
    inactiveIconClass: {
      type: String,
      default: '',
    },
    activeText: {
      type: String,
      default: '',
    },
    inactiveText: {
      type: String,
      default: '',
    },
    activeColor: {
      type: String,
      default: '',
    },
    inactiveColor: {
      type: String,
      default: '',
    },
    activeValue: {
      type: [Boolean, String, Number],
      default: true,
    },
    inactiveValue: {
      type: [Boolean, String, Number],
      default: false,
    },
    name: {
      type: String,
      default: '',
    },
    validateEvent: {
      type: Boolean,
      default: true,
    },
    id: {
      type: String,
      default: '',
    },
    loading:{
      type: Boolean,
      default: false,
    },
  },
  emits: ['update:modelValue', 'change', 'input'],
  setup(props: ISwitchProps, ctx) {
    const elForm = inject(elFormKey, {} as ElFormContext)
    const elFormItem = inject(elFormItemKey, {} as ElFormItemContext)

    const coreWidth = ref(props.width)
    const isModelValue = ref(props.modelValue !== false)
    const input = ref(null)
    const core = ref(null)

    watch(() => props.modelValue, () => {
      isModelValue.value = true
    })

    watch(() => props.value, () => {
      isModelValue.value = false
    })

    const actualValue = computed((): ValueType => {
      return isModelValue.value ? props.modelValue : props.value
    })

    const checked = computed((): boolean => {
      return actualValue.value === props.activeValue
    })

    if (!~[props.activeValue, props.inactiveValue].indexOf(actualValue.value)) {
      ctx.emit('update:modelValue', props.inactiveValue)
      ctx.emit('change', props.inactiveValue)
      ctx.emit('input', props.inactiveValue)
    }

    watch(checked, () => {
      input.value.checked = checked.value

      if (props.activeColor || props.inactiveColor) {
        setBackgroundColor()
      }

      if (props.validateEvent) {
        elFormItem.formItemMitt?.emit('el.form.change', [actualValue.value])
      }
    })

    const switchDisabled = computed((): boolean => {
      return props.disabled || props.loading ||(elForm || {}).disabled
    })

    const handleChange = (): void => {
      const val = checked.value ? props.inactiveValue : props.activeValue
      ctx.emit('update:modelValue', val)
      ctx.emit('change', val)
      ctx.emit('input', val)
      nextTick(() => {
        input.value.checked = checked.value
      })
    }

    const switchValue = (): void => {
      !switchDisabled.value && handleChange()
    }

    const setBackgroundColor = (): void => {
      const newColor = checked.value ? props.activeColor : props.inactiveColor
      const coreEl = core.value
      coreEl.style.borderColor = newColor
      coreEl.style.backgroundColor = newColor
      coreEl.children[0].style.color = newColor
    }

    onMounted(() => {
      coreWidth.value = coreWidth.value || 40

      if (props.activeValue || props.inactiveValue) {
        setBackgroundColor()
      }

      input.value.checked = checked.value
    })

    return {
      input,
      core,
      coreWidth,
      switchDisabled,
      checked,
      handleChange,
      switchValue,
    }
  },
})
</script>



<script lang="ts" setup>
import { flip, offset, shift } from '@floating-ui/vue'
import type { ASelectEvents, aSelectSlots } from './meta'
import { aSelectBaseInputSlots, aSelectCardSlots, aSelectListSlotsWithPrefixMeta, aSelectProps } from './meta'
import { ACard, AList } from '@/components'
import { ABaseInput, aBaseInputProps } from '@/components/base-input'
import { AFloating, sameWidthFloatingUIMiddleware } from '@/components/floating'
import type { AListPropItems } from '@/components/list'
import { extractItemValueFromItemOption } from '@/composables/useSelection'
import { isObject } from '@/utils/helpers'

export interface ObjectOption { label: string; value: string | number }

const props = defineProps(aSelectProps)
const emit = defineEmits<ASelectEvents>()
defineSlots<typeof aSelectSlots>()

defineOptions({
  name: 'ASelect',
  inheritAttrs: false,
})

// const _baseInputProps = reactivePick(props, Object.keys(aBaseInputProps) as Array<keyof ABaseInputProps>)
const _baseInputProps = reactivePick(props, Object.keys(aBaseInputProps) as any)

// SECTION Floating
// Template refs
const refReference = ref()
const refFloating = ref()
const selectRef = ref<HTMLSelectElement>()

const isOptionsVisible = ref(false)

onClickOutside(
  refFloating,
  _event => {
    if (isOptionsVisible.value)
      isOptionsVisible.value = false
  },
  {
    ignore: [refReference],
  },
)

// !SECTION
// TODO: You can use it as utility in another components
// TODO: Add some style to indicate currently selected item
function handleInputClick() {
  if (!(props.disabled || props.readonly)) {
    isOptionsVisible.value = !isOptionsVisible.value
    selectRef.value?.focus()
  }
}

// 👉 Options
function handleOptionClick(item: AListPropItems[number]) {
  const valueToEmit = props.emitObject ? item : extractItemValueFromItemOption(item)

  emit('change', valueToEmit)
  emit('input', valueToEmit)
  emit('update:modelValue', valueToEmit)
}

function closeOptions(event: MouseEvent) {
  if (event.target !== refFloating.value)
    isOptionsVisible.value = false
}

// 👉 Middleware
function middleware() {
  return [
    offset(6),
    sameWidthFloatingUIMiddleware(refFloating),
    flip(),
    shift({ padding: 10 }),
  ]
}
</script>

<template>
  <!-- 👉 Select Input -->
  <ABaseInput
    v-bind="{
      ..._baseInputProps,
      inputWrapperClasses: ['!cursor-pointer', _baseInputProps.inputWrapperClasses],
      class: $attrs.class,
    }"
    ref="refReference"
    append-inner-icon="i-bx-chevron-down"
    class="a-select"
    :input-wrapper-attrs="{
      onClick: handleInputClick,
    }"
  >
    <!-- ℹ️ Recursively pass down slots to child -->
    <template
      v-for="(_, name) in aSelectBaseInputSlots"
      #[name]="slotProps"
    >
      <slot
        :name="name"
        v-bind="slotProps"
      />
    </template>
    <template #default="slotProps">
      <input
        v-bind="{ ...$attrs, ...slotProps }"
        ref="selectRef"
        readonly
        class="a-select-input cursor-pointer"
        :value="isObject(modelValue) ? modelValue.text : modelValue"
      >
    </template>
  </ABaseInput>

  <!-- 👉 Select options -->
  <AFloating
    :reference-el="refReference && refReference.refInputContainer"
    :middleware="middleware"
    class="a-select-floating"
  >
    <ACard
      v-show="isOptionsVisible"
      ref="refFloating"
      class="a-select-options-container bg-[hsl(var(--a-surface-c))]"
      :class="props.optionsWrapperClasses"
      @click="closeOptions"
    >
      <!-- ℹ️ Recursively pass down slots to child -->
      <template
        v-for="(_, name) in aSelectCardSlots"
        #[name]="slotProps"
      >
        <slot
          :name="name"
          v-bind="slotProps"
        />
      </template>
      <AList
        :items="options"
        :model-value="props.modelValue"
        :emit-object="props.emitObject"
        class="a-select-options-list"
        :class="props.listClasses"
        @click:item="(item) => handleOptionClick(item)"
      >
        <!-- ℹ️ Recursively pass down slots to child -->
        <template
          v-for="{ originalKey: originalSlotName, prefixedKey: updatedSlotName } in aSelectListSlotsWithPrefixMeta"
          #[originalSlotName]="slotProps"
        >
          <slot
            :name="updatedSlotName"
            v-bind="{
              ...slotProps,
              handleOptionClick,
            }"
          />
        </template>
      </AList>
    </ACard>
  </AFloating>
</template>

<template>
  <TabGroup
    as="div"
    class="flex flex-1 flex-col overflow-y-hidden"
    :style="containerStyle"
    :defaultIndex="changedIndex"
    :selectedIndex="changedIndex"
    @change="(idx) => (changedIndex = idx)"
  >
    <TabList
      ref="tabListRef"
      class="relative flex items-center gap-7.5 overflow-x-auto border-b px-5"
      :class="tablistClass"
    >
      <Tab
        ref="tabRef"
        as="template"
        v-for="(tab, i) in tabs"
        :key="i"
        v-slot="{ selected }"
        class="focus:outline-none focus:transition-none"
      >
        <slot name="tab" v-bind="{ tab, selected }">
          <button
            class="flex items-center gap-1.5 border-b border-transparent py-3 text-base text-gray-600 duration-300 ease-in-out hover:border-gray-400 hover:text-gray-900"
            :class="{ 'text-gray-900': selected }"
          >
            <component v-if="tab.icon" :is="tab.icon" class="size-4" />
            {{ tab.label }}
          </button>
        </slot>
      </Tab>
      <div
        ref="indicator"
        class="tab-indicator absolute bottom-0 h-px bg-gray-900"
        :class="transitionClass"
      />
    </TabList>
    <TabPanels class="flex flex-1 overflow-hidden" :class="tabPanelClass">
      <TabPanel
        class="flex flex-1 flex-col overflow-y-auto focus:outline-none"
        v-for="(tab, i) in tabs"
        :key="i"
      >
        <slot v-bind="{ tab }" />
      </TabPanel>
    </TabPanels>
  </TabGroup>
</template>

<script setup>
import { TabGroup, TabList, Tab, TabPanels, TabPanel } from '@headlessui/vue'
import { ref, watch, computed, onMounted, nextTick } from 'vue'

const props = defineProps({
  tabs: {
    type: Array,
    required: true,
  },
  modelValue: {
    type: Number,
    default: 0,
  },
  tablistClass: {
    type: String,
    default: '',
  },
  tabPanelClass: {
    type: String,
    default: '',
  },
})

const emit = defineEmits(['update:modelValue'])

const changedIndex = computed({
  get: () => props.modelValue,
  set: (index) => emit('update:modelValue', index),
})

const tabListRef = ref(null)
const tabRef = ref([])
const indicator = ref(null)
const tabsLength = computed(() => props.tabs?.length)

const transitionClass = ref('')

const containerStyle = computed(() => {
  let headerHeight = 0
  if (tabListRef.value) {
    headerHeight = tabListRef.value.offsetTop
  }
  return `height: calc(100vh - ${headerHeight}px)`
})

function moveIndicator(index) {
  if (index >= tabsLength.value) {
    index = tabsLength.value - 1
  }
  const selectedTab = tabRef.value[index].el
  indicator.value.style.width = `${selectedTab.offsetWidth}px`
  indicator.value.style.left = `${selectedTab.offsetLeft}px`
}

watch(changedIndex, (index) => {
  if (index >= tabsLength.value) {
    changedIndex.value = tabsLength.value - 1
  }
  transitionClass.value = 'transition-all duration-300 ease-in-out'
  nextTick(() => moveIndicator(index))
})

onMounted(() => {
  nextTick(() => moveIndicator(changedIndex.value))
  // Fix for indicator not moving on initial load
  setTimeout(() => moveIndicator(changedIndex.value), 100)
})
</script>

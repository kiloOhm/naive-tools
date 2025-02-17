<template>
  <div class="n-popup-provider" :class="{ 'dark-mode': darkMode }">
    <div class="app-wrapper" :class="{ 'unfocused': popups.length >= 1 }">
      <slot name="default" />
    </div>
    <div class="popups-wrapper">
      <transition-group name="popup">
        <n-popup-wrapper v-for="({ slotNodes, slotNodesTitle, slotNodesActions, slotNodesCloseBtn, slotNodesHeader, id, type, transculent, fixedHeight, fixedWidth }, i) in popups" :id="id" :key="i" :level="popups.length - 1 - i" :type="type" :transculent="transculent" :fixed-height="fixedHeight" :fixed-width="fixedWidth" :slot-nodes="slotNodes" :slot-nodes-title="slotNodesTitle" :slot-nodes-actions="slotNodesActions" :slot-nodes-close-btn="slotNodesCloseBtn" :slot-nodes-header="slotNodesHeader" :pull-down-tolerance="15" @close="leavePopup(id)">
          <template v-if="slotNodesTitle" #title>
            <component :is="slotNode" v-for="(slotNode, n) in slotNodesTitle" :key="n" />
          </template>
          <template v-if="slotNodesActions" #actions>
            <component :is="slotNode" v-for="(slotNode, n) in slotNodesActions" :key="n" />
          </template>
          <template v-if="slotNodesCloseBtn" #close-btn>
            <component :is="slotNode" v-for="(slotNode, n) in slotNodesCloseBtn" :key="n" />
          </template>
          <template v-if="slotNodesHeader" #header>
            <component :is="slotNode" v-for="(slotNode, n) in slotNodesHeader" :key="n" />
          </template>
        </n-popup-wrapper>
      </transition-group>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, provide, VNode, computed, Ref, ComputedRef } from 'vue';
import isDarkMode from '../../util/isDarkMode';
import NPopupWrapper from './NPopupWrapper.vue';

export type VueSlot = () => VNode[];

export type NPopupWrappedDescriptorWrapped = {
  id: string;
  slotNodes: ComputedRef<Ref<VNode[]>>;
  slotNodesTitle?: ComputedRef<Ref<VNode[] | undefined>>;
  slotNodesCloseBtn?: ComputedRef<Ref<VNode[] | undefined>>;
  slotNodesActions?: ComputedRef<Ref<VNode[] | undefined>>;
  slotNodesHeader?: ComputedRef<Ref<VNode[] | undefined>>;
  title?: string;
  level?: number;
  type: 'layer' | 'frame';
  transculent?: boolean;
  fixedHeight?: boolean | number;
  fixedWidth?: boolean | number;
}

const darkMode = isDarkMode();

const popups = ref<NPopupWrappedDescriptorWrapped[]>([]);
function showPopup({ id, type, transculent, fixedHeight, fixedWidth, slotTitle, slotActions, slotCloseBtn, slotHeader, slot }: { id: string, type: 'layer' | 'frame', transculent: boolean, fixedHeight: boolean | number, fixedWidth: boolean | number, slot: VueSlot, slotTitle?: VueSlot, slotActions?: VueSlot, slotCloseBtn?: VueSlot, slotHeader?: VueSlot }) {
  popups.value = [
    ...popups.value,
    {
      id,
      slotNodes: computed(() => slot()),
      slotNodesTitle: computed(() => slotTitle ? slotTitle() : undefined),
      slotNodesActions: computed(() => slotActions ? slotActions() : undefined),
      slotNodesCloseBtn: computed(() => slotCloseBtn ? slotCloseBtn() : undefined),
      slotNodesHeader: computed(() => slotHeader ? slotHeader() : undefined),
      type,
      transculent,
      fixedHeight,
      fixedWidth
    },

  ];
  
}
function leavePopup(popupId: string) {
  popups.value = popups.value.filter(({ id }) => id !== popupId);
}
function getPopupIndex(popupId: string) {
  const popupDescriptor = popups.value.find(({ id }) => id === popupId);
  return popupDescriptor ? popups.value.indexOf(popupDescriptor) : -1;
}
provide('showPopup', showPopup);
provide('leavePopup', leavePopup);
provide('getPopupIndex', getPopupIndex);


</script>
<style scoped lang="scss">
.n-popup-provider {
  position: relative;
  --transition-duration: .4s;
  height: 100%;
  --offset-top-safe-area: env(safe-area-inset-top);
  .app-wrapper {
    height: 100%;
    transform-origin: center center;
    transition: transform calc(1.2 * var(--transition-duration)), filter var(--transition-duration);
  }
  .app-wrapper.unfocused {
    filter: brightness(0.9);
    border-radius: 10px;
    clip-path: inset(0 0 0 0 round 10px);
    -webkit-clip-path: inset(0 0 0 0 round 10px);
    overflow: hidden;
    transform: scale(0.9) translate(0, calc(-3% - var(--offset-top-safe-area)));
  }
  .popups-wrapper {
    width: 100%;
    height: 100%;
    position: fixed;
    pointer-events: none;
    left: 0;
    top: 0;
  }
}
.n-popup-provider.dark-mode {
  .app-wrapper.unfocused {
    filter: brightness(1.2);
  }
}
</style>
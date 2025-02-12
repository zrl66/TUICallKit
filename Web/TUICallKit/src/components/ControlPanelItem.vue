<script setup lang="ts">
import { ref, onMounted, nextTick, withDefaults, toRefs, defineProps, computed } from "vue";
import TriangleSVG from "../icons/triangle.vue";

const openDetail = ref<boolean>(false);
const props = withDefaults(
  defineProps<{
    action: (payload: MouseEvent) => void;
    hasDetail?: boolean;
    size?: string;
    isMobile?: boolean;
  }>(),
  {
    hasDetail: false,
    size: 'small',
    isMobile: false
  }
);
const { action, hasDetail, isMobile, size } = toRefs(props);
const controlItemIconContainerClass = computed(() => `control-item-icon-container-${size.value}`)

onMounted(() => {
  document.documentElement.addEventListener('click', (e) => {
    openDetail.value = false;
  });
});

async function stayFocus() {
  await nextTick();
  const els = document.getElementsByClassName("control-item-detail");
  for (let i = 0; i < els.length; i++) {
    els[i].addEventListener('click', (e: any) => e.stopPropagation());
  }
}

async function toggleDetail(event: Event) {
  event.stopPropagation();
  stayFocus();
  openDetail.value = !openDetail.value;
};
</script>

<template>
  <div :class="isMobile ? 'control-item-h5' : 'control-item'">
    <div class="control-item-icon">
      <div :class="controlItemIconContainerClass" @click="action">
        <slot name="icon"> </slot>
      </div>
      <div class="control-item-summary" @click="toggleDetail" v-if="hasDetail">
        <TriangleSVG />
      </div>
      <div class="control-item-detail" v-show="openDetail">
        <slot name="detail"> </slot>
      </div>
    </div>
    <div class="control-item-text">
      <slot name="text"> </slot>
    </div>
  </div>
</template>

<style scoped>
@import "../style.css";
</style>
<script setup lang="ts">
import { profile, status, isMinimized } from "../store";
import { STATUS } from "../constants";
import { TUICallKitServer } from "../index";
import FloatCallEnd from "../icons/floatCallEnd.vue";
import FloatMicrophoneSVG from "../icons/floatMicrophone.vue";
import FloatMicrophoneClosedSVG from "../icons/floatMicrophoneClosed.vue";
import FloatFullScreenSVG from "../icons/floatFullScreen.vue";

function toggleMinimize() {
  TUICallKitServer.toggleMinimize();
}

async function hangup() {
  await TUICallKitServer.hangup();
}

async function toggleMicrophone() {
  if (profile.value?.microphone) {
    await TUICallKitServer.closeMicrophone();
  } else {
    await TUICallKitServer.openMicrophone();
  }
};
</script>

<template>
  <div
    class="float-control-panel"
    v-show="isMinimized"
    v-if="status !== STATUS.IDLE"
  >
    <div class="float-control-item-icon">
      <div class="float-control-item-icon-container" @click="hangup">
        <FloatCallEnd />
      </div>
    </div>
    <div class="float-control-item-icon">
      <div class="float-control-item-icon-container" @click="toggleMicrophone">
        <FloatMicrophoneSVG v-if="profile?.microphone" />
        <FloatMicrophoneClosedSVG v-if="!profile?.microphone" />
      </div>
    </div>
    <div class="float-control-item-icon">
      <div class="float-control-item-icon-container" @click="toggleMinimize">
        <FloatFullScreenSVG />
      </div>
    </div>
  </div>
</template>

<style scoped>
@import "../style.css";
</style>
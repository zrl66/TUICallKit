<script setup lang="ts">
import { isFromGroup, remoteList, callType, profile, t, getVolumeByUserID, status } from '../store/index';
import { ref, watch, watchEffect, onMounted } from "vue";
import MicrophoneIcon from "./MicrophoneIcon.vue";
import { TUICallKitServer } from '../index';
import type { RemoteUser } from "../interface";
import { CALL_TYPE_STRING, STATUS } from '../constants';
import LeftSVG from "../icons/left.vue";
import RightSVG from "../icons/right.vue";
import MicrophoneClosedSVG from '../icons/microphoneClosed.vue';

const currentPage = ref<number>(1);
const currentPageRemoteList = ref<RemoteUser[]>([]);
const groupUserViewClass = ref<string>("group-user-view");
const groupCallingContainerClass = ref<string>("group-calling-container");
const userViewUserId = ref<string>("user-view-user-id");

onMounted(async () => {
  if (status.value === STATUS.CALLING_GROUP_VIDEO || status.value === STATUS.CALLING_GROUP_AUDIO) {
    await TUICallKitServer.renderLocal();
  }
  await TUICallKitServer.renderRemoteWaitList();
});

watch(currentPageRemoteList, async () => {
  const userViewCount = currentPageRemoteList.value.length + 1;
  groupUserViewClass.value = `group-user-view group-user-view-${userViewCount}`;
  groupCallingContainerClass.value = `group-calling-container group-calling-container-${userViewCount}`;
  userViewUserId.value = `user-view-user-id user-view-user-id-${userViewCount}`;
  TUICallKitServer.renderRemoteList(currentPageRemoteList.value);
}, { flush: 'post', deep: true });

watchEffect(async () => {
  await refreshCurrentPageRemoteList();
});

async function refreshCurrentPageRemoteList() {
  let newPageList: RemoteUser[] = [];
  for (let i = (currentPage.value - 1) * 8; i < (currentPage.value) * 8 && i < remoteList.value.length; i++) {
    newPageList.push(remoteList.value[i]);
  }
  currentPageRemoteList.value = newPageList;
}

function pageReduce() {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
}

function pageIncrease() {
  if (currentPage.value < Math.floor((remoteList.value.length - 1) / 8 + 1)) {
    currentPage.value++;
  }
}

</script>

<template>
  <div class="calling-wrapper">
    <template v-if="remoteList.length > 8">
      <div class="page-turn left" @click="pageReduce">
        <div class="turn-icon-container">
          <LeftSVG />
        </div>
      </div>
      <div class="page-turn right" @click="pageIncrease">
        <div class="turn-icon-container">
          <RightSVG />
        </div>
      </div>
    </template>
    <div :class="groupCallingContainerClass">
      <div id="local-group" :class="groupUserViewClass">
        <span class="tag">
          <div class="microphone-icon-container">
            <MicrophoneIcon :volume="getVolumeByUserID(profile.userID)" v-if="profile?.microphone" />
            <MicrophoneClosedSVG v-else />
          </div>
          {{ `${profile?.nick} ${t('me')}` }}
        </span>
      </div>
        <div :class="groupUserViewClass" :id='remoteUserItem.userID' v-for='remoteUserItem in currentPageRemoteList' :key='remoteUserItem.userID'>
          <template v-if="!remoteUserItem.isEntered || (isFromGroup && callType === CALL_TYPE_STRING.AUDIO)">
            <div class="user-view-text-container">
              <div :class="userViewUserId"> {{ remoteUserItem?.nick }} </div>
              <div class="user-view-info"> {{ remoteUserItem.isEntered ? t('already-enter') : t('waiting') }} </div>
            </div>
          </template>
          <span class="tag">
            <div class="microphone-icon-container">
              <MicrophoneIcon :volume="getVolumeByUserID(remoteUserItem?.userID)" v-if="remoteUserItem?.microphone" />
              <MicrophoneClosedSVG v-else />
            </div>
            {{ remoteUserItem?.nick }}
          </span>
        </div>
    </div>
  </div>
</template>

<style scoped>
@import "../style.css";
</style>
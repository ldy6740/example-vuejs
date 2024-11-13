<script setup>
  import { ref, defineProps } from "vue";
  // import BreakSettingView from "./tabs/BreakSettingView.vue";
  import AccelSettingView from "./tabs/AccelSettingView.vue";

  const props = defineProps({
    settingValue: Array
  });

</script>

<template>
  <section class="setting-view-box">
    <!-- <ul class="tab-menu">
      <li
        v-for="(tab, index) in TAB_TITLE"
        :key="index"
        :class="['tab-title', {active: tabView.__file.slice(46, 62) === tab}]"
        @click="viewChange(tab)"
      >
        {{ tab === 'BreakSettingView'? "브레이크" : "감속" }}
      </li>
    </ul> -->

    <AccelSettingView @close-event="$emit('closeEvent')" :settingValue="settingValue"></AccelSettingView>
  </section>
</template>

<style lang="scss">
  .setting-view-box {
    background: #fff;
    border:1px solid #ededed;
    box-shadow: 1px 1px 20px 1px #8a8a8a;
    position: absolute;
    top: 140px;
    left: 10px;
    z-index: 9999;
    width: 380px;
    border-radius: 10px;
    padding: 20px;
    .tab-menu {
      display: flex;
      .tab-title {
        &.active {
          // box-shadow: 0 0 0 1px #0063c0 inset;
          background: #666666;
          color: #fff;
        }
        display: flex;
        align-items: center;
        justify-content: center;
        row-gap: 10px;
        margin-right: 10px;
        width: 90px;
        height: 30px;
        padding: 5px 10px;
        border-radius: 5px;
        background: #ededed;
        cursor: pointer;
      }
    }
  }

</style>

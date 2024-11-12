<script setup>
  import { ref, markRaw } from "vue";
  import BreakSettingView from "./tabs/BreakSettingView.vue";
  import AccelSettingView from "./tabs/AccelSettingView.vue";

  const TAB_TITLE = ref(["BreakSettingView", "AccelSettingView"]);
  const tabView = ref(markRaw(BreakSettingView));

  console.log(tabView.value.__file.slice(46, 62));

  /**
   * 동적으로 컴포넌트 명을 받아 해당 컴포넌트로 화면 변경
   * @param tab component name "String"
   */
  function viewChange(tab) {
    switch(tab) {
      case 'BreakSettingView':
        tabView.value = markRaw(BreakSettingView)
        break;
      case 'AccelSettingView':
        tabView.value = markRaw(AccelSettingView)
        break;
      default:
        alert('error');
    }
  }
</script>

<template>
  <section class="setting-view-box">
    <ul>
      <li
        v-for="(tab, index) in TAB_TITLE"
        :key="index"
        :class="['tab-title', {active: tabView.__file.slice(46, 62) === tab}]"
        @click="viewChange(tab)"
      >
        {{ tab === 'BreakSettingView'? "브레이크" : "감속" }}
      </li>
    </ul>
    <component :is="tabView"></component>
  </section>
</template>

<style lang="scss">
  .tab-title {
    &.active {
      border:1px solid red;
    }
  }
</style>

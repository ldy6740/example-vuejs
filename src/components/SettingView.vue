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
    <ul class="tab-menu">
      <li
        v-for="(tab, index) in TAB_TITLE"
        :key="index"
        :class="['tab-title', {active: tabView.__file.slice(46, 62) === tab}]"
        @click="viewChange(tab)"
      >
        {{ tab === 'BreakSettingView'? "브레이크" : "감속" }}
      </li>
      <li class="tab-title" @click="$emit('closeEvent')">닫기</li>
    </ul>

    <component :is="tabView" @save-event="$emit('closeEvent')"></component>
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

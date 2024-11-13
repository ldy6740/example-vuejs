<script setup>
  import { ref } from "vue";

  const strongMin     = ref("0");
  const strongMax     = ref("0");
  const strongOpcity  = ref("0");

  const mediumMin     = ref("0");
  const mediumMax     = ref("0");
  const mediumOpcity  = ref("0");

  const weakMin       = ref("0");
  const weakMax       = ref("0");
  const weakOpcity    = ref("0");

  const settingList   = [
    {
      "event":"brake",
      "step" : "강",
      "minValue" : strongMin.value,
      "maxValue" : strongMax.value,
      "opacity" : strongOpcity.value,
    },
    {
      "event":"brake",
      "step" : "중",
      "minValue" : mediumMin.value,
      "maxValue" : mediumMax.value,
      "opacity" : mediumOpcity.value,
    },
    {
      "event":"brake",
      "step" : "약",
      "minValue" : weakMin.value,
      "maxValue" : weakMax.value,
      "opacity" : weakOpcity.value,
    },
  ];

  function valueCheck() {
    console.log(settingList)
  }

  // console.log($attrs.settingValue);
</script>

<template>
<!-- <span @click="$emit('saveEvent')">저장</span> -->
  <section class="break-event-setting">
    <p class="setting-title">브레이크 이벤트 기준값 설정</p>
    <div class="setting-value-list">
      <div class="value-write-box"
        v-for="(list, index) in settingList"
        :key="index"
      >
        <p class="value-title">{{ list.step }}</p>
        <div class="input-box">
          <label for="min">최소값</label>
          <input type="text" id="min" v-model="list.minValue" placeholder="최소값">
        </div>
        <div class="input-box">
          <label for="max">최대값</label>
          <input type="text" id="max" v-model="list.maxValue" placeholder="최대값">
        </div>
        <div class="input-box">
          <label for="opacity">투명도</label>
          <input type="text" id="opacity" v-model="list.opacity" placeholder="투명도">
        </div>
      </div>
    </div>
    <div class="button-box">
      <button class="close-btn"  @click="$emit('closeEvent')">닫기</button>
      <button class="save-btn" @click.prevent="valueCheck">저장</button>
    </div>

    <div>{{ $attrs.settingValue[0].Event }}</div>
  </section>
</template>

<style lang="scss">
  .break-event-setting {
    .setting-title {
      margin: 20px 0;
      padding-left: 10px;
      border-left: 3px solid #2c2c2c;
    }
    .setting-value-list {
      margin-top: 5px;
      .value-write-box {
        display: flex;
        column-gap: 10px;
        align-items: center;
        margin-top: 5px;
        padding: 5px 0;
        border-bottom: 1px solid #ededed;
        &:last-child {
          border-bottom: 0;
        }
        .value-title {
          font-size: 14px;
          font-weight: bold;
        }
        .input-box {
          display: flex;
          flex-direction: column;
          & > label {
            font-size: 10px;
          }
          & > input {
            width: 100px;
            height: 40px;
            border:1px solid #d3d3d3;
            margin-top: 5px;
            border-radius: 5px;
            text-indent: 10px;
          }
        }
      }
    }
    .button-box {
      margin-top: 10px;
      text-align: center;
      & > button {
        display: inline-block;
        margin-left: 10px;
        width: 80px;
        height: 30px;
        background: #ededed;
        border-radius: 5px;
        border: 1px solid #c4c4c4;
        cursor: pointer;
        &.save-btn {
          background: #727272;
          color: #fff;
        }
      }
    }
  }
</style>

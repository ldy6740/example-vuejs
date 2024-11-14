<script setup>
  import { ref, defineProps } from "vue";
  import axios from 'axios';

  // let settingValues = useAttrs().settingValue
  const props = defineProps({
    settingValues: Array
  })



  let accelStrongMin     = ref("");
  let accelStrongMax     = ref("");
  let accelStrongOpcity  = ref("");

  let accelMediumMin     = ref("");
  let accelMediumMax     = ref("");
  let accelMediumOpcity  = ref("");

  let accelWeakMin       = ref("");
  let accelWeakMax       = ref("");
  let accelWeakOpcity    = ref("");

  let breakStrongMin     = ref("");
  let breakStrongMax     = ref("");
  let breakStrongOpcity  = ref("");

  let breakMediumMin     = ref("");
  let breakMediumMax     = ref("");
  let breakMediumOpcity  = ref("");

  let breakWeakMin       = ref("");
  let breakWeakMax       = ref("");
  let breakWeakOpcity    = ref("");


  // console.log(props.settingValues[0].Min_Value);

  accelStrongMin.value = props.settingValues[0].Min_Value
  accelStrongMax.value = props.settingValues[0].Max_Value
  accelStrongOpcity.value = props.settingValues[0].opacity

  accelMediumMin.value = props.settingValues[1].Min_Value
  accelMediumMax.value = props.settingValues[1].Max_Value
  accelMediumOpcity.value = props.settingValues[1].opacity

  accelWeakMin.value = props.settingValues[2].Min_Value
  accelWeakMax.value = props.settingValues[2].Max_Value
  accelWeakOpcity.value = props.settingValues[2].opacity

  breakStrongMin.value = props.settingValues[3].Min_Value
  breakStrongMax.value = props.settingValues[3].Max_Value
  breakStrongOpcity.value = props.settingValues[3].opacity

  breakMediumMin.value = props.settingValues[4].Min_Value
  breakMediumMax.value = props.settingValues[4].Max_Value
  breakMediumOpcity.value = props.settingValues[4].opacity

  breakWeakMin.value = props.settingValues[5].Min_Value
  breakWeakMax.value = props.settingValues[5].Max_Value
  breakWeakOpcity.value = props.settingValues[5].opacity


  let settingList   = ref([
    {
      "event":"accelerometer",
      "step" : "강",
      "min" : accelStrongMin.value,
      "max" : accelStrongMax.value,
      "opacity" : accelStrongOpcity.value,
    },
    {
      "event":"accelerometer",
      "step" : "중",
      "min" : accelMediumMin.value,
      "max" : accelMediumMax.value,
      "opacity" : accelMediumOpcity.value,
    },
    {
      "event":"accelerometer",
      "step" : "약",
      "min" : accelWeakMin.value,
      "max" : accelWeakMax.value,
      "opacity" : accelWeakOpcity.value,
    },
    {
      "event":"brake",
      "step" : "강",
      "min" : breakStrongMin.value,
      "max" : breakStrongMax.value,
      "opacity" : breakStrongOpcity.value,
    },
    {
      "event":"brake",
      "step" : "중",
      "min" : breakMediumMin.value,
      "max" : breakMediumMax.value,
      "opacity" : breakMediumOpcity.value,
    },
    {
      "event":"brake",
      "step" : "약",
      "min" : breakWeakMin.value,
      "max" : breakWeakMax.value,
      "opacity" : breakWeakOpcity.value,
    },
  ]);

  function valueCheck() {
  let testValue = { items: [] };
  let newValue = settingList.value.map((value) => {
    if (
      isNaN(value.max) === true ||
      isNaN(value.min) === true ||
      isNaN(value.opacity === true)
    ) {
      alert("숫자가 아닌 값이 들어 있습니다.");
      return;
    }

    return {
      event: value.event,
      step: value.step,
      min: value.min.toString(),
      max: value.max.toString(),
      opacity: value.opacity.toString(),
    };
  });

  testValue.items.push(...newValue);
  // console.log(testValue);

  saveValue(testValue);
}

  async function saveValue(value) {
  const API_URI = "http://localhost:3000/event/code";
  const options = {
    headers: {
      "content-type": "application/json",
    },
  };
  console.log(JSON.stringify(value));
  axios
    .post(API_URI, JSON.stringify(value), options)
    .then(function (response) {
      console.log("response : " + response);
    })
    .catch(function (error) {
      console.log("error : " + error);
    });
}
</script>

<template>
<!-- <span @click="$emit('saveEvent')">저장</span> -->
  <section class="break-event-setting">
    <p class="setting-title">감속 및 브레이크 이벤트 기준값 설정</p>
    <div class="setting-value-list">
      <div class="value-write-box"
        v-for="(list, index) in props.settingValues"
        :key="index"
      >
        <div>
          <p class="value-title">{{ list.Event === "accelerometer" ? "감속":"브레이크" }} <br> ({{ list.Step }})</p>
          <div class="input-box">
            <label for="min">최소값</label>
            <input type="text" id="min" placeholder="최소값" v-model="settingList[index].min">
          </div>
          <div class="input-box">
            <label for="max">최대값</label>
            <input type="text" id="max" placeholder="최대값" v-model="settingList[index].max">
          </div>
          <div class="input-box">
            <label for="opacity">투명도</label>
            <input type="text" id="opacity" placeholder="투명도" v-model="settingList[index].opacity">
          </div>
        </div>
      </div>
    </div>
    <div class="button-box">
      <button class="close-btn"  @click="$emit('closeEvent')">닫기</button>
      <button class="save-btn" @click.prevent="valueCheck">저장</button>
    </div>
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
        & > div {
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
            width: 80px;
          }
          .input-box {
            display: flex;
            flex-direction: column;
            & > label {
              font-size: 10px;
            }
            & > input {
              width: 80px;
              height: 40px;
              border:1px solid #d3d3d3;
              margin-top: 5px;
              border-radius: 5px;
              text-indent: 10px;
            }
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

<script setup>
  import { ref, useAttrs } from "vue";
  import axios from 'axios';

  let settingValues = useAttrs().settingValue

  let accelStrongMin     = ref(settingValues[0].Min_Value);
  let accelStrongMax     = ref(settingValues[0].Max_Value);
  let accelStrongOpcity  = ref(settingValues[0].opacity);

  let accelMediumMin     = ref(settingValues[2].Min_Value);
  let accelMediumMax     = ref(settingValues[2].Min_Value);
  let accelMediumOpcity  = ref(settingValues[2].Min_Value);

  let accelWeakMin       = ref(settingValues[1].Min_Value);
  let accelWeakMax       = ref(settingValues[1].Min_Value);
  let accelWeakOpcity    = ref(settingValues[1].Min_Value);

  let breakStrongMin     = ref(settingValues[3].Min_Value);
  let breakStrongMax     = ref(settingValues[3].Min_Value);
  let breakStrongOpcity  = ref(settingValues[3].Min_Value);

  let breakMediumMin     = ref(settingValues[5].Min_Value);
  let breakMediumMax     = ref(settingValues[5].Min_Value);
  let breakMediumOpcity  = ref(settingValues[5].Min_Value);

  let breakWeakMin       = ref(settingValues[4].Min_Value);
  let breakWeakMax       = ref(settingValues[4].Min_Value);
  let breakWeakOpcity    = ref(settingValues[4].Min_Value);


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
    let newValue = settingList.value.map((value) => {
      if(isNaN(value.max) === true || isNaN(value.min) === true || isNaN(value.opacity === true)) {
        alert('숫자가 아닌 값이 들어 있습니다.');
        return;
      }

      return ({
        "item":
        [
          {
            "event": value.event,
            "step" : value.opacity,
            // "min" : parseFloat(value.min),
            // "max" : parseFloat(value.max),
            // "opacity" : parseFloat(value.opacity),
            "min" : value.min,
            "max" : value.max,
            "opacity" : value.opacity,
          }
        ]
      })
    });

    // saveValue(newValue);
    console.log(newValue);

  }

  // async function saveValue(values) {
  //   const API_URI = "http://localhost:3000/event/code";
  //   console.log(values);

  //   const responses = await axios.post(API_URI, {

  //     },
  //     {
  //       headers: {
  //         'Content-Type': 'multipart/form-data'
  //       }
  //     }
  //   ).then((response)=>{
  //     return response
  //   })

  //   console.log(responses);

  // }
</script>

<template>
<!-- <span @click="$emit('saveEvent')">저장</span> -->
  <section class="break-event-setting">
    <p class="setting-title">감속 및 브레이크 이벤트 기준값 설정</p>
    <div class="setting-value-list">
      <div class="value-write-box"
        v-for="(list, index) in $attrs.settingValue"
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

    <!-- <div
      v-for="(value, index) in $attrs.settingValue"
      :key="index"
    >
      <div  v-if="value.Event === 'accelerometer'">
        <input type="text" :value="value.Event">
      </div>
    </div> -->
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

<script setup>
  import { ref, defineProps } from "vue";
  import axios from 'axios';

  // let settingValues = useAttrs().settingValue
  const props = defineProps({
    settingValues: Array
  })


  // 설정 값 ref 모음
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


  // PeriodSearch.vue 에서 DB에 저장된 감속정도 기준값을 조회하여
  // 자식 컴포넌트 SettingView.vue -> AccelSettingView.vue 로 데이터를 전달하여
  // 설정 값 ref 변수에 담는다.
  accelStrongMin.value    = props.settingValues[0].Min_Value
  accelStrongMax.value    = props.settingValues[0].Max_Value
  accelStrongOpcity.value = props.settingValues[0].opacity

  accelMediumMin.value    = props.settingValues[1].Min_Value
  accelMediumMax.value    = props.settingValues[1].Max_Value
  accelMediumOpcity.value = props.settingValues[1].opacity

  accelWeakMin.value      = props.settingValues[2].Min_Value
  accelWeakMax.value      = props.settingValues[2].Max_Value
  accelWeakOpcity.value   = props.settingValues[2].opacity

  breakStrongMin.value    = props.settingValues[3].Min_Value
  breakStrongMax.value    = props.settingValues[3].Max_Value
  breakStrongOpcity.value = props.settingValues[3].opacity

  breakMediumMin.value    = props.settingValues[4].Min_Value
  breakMediumMax.value    = props.settingValues[4].Max_Value
  breakMediumOpcity.value = props.settingValues[4].opacity

  breakWeakMin.value      = props.settingValues[5].Min_Value
  breakWeakMax.value      = props.settingValues[5].Max_Value
  breakWeakOpcity.value   = props.settingValues[5].opacity


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

/**
 * 사용자가 입력한 셋팅값이 숫자인지 체크하고 JSON형태로 변황
 */
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
let newDateValue = new Date("2024-11-05 12:00:00");
newDateValue.setSeconds(newDateValue.getSeconds() - 30);

console.log(newDateValue);

/**
 * 사용자가 입력한 설정 값을 받아 API 통신을 이용하여 데이터 전달하여 DB에 저장
 * @param {Arrar} values 사용자 페이지에서 입력한 설정 값
 */
async function saveValue(values) {
  const API_URI = "http://221.164.108.130:8088/event/code";
  //NOTE const API_URI = "http://221.164.108.130:8088/event/code"; 서버 컴퓨터로 옮겼을 경우 해당 주소로 변경

  const options = {
    headers: {
      "content-type": "application/json",
    },
  };
  console.log(JSON.stringify(values));
  axios
    .post(
      API_URI,
      JSON.stringify(values),
      options
    )
    .then(function (response) {
      console.log("response : " + response);
    })
    .catch(function (error) {
      console.log("error : " + error);
    });
}


// NOTE validation rule 체크하면은 입력 값을 넘겨주고, 넘겨준 값이 형식에 맞지 않으면 포커스를 올겨주고 어떤 rule에 맞지 않는지 사용자에서 알려줘야 한다.
function numverCheck(value) {
  let numValue = Number(value); // String 으로 전달되는 입력값을 Number type 으로 변환

  if (typeof(numValue) === 'number' && isNaN(value) === false) {
    return true;
  };

  return false;
}

function validationRuleCheck(item) {

  // 최대값은 중복되지 않아야 한다.
  if (item.target.id === 'max') {
    let isMaxValue = props.settingValues.find((value) => {
			return value.Max_Value === Number(item.target.value);
		})
		console.log(isMaxValue);
		if (isMaxValue) {
			alert(`${isMaxValue.Event} 의 ${isMaxValue.Step} 의 값과 일치 합니다. Max 값은 중복되는 값을 설정할 수 없습니다.`)
      item.target.focus();
		}
  }

  // opacity 는 1 - 0 사이값이 입력 되어야 한다.
  if(item.target.id === "opacity") {
    let integerValue = item.target.value * 10;

    const regex = /^(?!.*(^\.?\d+\.$|^\.\d$)).*$/;
    if(!regex.test(item.target.value)) {
      alert('입력하신 값은 사용할수 없는 값입니다.')
      item.target.focus();
      return;
    }
    console.log(integerValue);
    if(integerValue > 10) {
      alert('입력하신 값이 1보다 큽니다.');
      item.target.focus();
    } else if (integerValue < 0) {
      alert('입력하신 값이 0보다 작습니다.')
      item.target.focus();
    }
  }

  // 입력값이 있는지 체크(빈값을 입력 하지 못하도록 한다.)
	if (item.target.value) {
		let isNumber = numverCheck(item.target.value); // 입력한 값이 숫자를 입력했는지 체크한다.
		if(isNumber) {
			console.log('숫자 입니다. ');
		} else {
			alert("숫자만 입력 가능합니다.");
			item.target.focus();
		}
	} else {
		alert("빈값을 입력할 수 없습니다. 값을 입력해주세요");
		item.target.focus();	}
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
            <input type="text" id="min" @change="validationRuleCheck($event)" placeholder="최소값" v-model="settingList[index].min" maxlength="3">
          </div>
          <div class="input-box">
            <label for="max">최대값</label>
            <input type="text" id="max" @change="validationRuleCheck($event)" placeholder="최대값" v-model="settingList[index].max" maxlength="3">
          </div>
          <div class="input-box">
            <label for="opacity">투명도</label>
            <input type="text" id="opacity" @change="validationRuleCheck($event)" placeholder="투명도" v-model="settingList[index].opacity" maxlength="3">
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

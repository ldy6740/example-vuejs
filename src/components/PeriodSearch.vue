<script setup>
//component import
import MapView from './MapView.vue';

// imports
import {ref} from 'vue';
import axios from 'axios';

// flatPickr imports
import flatPickr from 'vue-flatpickr-component';
import 'flatpickr/dist/flatpickr.css';
import ConfirmDatePlugin from 'flatpickr/dist/plugins/confirmDate/confirmDate.js';
import { Korean } from 'flatpickr/dist/l10n/ko.js';

// flatPickr config
const config        = ref({
    wrap: true, // set wrap to true only when using 'input-group'
    dateFormat: "Y-m-d H:i:S",
    locale: Korean, // locale for this instance only
    enableTime: true,
    enableSeconds: true,
    time_24hr: true,
    plugins: [new ConfirmDatePlugin({})]
});


//검색 조건 저장 변수
const FIRST_START_TIME   = ref('');    // 조회 버튼 누른 시간
const vehicleNumber      = ref('');    // 차량번호
const startDate          = ref('');    // 시작일 및 시작시간
const endData            = ref('');    // 종료일 및 종료시간

// ref 변수 모음
const isFunction         = ref(true);  // 함수 실행 여부
let responseData         = ref([]);    // 조회결과 데이터

/**
 * 조회 기간을 설정하여 차량 운행 데이터 조회
 * @param vehicleNumber   차량 번호(String)
 * @param startTime       시작일 및 시작시간(String) "2024-10-25 11:20:20"
 * @param endTime         종료일 및 종료시간(String) "2024-10-25 11:20:20"
 */
async function getCoordinates(vehicleNumber, startTime, endTime) {
  FIRST_START_TIME.value = startTime // 조회 버튼 누른 시간을 시작일로 셋팅
	const URI = "http://localhost:3000/event/accelerometer"; // API 주소

  //데이터 리스트 초기화
  if (responseData.value.length) {
    responseData.value  = []; // 조회결과 데이터 리스트 초기화
  }

  // 검색 조건 params
	const params = {
		"number"          :`${vehicleNumber}`,
    "firststarttime"  : FIRST_START_TIME,   // 조회 버튼을 누른 시간이며, 실시간 조회에서 사용(조회 페이지에서는 시작일로설정)
		"starttime"       : startTime,          // 시작일 및 시작시간
		"endtime"         : endTime,            // 종료일 및 종료시간
	}

	const queryString = new URLSearchParams(params).toString();
	const requrl = `${URI}?${queryString}`;

  try {
		// axios를 이용하여 api 호출
		const responses = await axios.get(requrl, {
			headers: {
				"Content-Type": "application/json",
			}
		}).then((response) => {
      return response;
    })

    if(!responses.data){
      alert("존재하는 데이터가 없습니다.");
    } else {
      responseData.value.push(...responses.data);
    };

	} catch (error) {
    // console.log(response);
		console.log(error);
		alert("Network Error 서버연결을 확인해주세요.");
	};
};


</script>

<template>
 	<section class="search-area">
		<!-- 조회 화면 -->
		<form id="form">
			<div class="input-box">
				<input type="text" v-model="vehicleNumber" id="vehicle-number" placeholder="차량번호를 입력하세요">
			</div>
      <div class="input-box">
        <flat-pickr
          v-model="startDate"
          :config="config"
          class="form-control"
          placeholder="시작일 및 시간을 선택해주세요"
          name="date"/>
      </div>
      <div class="input-box">
        <flat-pickr
          v-model="endData"
          :config="config"
          class="form-control"
          placeholder="종료일 및 시간을 선택해주세요"
          name="date"/>
      </div>

			<div class="button-box">
				<button @click.prevent="getCoordinates(vehicleNumber, startDate, endData)" type="submit" id="submit">조회</button>
			</div>
		</form>
  </section>
  <MapView
    :isFunction="isFunction"
    :responseData="responseData" />
</template>

<style scoped>

</style>

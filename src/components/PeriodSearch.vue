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

let pointList       = ref([]); // 조회된 좌표 값 리스트
const isFunction    = ref(true); // 함수 실행 여부
let brakePoint      = ref([]); // 브레이크 이벤트 좌표
let brakeData       = ref([]); // 브레이크 이벤트 데이터
let searchAllData   = ref([]);

const vehicleNumber = ref('');
const startDate     = ref(''); // start date
const endData       = ref(''); // end date
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

// 현재 시간 조회 함수
// function getCurrentTime() {

//   let today = new Date();

//   let year = today.getFullYear() //년
//   let month = today.getMonth() < 10 ? `0${today.getMonth() + 1}`: today.getMonth() + 1; // 월
//   let date = today.getDate() < 10 ? `0${today.getDate()}`: today.getDate(); // 일

//   let hours = today.getHours() < 10 ? `0${today.getHours()}`: today.getHours(); // 시
//   let minutes = today.getMinutes() < 10 ? `0${today.getMinutes()}`: today.getMinutes();  // 분
//   let seconds = today.getSeconds() < 10 ? `0${today.getSeconds()}`: today.getSeconds();  // 초


//   return `${year}-${month}-${date} ${hours}:${minutes}:${seconds}`;
// }

// 차량 accelerometer 운행 데이터 조회 함수
async function getCoordinates(vehicleNumber, startTime, endTime) {
  //
  if (pointList.value.length){
    pointList.value = [];
    brakePoint.value = [];
    brakeData.value = [];
    searchAllData.value = [];
  }
	const uri = "http://localhost:3000/event/accelerometer";
	// const uri = API_URL;
	const params = {
		"number":`${vehicleNumber}`,
		// "starttime": "2024-10-25 15:06:06.000",
		// "endtime": "2024-10-25 15:45:06.000",
    // "firststarttime": getCurrentTime(), //조회 버튼 누른 시간
    "firststarttime": startTime,//조회 버튼 누른 시간
		"starttime": startTime,
		"endtime": endTime,
	}
	const queryString = new URLSearchParams(params).toString();
	const requrl = `${uri}?${queryString}`;
	try {
		// axios API를 이용하여 api 호출
		const response = await axios.get(requrl, {
			// method: 'GET',
			headers: {
				"Content-Type": "application/json",
			}
		}).then((responses) => {
      return responses;
    })

		let data = response.data;
		// console.log("data :" + response.data);

    searchAllData.value.push(...data);
    // 브레이크 포인트 데이터 가공을 위해 전체 데이터 전달
    getBreakData(data)

		//운행 데이터 조회하여 좌표 값 데이터 가공
		var coordinatesValue = data.map((items) => {
			// data에서 좌표값만 답기
			let lat = items.GPS_Latitude;
			let lon = items.GPS_Longitude;

			//자표값을 배열 형태로 반환
			return new window.kakao.maps.LatLng(lat, lon);
		});

		pointList.value.push(...coordinatesValue);


	} catch (error) {
		console.log(error);
		alert("존재하는 데이터가 없습니다.");
	};
  // console.log("pointList :" + pointList.value);
};


function getBreakData(datas) {
  // console.log(`datas: ${datas}`)

  const items = datas.filter((data) => {
    return data.gubun === 'brake';
  })

  brakeData.value.push(...items);

  const values = items.map((item, index) => {

    const value = {
      id: index,
      title: "brake",
      latlng: new window.kakao.maps.LatLng(item.GPS_Latitude, item.GPS_Longitude)
    }
    return value;
  });

  // console.log(...values);
  // brakePoint.value.push(values);
  brakePoint.value.push(...values);

};


</script>

<template>
 	<section class="search-area">
		<!-- 조회 화면 -->
		<form id="form">
			<div class="input-box">
				<input type="text" v-model="vehicleNumber" id="vehicle-number" placeholder="차량번호를 입력하세요">
			</div>
			<!-- <div class="input-box">
				<input type="text" id="vehicle-date" placeholder="날짜">
			</div> -->
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
  <MapView :pointList="pointList" :isFunction="isFunction" :breakPoint="brakePoint" :breakData="brakeData" :searchAllData />
  <!-- <MapView :data="{pointList: pointList, isFunction:isFunction, brakePoint: brakePoint}" /> -->
</template>

<style scoped>

</style>

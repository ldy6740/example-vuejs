<script setup>
import { ref } from 'vue';
import MapView from './MapView.vue';
import axios from 'axios';


let pointList = ref([]);
const isFunction = ref(false);

let FS_START_TIME = "";
let START_TIME = "";
let END_TIME = "";

// 현재 시간 조회 함수
function getCurrentTime() {

	let today = new Date();

	let year = today.getFullYear() //년
	let month = today.getMonth() < 10 ? `0${today.getMonth() + 1}`: today.getMonth() + 1; // 월
	let date = today.getDate() < 10 ? `0${today.getDate()}`: today.getDate(); // 일

	let hours = today.getHours() < 10 ? `0${today.getHours()}`: today.getHours(); // 시
	let minutes = today.getMinutes() < 10 ? `0${today.getMinutes()}`: today.getMinutes();  // 분
	let seconds = today.getSeconds() < 10 ? `0${today.getSeconds()}`: today.getSeconds();  // 초


	return `${year}-${month}-${date} ${hours}:${minutes}:${seconds}`;
}

// 조회 버튼 첫번째 클릭
function firstSearch() {
  // 차량 번호
  let vehicleNumber = document.querySelector('#vehicle-number').value;

  // 최초 시작 시간 설정
  FS_START_TIME = getCurrentTime();
	START_TIME = getCurrentTime();
  END_TIME = START_TIME;

  getCoordinates(vehicleNumber, START_TIME, END_TIME);
  realTimeCheck(vehicleNumber);
}


function realTimeCheck(vehicleNumber) {

  // 실시간인 경우 10초에 한번씩 자동 조회
	setInterval(() => {
		START_TIME = END_TIME;
		END_TIME = getCurrentTime();

		getCoordinates(vehicleNumber, START_TIME, END_TIME);
	}, 10000);


}

// 차량 accelerometer 운행 데이터 조회 함수
async function getCoordinates(vehicleNumber, startTime, endTime) {
	const uri = "http://localhost:3000/event/accelerometer";
	// const uri = API_URL;
	const params = {
		"number":`${vehicleNumber}`,
		// "starttime": "2024-11-05 12:00:00",
		// "endtime": "2024-11-05 14:00:00",
    "firststarttime": FS_START_TIME,
		"starttime": startTime,
		"endtime": endTime,
	}
	const queryString = new URLSearchParams(params).toString();
	const requrl = `${uri}?${queryString}`;
	try {
		// fetch API를 이용하여 api 호출
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

		//운행 데이터 조회하여 좌표 값 데이터 가공
		var coordinatesValue = data.map((items) => {
			// data에서 좌표값만 답기
			let lat = items.GPS_Latitude;
			let lon = items.GPS_Longitude

			//자표값을 배열 형태로 반환
			return new window.kakao.maps.LatLng(lat, lon);
		});

		pointList.value.push(...coordinatesValue);


	} catch (error) {
		console.log(error);
		alert("존재하는 데이터가 없습니다.")
	};
  // console.log("pointList :" + pointList.value);
};


</script>


<template>
 	<section class="search-area">
		<!-- 조회 화면 -->
		<form id="form">
			<div class="input-box">
				<input v-model="vehicleNumber" type="text" id="vehicle-number" placeholder="차량번호를 입력하세요">
			</div>
			<div class="button-box">
				<button @click.prevent="firstSearch" type="submit" id="submit">조회</button>
			</div>
		</form>
	</section>
  <MapView :pointList="pointList" :isFunction="isFunction" />

</template>

<style scoped>

</style>

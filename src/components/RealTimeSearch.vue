<script setup>
import MapView from './MapView.vue';

let pointList = [];
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
	START_TIME = getCurrentTime();
  END_TIME = START_TIME;


  realTimeCheck(vehicleNumber);
}

function realTimeCheck(vehicleNumber) {
  getCoordinates(vehicleNumber, START_TIME, END_TIME);
}

// 차량 accelerometer 운행 데이터 조회 함수
async function getCoordinates(vehicleNumber, startTime, endTime) {
	const uri = "http://localhost:3000/event/accelerometer";
	const params = {
		"number":`${vehicleNumber}`,
		"starttime": "2024-10-25 15:06:06.000",
		"endtime": "2024-10-25 15:06:06.000",
		// "starttime": startTime,
		// "endtime": endTime,
	}
	const queryString = new URLSearchParams(params).toString();
	const requrl = `${uri}?${queryString}`;
	try {
		// fetch API를 이용하여 api 호출
		const response = await fetch(requrl, {
			method: 'GET',
			headers: {
				"Content-Type": "application/json",
			}
		});

		const data = await response.json();
		console.log(data);

		//운행 데이터 조회하여 좌표 값 데이터 가공
		var coordinatesValue = data.map((items) => {
			// data에서 좌표값만 답기
			let lat = items.GPS_Latitude;
			let lon = items.GPS_Longitude

			//자표값을 배열 형태로 반환
			return new window.kakao.maps.LatLng(lat, lon);
		});

		pointList.push(...coordinatesValue);


	} catch (error) {
		console.log(error);
		// alert("존재하는 데이터가 없습니다.")
	};
}

console.log(pointList);

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
  <MapView />

</template>

<style scoped>

</style>

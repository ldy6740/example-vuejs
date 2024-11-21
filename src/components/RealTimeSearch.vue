<script setup>
import { ref, onUnmounted } from 'vue';
import MapView from './MapView.vue';
import axios from 'axios';

let autoSearch   = null;
let responseData = ref([]);
const isFunction = ref(false);

let FS_START_TIME = "";
let START_TIME = "";
let END_TIME = "";

// 시간 포맷 변경 함수 들어온 시간 데이터를 "yyyy-MM-dd hh:mm:ss" 형태로 변형한다.
function getFormatDate(dateValue) {
	let today = dateValue;

	let year = today.getFullYear() //년
	let month = today.getMonth() < 10 ? `0${today.getMonth() + 1}`: today.getMonth() + 1; // 월
	let date = today.getDate() < 10 ? `0${today.getDate()}`: today.getDate(); // 일

	let hours = today.getHours() < 10 ? `0${today.getHours()}`: today.getHours(); // 시
	let minutes = today.getMinutes() < 10 ? `0${today.getMinutes()}`: today.getMinutes();  // 분
	let seconds = today.getSeconds() < 10 ? `0${today.getSeconds()}`: today.getSeconds();  // 초


	return `${year}-${month}-${date} ${hours}:${minutes}:${seconds}`;
}

// 조회 버튼 첫번째 클릭
// function firstSearch() {
//   // 차량 번호
//   let vehicleNumber = document.querySelector('#vehicle-number').value;

//   let firstStartTime = new Date();
//   firstStartTime.setSeconds(firstStartTime.getSeconds() - 30)
//   // 최초 시작 시간 설정
//   FS_START_TIME = getFormatDate(firstStartTime);
// 	START_TIME = getFormatDate(firstStartTime);
//   END_TIME = getFormatDate(new Date());

//   getCoordinates(vehicleNumber, START_TIME, END_TIME);
//   realTimeCheck(vehicleNumber);
// }

// 테스트를 위한 코드 구성 이전 시간 기준 데이터 조회 테스트
function firstSearch() {
  // 차량 번호
  let vehicleNumber = document.querySelector('#vehicle-number').value;

  let firstStartTime = new Date("2024-11-05 13:20:00");
  firstStartTime.setSeconds(firstStartTime.getSeconds() - 30)
  // 최초 시작 시간 설정
  FS_START_TIME = getFormatDate(firstStartTime);
	START_TIME = getFormatDate(firstStartTime);
  END_TIME = getFormatDate(new Date("2024-11-05 13:20:00"));

  getCoordinates(vehicleNumber, START_TIME, END_TIME);
  realTimeCheck(vehicleNumber);
}

// function realTimeCheck(vehicleNumber) {
//   // 실시간인 경우 10초에 한번씩 자동 조회
// 	setInterval(() => {
// 		START_TIME = END_TIME;
// 		END_TIME = getFormatDate(new Date());

// 		getCoordinates(vehicleNumber, START_TIME, END_TIME);
// 	}, 10000);

// }

function realTimeCheck(vehicleNumber) {
  // 실시간인 경우 10초에 한번씩 자동 조회
	autoSearch = setInterval(() => {
    let testDate = new Date(END_TIME);
    testDate.setSeconds(testDate.getSeconds() + 10);
		START_TIME = END_TIME;
		END_TIME = getFormatDate(testDate);

		getCoordinates(vehicleNumber, START_TIME, END_TIME);
	}, 10000);

}
// 페이지 이탈시 실시간 조회 종료
onUnmounted(() => {
  clearInterval(autoSearch);
});

/**
 * 조회 기간을 설정하여 차량 운행 데이터 조회
 * @param vehicleNumber   차량 번호(String)
 * @param startTime       시작일 및 시작시간(String) "2024-10-25 11:20:20"
 * @param endTime         종료일 및 종료시간(String) "2024-10-25 11:20:20"
 */
async function getCoordinates(vehicleNumber, startTime, endTime) {
	const URI = "http://221.164.108.130:8088/event/accelerometer"; // API 주소
  //NOTE const URI = "http://221.164.108.130:8088/event/accelerometer"; 서버 컴퓨터로 옮겼을 경우 해당 주소로 변경

  // //데이터 리스트 초기화
  // if (responseData.value.length) {
  //   responseData.value  = []; // 조회결과 데이터 리스트 초기화
  // }

  // 검색 조건 params
	const params = {
		"number"          :`${vehicleNumber}`,
    "firststarttime"  : FS_START_TIME,           // 조회 버튼을 누른 시간이며, 실시간 조회에서 사용(조회 페이지에서는 시작일로설정)
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
      console.log('존재하는 데이터가 없습니다');
      // alert("존재하는 데이터가 없습니다.");
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
				<input v-model="vehicleNumber" type="text" id="vehicle-number" placeholder="차량번호를 입력하세요">
			</div>
			<div class="button-box">
				<button @click.prevent="firstSearch" type="submit" id="submit">조회</button>
			</div>
		</form>
	</section>
  <MapView :responseData="responseData" :isFunction="isFunction" />

</template>

<style scoped>

</style>

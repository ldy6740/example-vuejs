<script setup>
import { ref, onMounted, watch, defineProps} from 'vue';
const { VITE_KAKAO_MAP_KEY } = import.meta.env;

const mapCountainer          = ref(null);

// 대쉬보드 데이터 변수
let breakStrong              = ref(0);         // 브레이크 이벤트(강)
let breakMedium              = ref(0);         // 브레이크 이벤트(중)
let breakWeak                = ref(0);         // 브레이크 이벤트(약)
let breakSum                 = ref(0);         // 브레이크 이벤트(계)
let breakDeceleration        = ref(0);         // 누적 감속 값(브레이크)
let accelStrong              = ref(0);         // 감속 이벤트(강)
let accelMedium              = ref(0);         // 감속 이벤트(중)
let accelWeak                = ref(0);         // 감속 이벤트(약)
let accelSum                 = ref(0);         // 감속 이벤트(계)
let accelDeceleration        = ref(0);         // 누적 감속 값(가속도계)
let totalMileage             = ref(0);         // 총 주행거리 값

let showOverlay              = ref(false);     // custom overlay 화면 표시 여부 true or false

let map                      = null;           // kakao map Object 변수

// 이벤트 정보 데이터 변수
let clickMarkerVehicleNumber = ref("-");       // 마커를 클릭하면 해당 위치의 차량번호 값
let clickMarkerGubun         = ref("-");       // 마커를 클릭하면 해당 위치의 gubun 값 [브레이크 or 감속]
let clickMarkerCalcValue     = ref("-");       // 마커를 클릭하면 해당 위치의 감속정도 값

let colorValue               = ref("rgb(255, 0, 0)");


// props items
const props = defineProps({
  isFunction: Boolean,
  responseData: Array
});

onMounted(() => {
  loadKakaoMap(mapCountainer.value);
  // colorClick();
})

/**
 * 지도 화면 대쉬보드 데이터 추출
 */
function dashboardData() {
  // "brake" 데이터 추출
  const breakList = props.responseData.filter((data) => data.gubun === 'brake');
  if (breakList.length) {
    breakStrong.value = breakList[0].Calc_Value_sum1;
    breakMedium.value = breakList[0].Calc_Value_sum2;
    breakWeak.value = breakList[0].Calc_Value_sum3;
    breakSum.value = Number(breakStrong.value) + Number(breakMedium.value) + Number(breakWeak.value);
    breakDeceleration.value = breakList[0].calc_value_sum.toFixed(2);
  }

  // "accel" 데이터 추출
  const accelList = props.responseData.filter((data) => data.gubun === 'accel');
  if (accelList.length) {
    accelStrong.value = accelList[0].Calc_Value_sum1;
    accelMedium.value = accelList[0].Calc_Value_sum2;
    accelWeak.value = accelList[0].Calc_Value_sum3;
    accelSum.value = Number(accelStrong.value) + Number(accelMedium.value) + Number(accelWeak.value);
    accelDeceleration.value = accelList[0].calc_value_sum.toFixed(2);
    totalMileage.value = accelList[0].Distance_Traveled;
  }
}

function colorClick() {


  let colorItem = document.querySelectorAll('.color-bg-pick');
  colorItem.forEach((item) => {
    item.addEventListener('click', (e) => {
      colorItem.forEach((remove) => {
        remove.classList.remove('on');
      })
      e.target.classList.add('on');
      colorValue.value = getComputedStyle(e.target).backgroundColor;
    })
  });

  map.polyline.setOptions({
    strokeColor : colorValue.value,
  });
}


// 경로 투명도 조절
let opacityValue = ref(1);
let number = 10;
let value = '';

function valueUp() {
  if (opacityValue.value == 1) {
    return;
  }
  if (number < 10) {
    number += 1;
    value = `0.${number}`;
  }
  // console.log(value === '0.10'? 10 : value);
  opacityValue.value = Number(value === '0.10'? 1 : value);
  // map.polyline.setOpacity(opacityValue.value);

  optionValueChange(opacityValue.value)
};
function valueDow() {
  if(opacityValue.value == 0) {
    return;
  }

  if (number <= 10) {
    number -= 1;
    value = `0.${number}`;
  }

  opacityValue.value = Number(value === '0.0'? 0 : value);
  optionValueChange(opacityValue.value)
};

function optionValueChange(value) {
  map.polyline.setOptions({
    strokeOpacity : value,
  })
}

// 운행경로를 지도에 표시하는 함수
const drawPolyline = (pathCoordinates) => {
  // 이전 경로를 지우기 위해 기존 폴리라인 삭제
  if(props.isFunction) {
    if (map && map.polyline) {
      map.polyline.setMap(null);
    }
  }
  //	지도에 선을 표시한다
  map.polyline = new window.kakao.maps.Polyline({
		map									:	map, 										// 선을 표시할 지도 객체
		path								:	pathCoordinates,				// 경로 라인 좌표 리스트
		strokeWeight				:	5,											// 선의 두께
		strokeColor					:	"rgb(255, 0, 0)",							// 선 색
		strokeOpacity				:	opacityValue.value,			// 선 투명도
		strokeStyle					:	"solid"									// 선 스타일
	});

};

const loadKakaoMap = (container) => {
  const script = document.createElement('script');
  script.src = `https://dapi.kakao.com/v2/maps/sdk.js?appkey=${VITE_KAKAO_MAP_KEY}&autoload=false`
  document.head.appendChild(script);

  script.onload = () => {
    window.kakao.maps.load(() => {
      const options = {
        center: new window.kakao.maps.LatLng(35.481937, 129.355698),
        level: 3,
      }

      map =  new window.kakao.maps.Map(container, options);

    });
  };
};

 /**
  * Response Data를 Polyline 좌표값으로 가공
  * @param {*} data
  */
function getPolylineCoords(data) {
  let coordinatesValue = data.map((items) => {
    // response data 에서 경로라인에 사용할 좌표 추출
    let lat = items.GPS_Latitude;
    let lon = items.GPS_Longitude;

    // 좌표 값을 kakao map api polyline 좌표값으로 변환하여 반환
    return new window.kakao.maps.LatLng(lat, lon);
  });
  drawPolyline(coordinatesValue) // 지도에 좌표 표시 함수로 데이터 전달
}


/**
 * Response Data를 Event 데이터로 가공
 * @param {*} data
 */
function getEventData(data) {
  const eventDatas = data.map((item) => {
    const eventData = {
      latlng: new window.kakao.maps.LatLng(item.GPS_Latitude, item.GPS_Longitude),
      gubun: item.gubun,
      calc_value: item.calc_value
    }
    return eventData;
  });

  eventPointMarker(eventDatas)
}

//props.responseData 데이터 변화 감지
watch(
  () => props.responseData,
  (newResponseData) => {
    if (map) {
      getPolylineCoords(newResponseData);
      getEventData(newResponseData)
      dashboardData();
      colorClick();
    }
  },
  {deep: true}
)



let markers = [];
let oldMarker = null;
let markerImages = null;
let testMakerImg = null;
/**
 * Event 발생 지점 Marker 표시 및 마커 클릭시 이벤트 정보 표시
 * @param {*} eventDatas
 */
function eventPointMarker(eventDatas) {

  markers.forEach(marker => marker.setMap(null));
  markers = [];

  let Marker = null;

  //마커 이미지 주소
  const imageSrc =  "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png";
	const testImgSrc = 'https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/marker_red.png';

  // 지도에 마커 생성
  eventDatas.forEach((eventData, index) => {
    //	마커 이미지의 이미지 크기 입니다.
	  var imageSize	=	new window.kakao.maps.Size(24, 35);
    //	마커 이미지를 생성합니다.
	  markerImages	=	new window.kakao.maps.MarkerImage(imageSrc, imageSize);
    testMakerImg = new window.kakao.maps.MarkerImage(testImgSrc, imageSize);

    Marker = new window.kakao.maps.Marker({
      map				:	map,											        // 마커를 표시할 지도
      position	:	eventData.latlng,				          // 마커를 표시할 위치
      image			:	eventData.gubun === 'accel'
                  ? testMakerImg : markerImages,    // 이벤트별 마커 이미지 구분
      clickble  : true,                             // 마커 클릭이벤트 감지 옵션
    });

    // 마커 클릭 이벤트 감지
    window.kakao.maps.event.addListener(Marker, 'click', function() {
      clickMarkerVehicleNumber.value  = props.responseData[index].Vehicle_Number                          // 클릭한 마커의 차량번호 정보 추출
      clickMarkerGubun.value          = props.responseData[index].gubun === 'brake' ? '브레이크' : '감속';  // 클릭한 마커의 이벤트 명 추출
      clickMarkerCalcValue.value      = props.responseData[index].calc_value                              // 클릭한 마커의 감속정도 추출

      var imageSize	=	new window.kakao.maps.Size(34, 45); // 이미지 사이즈 지정
      var markerImage	=	new window.kakao.maps.MarkerImage(imageSrc, imageSize); // 마커 이미지 생성

      if (oldMarker) {
        oldMarker.setImage(markerImages); // oldMarker에 등록된 값이 있으면
      }
      this.setImage(markerImage);
      oldMarker = this;
      showOverlay.value = true;
    });

    markers.push(Marker);
  });
}


function hideOverlay() {
  // showOverlay.value = false;
  // 클릭마커 정보 초기화
  clickMarkerVehicleNumber.value = "-";
  clickMarkerGubun.value = "-";
  clickMarkerCalcValue.value = "-";

  oldMarker.setImage(markerImages);
}


// console.log(attrs);

</script>

<template>
  <section class="map-area" ref="mapCountainer">
		<article class="map-view" id="map">
      <article  class="custom-overlay">
        <span @click="hideOverlay" class="overlay-reset-btn">초기화</span>
        <div class="overlay-view">
          <p class="overlay-view-label">이벤트 정보</p>
          <div class="overlay-item">
            <p class="overlay-label">차량번호</p>
            <p class="overlay-value">{{ clickMarkerVehicleNumber }}</p>
          </div>
          <div class="overlay-item">
            <p class="overlay-label">이벤트명</p>
            <p class="overlay-value">{{ clickMarkerGubun }}</p>
          </div>
          <div class="overlay-item">
            <p class="overlay-label">감속정도</p>
            <p class="overlay-value">{{ clickMarkerCalcValue }}</p>
          </div>
        </div>
      </article>
			<div class="dashboard-area">
        <div class="break-area">
          <p class="break-area-label">브레이크 이벤트</p>
          <div class="dashboard-item">
            <p class="item-label">강</p>
            <p class="item-value"> {{ breakStrong }}회</p>
          </div>
          <div class="dashboard-item">
            <p class="item-label">중</p>
            <p class="item-value"> {{ breakMedium }}회</p>
          </div>
          <div class="dashboard-item">
            <p class="item-label">약</p>
            <p class="item-value"> {{ breakWeak }}회</p>
          </div>
          <div class="dashboard-item">
            <p class="item-label">계</p>
            <p class="item-value"> {{ breakSum }}회</p>
          </div>
        </div>
        <div class="break-deceleration">
          <p class="break-deceleration-label">누적 감속 값(브레이크)</p>
          <div class="dashboard-item">
            <p class="item-label"></p>
            <p class="item-value">{{ breakDeceleration }}km/h</p>
          </div>
        </div>
        <div class="accelerator-area">
          <p class="accelerator-area-label">감속 이벤트</p>
          <div class="dashboard-item">
            <p class="item-label">강</p>
            <p class="item-value">{{ accelStrong }}회</p>
          </div>
          <div class="dashboard-item">
            <p class="item-label">중</p>
            <p class="item-value">{{ accelMedium }}회</p>
          </div>
          <div class="dashboard-item">
            <p class="item-label">약</p>
            <p class="item-value">{{ accelWeak }}회</p>
          </div>
          <div class="dashboard-item">
            <p class="item-label">계</p>
            <p class="item-value">{{ accelSum }}회</p>
          </div>
        </div>
        <div class="break-deceleration">
          <p class="break-deceleration-label">누적 감속 값(가속도계)</p>
          <div class="dashboard-item">
            <p class="item-label"></p>
            <p class="item-value">{{ accelDeceleration }}km/h</p>
          </div>
        </div>
        <div class="total-Mileage">
          <p class="total-Mileage-label">총 주행거리</p>
          <div class="dashboard-item">
            <p class="item-label"></p>
            <p class="item-value">{{ totalMileage }}km/h</p>
          </div>
        </div>
			</div>
      <div class="style-controll-box">
        <p class="style-controll-label">주행 경로 스타일</p>
        <div class="style-controll-item color" @click="colorClick">
          <p class="item-label">선색</p>
          <p class="item-value">
            <span class="color-bg-pick on" style="background:#FF0000"></span>
            <span class="color-bg-pick" style="background:#07AE07"></span>
            <span class="color-bg-pick" style="background:#7A6FE0"></span>
            <span class="color-bg-pick" style="background:#5AB1A3"></span>
            <span class="color-bg-pick" style="background:#ED9F0F"></span>
            <span class="color-bg-pick" style="background:#009DFF"></span>
            <span class="color-bg-pick" style="background:#0015FF"></span>
            <span class="color-bg-pick" style="background:#656565"></span>
          </p>
        </div>
        <div class="style-controll-item opacity">
          <p class="item-label">투명도</p>
          <p class="item-value">
            <span class="plus" @click="valueUp">+</span>
            <span class="value">{{ opacityValue }}</span>
            <span class="minus" @click="valueDow">-</span>
          </p>
        </div>
        <div class="style-controll-item switch">
          <p class="item-label">표시여부</p>
          <p class="item-value">
            <span class="on">ON</span>
            <span>OFF</span>
          </p>
        </div>
      </div>
		</article>

	</section>
</template>

<style scoped>

</style>

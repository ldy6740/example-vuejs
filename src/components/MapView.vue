<script setup>
import { ref, onMounted, watch, defineProps} from 'vue';
import marker_blue from '../assets/images/marker_blue.png';
import marker_red from '../assets/images/marker_red.png';
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

// let showOverlay              = ref(false);     // custom overlay 화면 표시 여부 true or false

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
function dashboardData(newResponseData) {
  // "brake" 데이터 추출
  const breakList             = newResponseData.filter((data) => data.gubun === 'brake');
  if (breakList.length) {
    breakStrong.value         = breakList[breakList.length - 1].Calc_Value_sum1;
    breakMedium.value         = breakList[breakList.length - 1].Calc_Value_sum2;
    breakWeak.value           = breakList[breakList.length - 1].Calc_Value_sum3;
    breakSum.value            = Number(breakStrong.value) + Number(breakMedium.value) + Number(breakWeak.value);
    breakDeceleration.value   = breakList[breakList.length - 1].calc_value_sum.toFixed(2);
  }

  // "accel" 데이터 추출
  const accelList             = newResponseData.filter((data) => data.gubun === 'accel');
  if (accelList.length) {
    accelStrong.value         = accelList[accelList.length - 1].Calc_Value_sum1;
    accelMedium.value         = accelList[accelList.length - 1].Calc_Value_sum2;
    accelWeak.value           = accelList[accelList.length - 1].Calc_Value_sum3;
    accelSum.value            = Number(accelStrong.value) + Number(accelMedium.value) + Number(accelWeak.value);
    accelDeceleration.value   = accelList[accelList.length - 1].calc_value_sum.toFixed(2);
    totalMileage.value        = accelList[accelList.length - 1].Distance_Traveled;
  }
}

/**
 * 차량경로 선색 변경 함수
 */
function colorClick() {
  if (map.polyline) {
    let colorItem = document.querySelectorAll('.color-bg-pick');
    colorItem.forEach((item) => {
      item.addEventListener('click', (e) => {
        // on class 가 있으면 삭제
        colorItem.forEach((remove) => {
          remove.classList.remove('on');
        })
        e.target.classList.add('on'); // target에 on 클래스 삽입
        colorValue.value = getComputedStyle(e.target).backgroundColor; // target의 inline style 값 추출
      })
    });

    map.polyline.setOptions({
      strokeColor : colorValue.value,
    });
  }
}

function isShowPolyline() {
  if (map.polyline) {
    let isShowBtn = document.querySelectorAll('.is-show-polyline');

    isShowBtn.forEach((item) => {
      item.addEventListener('click', (e) => {
        isShowBtn.forEach((remove) => {
          remove.classList.remove('on');
        })
        e.target.classList.add('on');
        if (e.target.innerText === 'ON') {
          optionValueChange(opacityValue.value);
        } else {
          optionValueChange(0);
        }
      })
    })
  }
}



// 경로 투명도 조절
let opacityValue = ref(1);
let number = 10;
let value = '';

function valueUp() {
  if(map.polyline) {
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
  }
};
function valueDow() {
  if(map.polyline) {
    if(opacityValue.value == 0) {
      return;
    }

    if (number <= 10) {
      number -= 1;
      value = `0.${number}`;
    }

    opacityValue.value = Number(value === '0.0'? 0 : value);
    optionValueChange(opacityValue.value)
  }
};

function optionValueChange(value) {
  map.polyline.setOptions({
    strokeOpacity : value,
  })
}
// 운행경로를 지도에 표시하는 함수
const drawPolyline = (pathCoordinates) => {
  // 이전 경로를 지우기 위해 기존 폴리라인 삭제
  // if(props.isFunction) {
  //   if (map && map.polyline) {
  //     map.polyline.setMap(null);
  //   }
  // }
  if (map && map.polyline) {
    map.polyline.setMap(null);
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
      calc_value: item.calc_value,
      opacity: item.Calc_Value_opacity
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
      dashboardData(newResponseData);
      colorClick();
      isShowPolyline();
    }
  },
  {deep: true}
)


let markers = [];
// let oldMarker = null;
let markerAccelImages = null;
let markerBreakImages = null;
/**
 * Event 발생 지점 Marker 표시 및 마커 클릭시 이벤트 정보 표시
 * @param {*} eventDatas
 */
function eventPointMarker(eventDatas) {
// NOTE 해당 함수가 조회 버튼을 누르면 두번 실행 됨, 최초 1회는 정상,, 이후 조회 버튼을 클릭하면 첫번째 시도에서 데이터 없음.. 두번째 시도에 데이터가 전달 되어 watch에서 데이터 변화를 감지하고 함수 실행 getEventData -> eventPointMarker 순으로 함수 실행
  markers.forEach(marker => marker.setMap(null));
  if (markers) {
    markers = [];
  }

  let Marker = null;

  //마커 이미지 주소
  const accelImageSrc = marker_blue
	const breakImageSrc = marker_red
  // 지도에 마커 생성
  eventDatas.forEach((eventData, index) => {
    //	마커 이미지의 이미지 크기 입니다.
	  var imageSize	=	new window.kakao.maps.Size(32, 35);
    //	마커 이미지를 생성합니다.
	  markerAccelImages	=	new window.kakao.maps.MarkerImage(accelImageSrc, imageSize);
    markerBreakImages = new window.kakao.maps.MarkerImage(breakImageSrc, imageSize);

    // Marker 생성
    Marker = new window.kakao.maps.Marker({
      map				:	map,											        // 마커를 표시할 지도
      position	:	eventData.latlng,				          // 마커를 표시할 위치
      image			:	eventData.gubun === 'accel'
                  ? markerAccelImages : markerBreakImages,    // 이벤트별 마커 이미지 구분
      clickble  : true,                             // 마커 클릭이벤트 감지 옵션
      opacity   : Number(eventData.opacity),
    });

    // 마커 클릭 이벤트 감지
    window.kakao.maps.event.addListener(Marker, 'click', function() {
      clickMarkerVehicleNumber.value  = props.responseData[index].Vehicle_Number                          // 클릭한 마커의 차량번호 정보 추출
      clickMarkerGubun.value          = props.responseData[index].gubun === 'brake' ? '브레이크' : '감속';  // 클릭한 마커의 이벤트 명 추출
      clickMarkerCalcValue.value      = props.responseData[index].calc_value                              // 클릭한 마커의 감속정도 추출
    });
    // console.log(Marker);
    markers.push(Marker);
  });
  const breakMarkerHideBtn = document.getElementById('breakMarkerHide');
  const accelMarkerHideBtn = document.getElementById('accelMarkerHide');

  breakMarkerHideBtn.addEventListener('click', (e) => {
    markers.map((item) => {
      if (item.getImage().ok.slice(19).indexOf('.') === 10) {
        if(!item.getVisible()) {
          item.setVisible(true);
          e.target.innerText = 'OFF';
        } else {
          item.setVisible(false);
          e.target.innerText = 'ON';
        }
      }
    })
  });
  accelMarkerHideBtn.addEventListener('click', (e) => {
    markers.map((item) => {
      if (item.getImage().ok.slice(19).indexOf('.') === 11) {
        if(!item.getVisible()) {
          item.setVisible(true);
          e.target.innerText = 'OFF';
        } else {
          item.setVisible(false);
          e.target.innerText = 'ON';
        }
      }
    })
  });

}

function hideOverlay() {
  // showOverlay.value = false;
  // 클릭마커 정보 초기화
  clickMarkerVehicleNumber.value = "-";
  clickMarkerGubun.value = "-";
  clickMarkerCalcValue.value = "-";

  // oldMarker.setImage(clickMarkerGubun.value === 'brake' ? markerBreakImages : markerAccelImages);
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
        <div class="style-controll-item switch" @click="isShowPolyline">
          <p class="item-label">표시여부</p>
          <p class="item-value">
            <span class="is-show-polyline on">ON</span>
            <span class="is-show-polyline">OFF</span>
          </p>
        </div>
        <div class="marker-isHide">
          <p class="item-label">브레이크 마커 표시여부</p>
          <p class="item-value"><button id="breakMarkerHide">OFF</button></p>
        </div>
        <div class="marker-isHide">
          <p class="item-label">엑셀 마커 표시여부</p>
          <p class="item-value"><button id="accelMarkerHide">OFF</button></p>
        </div>
      </div>
		</article>
	</section>
</template>

<style scoped>

</style>

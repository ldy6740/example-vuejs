<script setup>
import { ref, onMounted, watch, defineProps} from 'vue';
const { VITE_KAKAO_MAP_KEY } = import.meta.env;

const mapCountainer = ref(null);
let breakStrong = ref(0);
let breakMedium = ref(0);
let breakWeak = ref(0);
let breakSum = ref(0);
let breakDeceleration = ref(0);
let accelStrong = ref(0);
let accelMedium = ref(0);
let accelWeak = ref(0);
let accelSum = ref(0);
let accelDeceleration = ref(0);
let totalMileage = ref(0);

let showOverlay = ref(false);
let overlayPosition = ref({top:0, left:0});

const props = defineProps({
  pointList: Array,
  isFunction: Boolean,
  eventPointData: Array,
  breakData: Array,
  searchAllData: Array
});


function dashboardData() {
  let items = props.searchAllData;
   // "brake" 데이터 추출
   const breakList = items.filter((data) => data.gubun === 'brake');
  if (breakList.length > 0) {
    breakStrong.value = breakList[0].Calc_Value_sum1;
    breakMedium.value = breakList[0].Calc_Value_sum2;
    breakWeak.value = breakList[0].Calc_Value_sum3;
    breakSum.value = Number(breakStrong.value) + Number(breakMedium.value) + Number(breakWeak.value);
    breakDeceleration.value = breakList[0].calc_value_sum.toFixed(2);
  }

  // "accel" 데이터 추출
  const accelList = items.filter((data) => data.gubun === 'accel');
  if (accelList.length > 0) {
    accelStrong.value = accelList[0].Calc_Value_sum1;
    accelMedium.value = accelList[0].Calc_Value_sum2;
    accelWeak.value = accelList[0].Calc_Value_sum3;
    accelSum.value = Number(accelStrong.value) + Number(accelMedium.value) + Number(accelWeak.value);
    accelDeceleration.value = accelList[0].calc_value_sum.toFixed(2);
    totalMileage.value = accelList[0].Distance_Traveled;
  }
}


let map;

onMounted(() => {
  loadKakaoMap(mapCountainer.value);
})


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
		path								:	pathCoordinates,							//	선을 구성하는 좌표 배열
		strokeWeight				:	5,											//	선의 두께
		strokeColor					:	"#FF0000",							//	선 색
		strokeOpacity				:	opacityValue.value,										//	선 투명도
		strokeStyle					:	"solid"									//	선 스타일
	});

};

const loadKakaoMap = (container) => {
  const script = document.createElement('script');
  script.src = `https:////dapi.kakao.com/v2/maps/sdk.js?appkey=${VITE_KAKAO_MAP_KEY}&autoload=false`
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

watch(
  () => props.pointList,
  (newList) => {
    if (map) {
      drawPolyline(newList);
      dashboardData();
    }
  },
  {deep: true}
)

// console.log(props.breakPoint);

watch(
  () => props.eventPointData,
  (newPosition) => {
    if(map) {
      eventPointMarker(newPosition);
    }
  },
  {deep: true}
)


let breakRawData = ref(null);
let markers = [];
let oldMaker = null;
let markerImages = null;
let testMakerImg = null;

function eventPointMarker(positions) {

  markers.forEach(marker => marker.setMap(null));
  markers = [];

  let Marker = null;

  //마커 이미지 주소
  const imageSrc =  "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png";
	const testImgSrc = 'https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/marker_red.png';

  positions.forEach((position, index) => {
    //	마커 이미지의 이미지 크기 입니다.
	  var imageSize	=	new window.kakao.maps.Size(24, 35);
    //	마커 이미지를 생성합니다.
	  markerImages	=	new window.kakao.maps.MarkerImage(imageSrc, imageSize);
    testMakerImg = new window.kakao.maps.MarkerImage(testImgSrc, imageSize);

    Marker = new window.kakao.maps.Marker({
      map						:	map,																	  //	마커를 표시할 지도
      id            : position.id,
      position			:	position.latlng,										//	마커를 표시할 위치
      title					:	position.title,										  //	마커의 타이틀, 마커에 마우스를 올리면 타이틀이 표시됩니다.
      image					:	position.gubun === 'accel'? testMakerImg : markerImages,
      clickble      : true,
			opacity				: 0.1
    });

    // console.log(positions[i]);

    window.kakao.maps.event.addListener(Marker, 'click', function() {

      breakRawData.value = props.breakData[index];

      var imageSize	=	new window.kakao.maps.Size(34, 45);
      //	마커 이미지를 생성합니다.
      var markerImage	=	new window.kakao.maps.MarkerImage(imageSrc, imageSize);

      // console.log(oldMaker);
      if (oldMaker) {
        oldMaker.setImage(markerImages);
      }
      this.setImage(markerImage);
      oldMaker = this;


      showOverlay.value = true;
    });

    markers.push(Marker);
  });
}



function hideOverlay() {
  showOverlay.value = false;
  oldMaker.setImage(markerImages);
}


// console.log(attrs);

</script>

<template>
  <section class="map-area" ref="mapCountainer">
		<article class="map-view" id="map">
			<div class="dashboard-area">
        <div class="dashboard-item">
					<p class="item-label">브레이크 이벤트(강)</p>
					<p class="item-value"> {{ breakStrong }}회</p>
				</div>
        <div class="dashboard-item">
					<p class="item-label">브레이크 이벤트(중)</p>
					<p class="item-value"> {{ breakMedium }}회</p>
				</div>
        <div class="dashboard-item">
					<p class="item-label">브레이크 이벤트(약)</p>
					<p class="item-value"> {{ breakWeak }}회</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">브레이크 이벤트(총합)</p>
					<p class="item-value"> {{ breakSum }}회</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">누적 감속 값(브레이크)</p>
					<p class="item-value">{{ breakDeceleration }}km/h</p>
				</div>
        <div class="dashboard-item">
					<p class="item-label">감속 이벤트(강)</p>
					<p class="item-value">{{ accelStrong }}회</p>
				</div>
        <div class="dashboard-item">
					<p class="item-label">감속 이벤트(중)</p>
					<p class="item-value">{{ accelMedium }}회</p>
				</div>
        <div class="dashboard-item">
					<p class="item-label">감속 이벤트(약)</p>
					<p class="item-value">{{ accelWeak }}회</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">감속 이벤트(총합)</p>
					<p class="item-value">{{ accelSum }}회</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">누적 감속 값(가속도계)</p>
					<p class="item-value">{{ accelDeceleration }}km/h</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">총 주행거리</p>
					<p class="item-value">{{ totalMileage }}km/h</p>
				</div>
        <div class="dashboard-item">
					<p class="item-label">투명도 조절</p>
					<p class="item-value">
            <span @click="valueUp">+</span>
            <span>{{ opacityValue }}</span>
            <span @click="valueDow">-</span>
          </p>
				</div>
			</div>
		</article>
    <article v-if="showOverlay" class="custom-overlay" :style="{ top: `${overlayPosition.top}px`, left: `${overlayPosition.left}px`, opacity: `${opacityValue}`}">
      <span @click="hideOverlay">닫기</span> <br>
      <div class="overlay-view">
        <p class="overlay-label">감속정도</p>
        <p class="overlay-value">{{ breakRawData.calc_value }}</p>
        <div class="opacity-controll-box">
          <span @click="valueUp">+</span>
          <span>{{ opacityValue }}</span>
          <span @click="valueDow">-</span>
        </div>
      </div>
    </article>
	</section>
</template>

<style scoped>

</style>

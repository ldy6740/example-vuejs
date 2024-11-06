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

const props = defineProps({
  pointList: Array,
  isFunction: Boolean,
  breakPoint: Array,
  breakData: Array,
  searchAllData: Array
});


function dashboardData() {
  let items = props.searchAllData;
   // "brake" 데이터 추출
   const breakList = items.filter((data) => data.gubun === 'brake');
  if (breakList.length > 0) {
    breakStrong.value = breakList[0].Calc_Value_sum3;
    breakMedium.value = breakList[0].Calc_Value_sum2;
    breakWeak.value = breakList[0].Calc_Value_sum1;
    breakSum.value = Number(breakStrong.value) + Number(breakMedium.value) + Number(breakWeak.value);
    breakDeceleration.value = breakList[0].calc_value_sum.toFixed(2);
  }

  // "accel" 데이터 추출
  const accelList = items.filter((data) => data.gubun === 'accel');
  if (accelList.length > 0) {
    accelStrong.value = accelList[0].Calc_Value_sum3;
    accelMedium.value = accelList[0].Calc_Value_sum2;
    accelWeak.value = accelList[0].Calc_Value_sum1;
    accelSum.value = Number(accelStrong.value) + Number(accelMedium.value) + Number(accelWeak.value);
    accelDeceleration.value = accelList[0].calc_value_sum.toFixed(2);
    totalMileage.value = accelList[0].Distance_Traveled;
  }
}


let map;

onMounted(() => {
  loadKakaoMap(mapCountainer.value);
})

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
		strokeWeight				:	10,											//	선의 두께
		strokeColor					:	"#FF0000",							//	선 색
		strokeOpacity				:	0.9,										//	선 투명도
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
  () => props.breakPoint,
  (newPosition) => {
    if(map) {
      brakePointMarker(newPosition);
    }
  },
  {deep: true}
)


let breakRawData = ref(null);
let markers = [];
// const iwContent = `<div style="padding:5px;">Hello World! <br> ${breakRawData.value}</div>`;

// console.log(breakRawData.value);

function brakePointMarker(positions) {

  markers.forEach(marker => marker.setMap(null));
  markers = [];

  //마커 이미지 주소
  const imageSrc =  "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png";

  positions.forEach((position, index) => {
    //	마커 이미지의 이미지 크기 입니다.
	  var imageSize	=	new window.kakao.maps.Size(24, 35);
    //	마커 이미지를 생성합니다.
	  var markerImage	=	new window.kakao.maps.MarkerImage(imageSrc, imageSize);

    let Marker = new window.kakao.maps.Marker({
      map						:	map,																	  //	마커를 표시할 지도
      id            : position.id,
      position			:	position.latlng,										//	마커를 표시할 위치
      title					:	position.title,										  //	마커의 타이틀, 마커에 마우스를 올리면 타이틀이 표시됩니다.
      image					:	markerImage,
      clickble      : true
    });

    map.InfoWindow = new window.kakao.maps.InfoWindow({
      position      : position.latlng,
      removable     : true
    });

    // console.log(positions[i]);

    window.kakao.maps.event.addListener(Marker, 'click', function() {

      breakRawData.value = props.breakData[index];
      map.InfoWindow.setContent(`<div style="padding:5px; line-height: 30px;">Hello World! <br> ${breakRawData.value.calc_value}</div>`);
      map.InfoWindow.open(map, Marker);
    })

    markers.push(Marker);
  });
  // console.log(breakRawData.value);
  // console.log(markers);
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
			</div>
		</article>
	</section>
</template>

<style scoped>

</style>

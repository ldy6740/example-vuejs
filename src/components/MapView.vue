<script setup>
import { ref, onMounted, watch, defineProps } from 'vue';
const { VITE_KAKAO_MAP_KEY } = import.meta.env;

const mapCountainer = ref(null);



const props = defineProps({
  pointList: Array,
  isFunction: Boolean,
  breakPoint: Array
});


let map;

onMounted(() => {
  loadKakaoMap(mapCountainer.value);
})

// console.log(props.pointList)


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
        level: 6,
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
    }
  },
  {deep: true}
)

console.log(props.breakPoint);

watch(
  () => props.breakPoint,
  (newPosition) => {
    if(map) {
      brakePointMarker(newPosition);
    }
  },
  {deep: true}
)


function brakePointMarker(position) {
  //마커 이미지 주소
  const imageSrc =  "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png";
  for ( var i = 0; i < position.length; i++ ) {
    //	마커 이미지의 이미지 크기 입니다.
	  var imageSize	=	new window.kakao.maps.Size(24, 35);
    //	마커 이미지를 생성합니다.
	  var markerImage	=	new window.kakao.maps.MarkerImage(imageSrc, imageSize);

    map.Marker = new window.kakao.maps.Marker({
      map						:	map,																	//	마커를 표시할 지도
      position			:	position[i].latlng,													//	마커를 표시할 위치
      title					:	position[i].title,														//	마커의 타이틀, 마커에 마우스를 올리면 타이틀이 표시됩니다.
      image					:	markerImage
    });
  }
}



// console.log(attrs);

</script>

<template>
  <section class="map-area" ref="mapCountainer">
		<article class="map-view" id="map">
			<div class="dashboard-area">
				<div class="dashboard-item">
					<p class="item-label">브레이크 이벤트</p>
					<p class="item-value">50회</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">누적 감속 값(브레이크)</p>
					<p class="item-value">50km/h</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">감속 이벤트</p>
					<p class="item-value">30회</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">누적 감속 값(가속도계)</p>
					<p class="item-value">50km/h</p>
				</div>
				<div class="dashboard-item">
					<p class="item-label">총 주행거리</p>
					<p class="item-value">100km/h</p>
				</div>
			</div>
		</article>
	</section>
</template>

<style scoped>

</style>

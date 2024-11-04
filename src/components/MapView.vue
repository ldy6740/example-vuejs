<script setup>
import { ref, onMounted } from 'vue';
const { VITE_KAKAO_MAP_KEY } = import.meta.env;

const mapCountainer = ref(null);

onMounted(() => {
  loadKakaoMap(mapCountainer.value);
})

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

      return  new window.kakao.maps.Map(container, options);
    })
  }
};

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

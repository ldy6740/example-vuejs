<script setup>
import MapView from './MapView.vue';
</script>

<template>
 	<section class="search-area">
		<!-- 조회 화면 -->
		<form id="form">
			<div class="input-box">
				<input type="text" id="vehicle-number" placeholder="차량번호를 입력하세요">
			</div>
			<div class="input-box">
				<input type="text" id="vehicle-date" placeholder="날짜">
			</div>
			<div class="input-box">
				<input type="text" id="start-day" placeholder="기간 선택">
			</div>
			<div class="select-box">
				<select type="text" id="start-time">
					<option value="">시작시간</option>
				</select>
			</div>
			<div class="select-box">
				<select type="text" id="end-time">
					<option value="">종료시간</option>
				</select>
			</div>
			<div class="button-box">
				<button type="submit" id="submit">조회</button>
			</div>
		</form>
  </section>
  <MapView />
</template>

<style scoped>

</style>

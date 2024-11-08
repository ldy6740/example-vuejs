<script setup>
// import GnbArea from './components/GnbArea.vue';
// import MapView from './components/MapView.vue';
</script>
<script>
import RealTimeSearch from './components/RealTimeSearch.vue';
import PeriodSearch from './components/PeriodSearch.vue';

export default {
  components: {
    PeriodSearch,
    RealTimeSearch
  },
  data() {
   return {
    currentTab: 'RealTimeSearch',
    tabs: ['RealTimeSearch', 'PeriodSearch']
   }
  }
}
</script>

<template>
  <!-- <GnbArea /> -->
  <div class="demo">

    <section class="header-area">
      <article id="logo" class="logo-area">
        <!-- <img src="" alt=""> -->
        logo
      </article>
      <nav class="gnb-area">
        <ul>
          <li
            v-for="tab in tabs"
            :key="tab.index"
            :class="['tab-button', {active: currentTab === tab}]"
            @click="currentTab = tab"
          >
            {{ tab === "RealTimeSearch" ? "실시간":"조회" }}
          </li>
        </ul>
      </nav>
    </section>

    <component :is="currentTab" class="tab"/>
  </div>
  <!-- <PeriodSearch></PeriodSearch> -->
   <!-- <RealTimeSearch></RealTimeSearch> -->

</template>



<style lang="scss">
// view css start

//hearer css
.header-area {
	background: #D9D9D9;
	display: flex;
	gap: 20px;
	padding: 10px;
	.logo-area {
		width: 150px;
		height: 40px;
		background: #B6B6B6;
		text-align: center;
		line-height: 40px;
	}
	.gnb-area {
		ul {
			display: flex;
			gap: 5px;
			li {
				width: 150px;
				height: 40px;
				background: #B6B6B6;
				border-radius: 5px;
				text-align: center;
				line-height: 40px;
				&.active {
					background: #656565;
					color: #fff;
				}
				&:hover {
					background: #656565;
					color: #fff;
				}
			}
		}
	}
}

// search css
.search-area {
	margin-top: 10px;
	background: #D9D9D9;
	padding: 5px 20px;
	height: 50px;
	display: flex;
	align-items: center;
	justify-content: end;
}

#form {
	display: flex;
	align-items: center;
	gap: 10px;
	& > div {
		& > * {
			width: 250px;
			height: 40px;
			border:1px solid #ededed;
			text-indent: 10px;
			border-radius: 5px;
		}
		.button-box {
			button {
				background: #B6B6B6;
			}
		}
	}
}


// map css
.map-area {
	margin-top: 10px;
	height: calc(100vh - 142px);
	position: relative;
	.map-view {
		background: #ededed;
		width: 100%;
		height: 100%;
	}
	.dashboard-area {
		position: absolute;
		top: 20px;
		left: 20px;
		display: flex;
    flex-wrap: wrap;
		align-items: center;
		gap: 10px;
		z-index: 999;
		.dashboard-item {
			background: #fff;
			border:1px solid #b9b9b9;
			border-radius: 10px;
			padding: 10px;
			min-width: 150px;
			height: 50px;
			display: flex;
			flex-direction: column;
			.item-label {
				font-size: 11px;
				font-weight: bold;
				flex: 1;
			}
			.item-value {
				flex: 1;
				text-align: right;
				font-size: 19px;
			}
		}
	}
  .custom-overlay {
    position: absolute;
    background: white;
    border:1px solid #ededed;
    padding: 10px;
    z-index: 999;
  }
}
</style>

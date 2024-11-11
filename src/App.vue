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
  .custom-overlay {
    position: absolute;
    background: rgba(255,255,255,.8);
    padding: 20px;
    z-index: 999;
    top: 10px;
    left: 10px;
    border-radius: 10px;
    .overlay-reset-btn {
      font-size: 13px;
      background: #ededed;
      padding: 5px;
      border-radius: 5px;
      position: absolute;
      right: 10px;
      top: 10px;
      cursor: pointer;
    }
    .overlay-view {
      display: flex;
      flex-direction: column;
      row-gap: 10px;
      .overlay-view-label {
        font-weight: bold;
        margin-bottom: 5px;
        font-size: 11px;
      }
      .overlay-item {
        border:1px solid red;
        background: #fff;
        border:1px solid #b9b9b9;
        border-radius: 13px;
        padding: 8px 14px;
        min-width: 150px;
        display: flex;
        flex-direction: row;
        column-gap: 5px;
        align-items: center;
        .overlay-label {
          font-size: 12px;
          font-weight: bold;
          flex: 1;
          color: #707070;
        }
        .overlay-value {
          flex: 1;
          text-align: right;
          font-size: 16px;
        }
      }
    }
  }
	.dashboard-area {
		position: absolute;
		top: 205px;
		left: 10px;
		display: flex;
    flex-direction: column;
    flex-wrap: wrap;
		align-items: center;
    row-gap: 10px;
		z-index: 999;
    background: rgba(255,255,255,.8);
    padding: 20px;
    border-radius: 10px;
    & > div {
      display:flex;
      flex-direction: column;
      row-gap: 5px;
      & > p {
        font-weight: bold;
        margin-bottom: 5px;
        font-size: 11px;
      }
      .dashboard-item {
        border:1px solid red;
        background: #fff;
        border:1px solid #b9b9b9;
        border-radius: 13px;
        padding: 8px 14px;
        min-width: 150px;
        display: flex;
        flex-direction: row;
        column-gap: 5px;
        align-items: center;
        .item-label {
          font-size: 12px;
          font-weight: bold;
          flex: 1;
          color: #707070;
        }
        .item-value {
          flex: 1;
          text-align: right;
          font-size: 13px;
        }
      }
    }
	}
  .style-controll-box {
    position: absolute;
    top: 10px;
    right: 10px;
    z-index: 999;
    background: rgba(255,255,255,.8);
    padding: 20px;
    display: flex;
    flex-direction: column;
    row-gap: 5px;
    .style-controll-label {
      font-weight: bold;
      margin-bottom: 5px;
      font-size: 11px;
    }
    .style-controll-item {
      border:1px solid red;
      background: #fff;
      border:1px solid #b9b9b9;
      border-radius: 13px;
      padding: 8px 14px;
      min-width: 150px;
      display: flex;
      flex-direction: row;
      align-items: center;
      &.color {
       flex-direction: column;
       align-items: flex-start;
       & > .item-value {
        margin-top: 5px;
        display: grid;
        grid-template-columns: 1fr 1fr 1fr 1fr;
        gap: 10px;
        & > span {
          width: 30px;
          height: 30px;
          cursor: pointer;
          border-radius: 5px;
          &.on {
            // border: 2px solid #0063c0;
            box-shadow: 0 0 0 3px #0063c0 inset;
          }
        }
       }
      }
      &.opacity {
        & > .item-value {
          display: flex;
          align-items: center;
          column-gap: 5px;
          & > .plus {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 15px;
            width: 15px;
            padding: 9px;
            cursor: pointer;
            background: #ededed;
          }
          & > .value {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 15px;
            width: 15px;
            padding: 9px;
            background: #ffffff;
          }
          & > .minus {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 15px;
            width: 15px;
            padding: 9px;
            cursor: pointer;
            background: #ededed;
          }
        }
      }
      &.switch {
        & > .item-value {
          display: flex;
          align-items: center;
          column-gap: 5px;
          & > span {
            display: flex;
            align-items: center;
            font-size: 12px;
            width: 25px;
            height: 15px;
            background: #ededed;
            padding: 9px;
            margin-left: 5px;
            cursor: pointer;
            &.on {
              background: #6d6d6d;
              color: #fff;
            }
          }
        }
      }
      .item-label {
        font-size: 12px;
        font-weight: bold;
        flex: 1;
        color: #707070;
      }
      .item-value {
        flex: 1;
        text-align: right;
        font-size: 16px;
      }
    }
  }
}
</style>

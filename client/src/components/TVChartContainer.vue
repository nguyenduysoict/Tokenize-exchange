<template>
  <div class="container">
    <div class="TVChartContainer" :id="container" />
    <div v-if="isShowInterval" class="interval-popup">
      <div class="popup-title">
        <div class="title">Set interval</div>
        <div class="icon-close" @click="isShowInterval = false">X</div>
      </div>
      <div class="popup-content">
        <div class="dis-flex">
          <input type="text" v-model="tmpInterval" />
          <div class="invalid-warning" v-if="!isValidInterval">
            Invalid interval
          </div>
        </div>
        <div class="mt-1">
          <button class="mr-1" @click="isShowInterval = false">Cancel</button>
          <button @click="applyInterval()">Confirm</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.dis-flex {
  display: flex;
}
.mt-1 {
  margin-top: 2rem;
}
.mr-1 {
  margin-right: 1rem;
}
.container {
  position: relative;
  height: 100%;
}
.popup-title {
  justify-content: space-between;
  display: flex;
  padding: 24px;
}
.popup-title .title {
  font-size: 20px;
  font-weight: 600;
}
.icon-close {
  cursor: pointer;
  font-weight: 600;
}
.invalid-warning {
  color: red;
  margin-left: 16px;
}

.interval-popup {
  width: 400px;
  position: absolute;
  top: 50%;
  left: 50%;
  margin-top: -130px;
  margin-left: -250px;
  background: white;
  border: 1px solid darkcyan;
}

.interval-popup .popup-content {
  padding: 24px;
}
</style>

<script>
import api from "./api";

export default {
  name: "TVChartContainer",
  props: {
    symbol: {
      default: "BTCUSDT",
      type: String,
    },
    interval: {
      default: "60",
      type: String,
    },
    container: {
      default: "tv_chart_container",
      type: String,
    },
    datafeedUrl: {
      default: "https://demo_feed.tradingview.com",
      type: String,
    },
    libraryPath: {
      default: "/charting_library/charting_library/",
      type: String,
    },
    chartsStorageUrl: {
      default: "https://saveload.tradingview.com",
      type: String,
    },
    chartsStorageApiVersion: {
      default: "1.2",
      type: String,
    },
    clientId: {
      default: "tradingview.com",
      type: String,
    },
    userId: {
      default: "public_user_id",
      type: String,
    },
    fullscreen: {
      default: false,
      type: Boolean,
    },
    autosize: {
      default: true,
      type: Boolean,
    },
    studiesOverrides: {
      type: Object,
    },
  },
  data() {
    return {
      tmpInterval: 60,
      originInterval: 60,
      isShowInterval: false,
      isValidInterval: true,
      validIntervals: [],
    };
  },
  tvWidget: null,
  methods: {
    applyInterval() {
      if (this.validIntervals.find((item) => item == this.tmpInterval.trim())) {
        this.isValidInterval = true;
        this.isShowInterval = false;
        this.tvWidget.setSymbol(this.symbol, this.tmpInterval, () => {
          this.originInterval = this.tmpInterval;
          console.log("Set interval success");
        });
      } else {
        this.isValidInterval = false;
      }
    },
  },
  created() {
    this.originInterval = this.interval;
  },
  mounted() {
    const widgetOptions = {
      symbol: this.symbol,
      // BEWARE: no trailing slash is expected in feed URL
      datafeed: api, // new window.Datafeeds.UDFCompatibleDatafeed(this.datafeedUrl),
      interval: this.interval,
      container_id: this.container,
      library_path: this.libraryPath,
      locale: "en",
      disabled_features: ["use_localstorage_for_settings"],
      // enabled_features: ['study_templates'],
      charts_storage_url: this.chartsStorageUrl,
      charts_storage_api_version: this.chartsStorageApiVersion,
      client_id: this.clientId,
      user_id: this.userId,
      fullscreen: this.fullscreen,
      autosize: this.autosize,
      studies_overrides: this.studiesOverrides,
      timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
    };

    this.tvWidget = new TradingView.widget(widgetOptions);

    this.tvWidget.onChartReady(() => {
      this.validIntervals = this.tvWidget.getIntervals();
      const button = this.tvWidget.createButton()[0];
      button.setAttribute("title", "Change time interval on chart");
      button.classList.add("apply-common-tooltip");
      button.addEventListener("click", () => {
        this.tmpInterval = this.originInterval;
        this.isValidInterval = true;
        this.isShowInterval = true;
      });
      button.innerHTML = "Set interval";
      this.tvWidget.addCustomCSSFile('../custom.css')
    });
  },

  destroyed() {
    if (this.tvWidget !== null) {
      this.tvWidget.remove();
      this.tvWidget = null;
    }
  },
};
</script>

<style>
.TVChartContainer {
  position: absolute;
  width: 100%;
  height: 100%;
}
</style>
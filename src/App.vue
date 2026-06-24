<template>
  <div class="container">

    <h2>Candle Aggregation Client</h2>

    <div class="form">

      <input
          v-model="symbol"
          placeholder="Symbol"
      />

      <select v-model="timeframe">
        <option value="1m">1m</option>
        <option value="5m">5m</option>
        <option value="15m">15m</option>
        <option value="30m">30m</option>
        <option value="60m">60m</option>
      </select>

      <input
          v-model="start"
          placeholder="Start Date"
      />

      <input
          v-model="end"
          placeholder="End Date"
      />

      <button @click="fetchCandles">
        Fetch Data
      </button>

    </div>

    <div
        v-if="errorMessage"
        class="error"
    >
      {{ errorMessage }}
    </div>

    <div id="chartContainer"></div>

  </div>
</template>

<script>
import axios from "axios";
import Highcharts from "highcharts/highstock";
import "./assets/App.css";

export default {

  data() {
    return {

      symbol: "TCS",
      timeframe: "5m",

      start: "2026-01-01T12:00:00Z",
      end: "2026-01-01T12:59:00Z",

      errorMessage: ""
    };
  },

  methods: {

    async fetchCandles() {

      this.errorMessage = "";

      try {

        const response = await axios.get(
            "http://localhost:8080/api/v1/candles",
            {
              params: {
                symbol: this.symbol,
                timeFrame: this.timeframe,
                startDate: this.start.replace("Z", ""),
                endDate: this.end.replace("Z", "")
              }
            }
        );

        const chartData =
            response.data.candles.map(c => [

              new Date(c.datetime).getTime(),

              c.open,
              c.high,
              c.low,
              c.close
            ]);

        Highcharts.stockChart(
            "chartContainer",
            {
              title: {
                text: "Candlestick Chart"
              },

              rangeSelector: {
                selected: 1
              },

              series: [
                {
                  type: "candlestick",
                  name: this.symbol,
                  data: chartData
                }
              ]
            }
        );

      } catch (error) {

        console.error(error);

        if (error.response) {

          this.errorMessage =
              error.response.data.error ||
              "Request Failed";

        } else {

          this.errorMessage =
              "Unable to connect to server";
        }
      }
    }
  }
};
</script>
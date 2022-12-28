<template>
  <div class="p-2 border border-gray-500 mt-4">
    <label class="block mb-2 font-bold"> Coba Chart </label>

    <FunctionChart
      :chartData="chartData"
      :options="{ responsive: true, mantainAspectRatio: false }"
    />
    <button @click="changeChartData">Change data</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      chartDataValues: [40, 39, 10, 40, 39, 80, 40],
    };
  },
  computed: {
    chartData() {
      return {
        labels: [
          "January",
          "February",
          "March",
          "April",
          "May",
          "June",
          "July",
        ],
        datasets: [
          {
            label: "Data One",
            backgroundColor: "#f87979",
            data: this.chartDataValues,
          },
        ],
      };
    },
  },
  methods: {
    changeChartData() {
      this.chartDataValues = this.chartDataValues.sort(
        (a, b) => 0.5 - Math.random()
      );
    },
  },
};
</script>

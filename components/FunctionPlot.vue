<template>
  <div id="root"></div>
</template>

<script>
import functionPlot from "function-plot";

export default {
  mounted() {
    console.log("mounted");
    functionPlot({
      target: "#root",
      width: 500,
      height: 800,
      grid: true,
      data: [
        {
          fn: "x^2",
          derivative: {
            fn: "2 * x",
            updateOnMouseMove: true,
          },
        },
      ],
    });
  },
};
</script>

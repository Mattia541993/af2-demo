<template>
  <div>
    <vue-web-speech-synth v-model="shouldRead" :text="textToRead" />
    <Keypress key-event="keyup" :key-code="13" @success="logKey" />
    <p>{{ fun }}</p>
    <div id="root"></div>
    <button v-tooltip="'Tooltip sul bottone'" @click="changeChartData">
      Change data
    </button>
    <v-popover>
      <button>Click me</button>
    </v-popover>
  </div>
</template>

<script>
import functionPlot from "function-plot";

export default {
  data() {
    return {
      fun: "x",
      shouldRead: false,
      textToRead: "",
    };
  },

  components: {
    Keypress: () => import("vue-keypress"),
  },
  mounted() {
    console.log("mounted");
    // window.addEventListener("keydown", function (e) {
    //   console.log(
    //     `premuto tasto ${e.which}, ${String.fromCharCode(e.keyCode)}`
    //   );
    // });
    this.updateFunctionChart();
  },

  methods: {
    changeChartData() {
      this.fun = "x^2";
      this.updateFunctionChart();
    },
    logKey(event) {
      console.log(event);
    },
    updateFunctionChart() {
      console.log(`window ${window}`);
      const fnPlotInstance = functionPlot({
        target: "#root",
        width: 500,
        height: 800,
        grid: true,
        data: [
          {
            fn: this.fun,
            derivative: {
              fn: "2 * x",
              updateOnMouseMove: true,
            },
          },
        ],
      });
      fnPlotInstance.on("mouseover", function () {
        console.log(`mouseover ${window.speechSynthesis}`);
        // setTimeout(() => {
        //   window.speechSynthesis.speak(
        //     new SpeechSynthesisUtterance("Hello world!")
        //   );
        // }, 100);
      });
    },
  },
};
</script>

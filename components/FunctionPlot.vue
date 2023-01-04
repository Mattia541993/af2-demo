<template>
  <div>
    <vue-web-speech-synth
      v-model="shouldRead"
      :text="textToRead"
      :voice="ttsVoice"
      @list-voices="listVoices"
    />
    <div v-if="isFunctionInteractionModeEnabled">
      <Keypress key-event="keyup" :key-code="13" @success="logKey" />
      <Keypress key-event="keyup" :key-code="65" @success="logKey" />
      <Keypress key-event="keyup" :key-code="83" @success="logKey" />
      <!-- TODO: Aggiungere shortcut per attivare disattivare modalità interazione con funzione -->
    </div>

    <select v-model="synthVoice">
      <option
        v-for="(voice, index) in voiceList"
        :key="index"
        :value="voice.name"
      >
        {{ voice.name }} [{{ voice.lang }}]
        <!-- {{ voice }} -->
      </option>
    </select>
    <input
      type="checkbox"
      id="enableFnCheckbox"
      v-model="isFunctionInteractionModeEnabled"
    />
    <label for="enableFnCheckbox"
      >Interazione tramite tastiera
      {{
        isFunctionInteractionModeEnabled ? "abilitata" : "disabilitata"
      }}</label
    >
    <p>{{ fun }}</p>
    <div id="root" role="application" tabindex="0"></div>
    <button v-tooltip="'Tooltip sul bottone'" @click="changeChartData">
      Change data
    </button>
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
      voiceList: [],
      synthVoice: null,
      isFunctionInteractionModeEnabled: false,
    };
  },

  components: {
    Keypress: () => import("vue-keypress"),
  },
  computed: {
    ttsVoice() {
      return this.voiceList.find((elem) => elem.name == this.synthVoice);
    },
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
    listVoices(list) {
      this.voiceList = list;
    },
    logKey(event) {
      // debugger;

      console.log(event);
      this.textToRead = `premuto ${event.event.key}`;
      this.shouldRead = true;
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

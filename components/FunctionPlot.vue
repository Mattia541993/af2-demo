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
      <Keypress key-event="keydown" :key-code="39" @success="handleKeyDown" />
      <Keypress key-event="keyup" :key-code="39" @success="handleKeyUp" />
      <!-- TODO: Aggiungere shortcut per attivare disattivare modalità interazione con funzione -->
    </div>

    <header ref="header">
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
      <mathlive-mathfield
        ref="mathfield"
        id="mf"
        :options="{
          virtualKeyboardMode: 'onfocus',
          keypressSound: 'none',
        }"
      >
      </mathlive-mathfield>
      <p>{{ fun }}</p>
      <p>Formula: {{ inputFunctionLatex }}</p>
    </header>

    <div
      class="d-flex flex-column w-100 h-100"
      id="functionContainer"
      ref="fnContainer"
    >
      <div class="w-100 h-100" style="position: relative">
        <resize-observer @notify="handleResize" :emitOnMount="true" />

        <div role="application" tabindex="0" aria-label="area del grafico">
          <div id="root" aria-hidden="true"></div>
        </div>
      </div>
      <footer ref="footer" class="w-100" style="background-color: blue">
        <button v-tooltip="'Tooltip sul bottone'" @click="changeChartData">
          Change data
        </button>
        <b-button variant="danger">Prova bottone con bootstrap vue</b-button>
      </footer>
    </div>
  </div>
</template>

<script>
import functionPlot from "function-plot";
import * as Tone from "tone";
import $ from "jquery";

export default {
  data() {
    return {
      fun: "x",
      shouldRead: false,
      textToRead: "",
      voiceList: [],
      synthVoice: null,
      isFunctionInteractionModeEnabled: false,
      instrument: new Tone.Synth().toDestination(),
      keyLongPressed: false,
      keyTimer: null,
      currentFnXValue: 0,
      fnContainerWidth: 0,
      fnContainerHeight: 0,
      inputFunctionLatex: "",
    };
  },
  watch: {
    fnContainerWidth: {
      handler(newVal) {
        this.updateFunctionChart();
      },
      immediate: true,
    },
    fnContainerHeight: {
      handler(newVal) {
        this.updateFunctionChart();
      },
      immediate: true,
    },
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
    const mathField = document.getElementById("mf");
    mathField.addEventListener("change", (evt) => {
      //Return o enter premuto
      this.inputFunctionLatex = evt.target.value;
      console.log(`focus-out. Latex: ${evt.target.value}`);
    });
    mathField.addEventListener("focus-out", (evt) => {
      //Quando col tab lascio il campo di input
      // debugger;
      this.inputFunctionLatex = evt.target.value;
      console.log(`focus-out. Latex: ${evt.target.value}`);
    });
    // debugger;
    // this.$refs.mathfield.addEventListener("input", (ev) =>
    //   console.log("input")
    // );
  },

  methods: {
    changeChartData() {
      this.fun = "x^2";
      this.updateFunctionChart();
      this.instrument.triggerAttackRelease("C4", "4n");
    },
    resizeWindowAmt(amt) {
      window.resizeBy(amt, 0);
    },
    resizeWindow() {
      window.dispatchEvent(new Event("resize"));
    },
    logKeystroke(key, _ev) {
      console.log("keystroke");
      return true;
    },
    ping: () => {
      console.log("ping");
    },
    onNewFnInputValue(newVal) {
      console.log(`new fun val ${newVal}`);
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
    handleResize({ width, height }) {
      // setTimeout(() => {
      //   console.log(`Sul resize timeout ${$("#functionContainer").height()}`);
      // }, 100);
      // console.log(`resize ${width} ${height}`);

      this.fnContainerWidth = width;
      this.fnContainerHeight = height;

      // this.updateFunctionChart();
    },

    handleKeyDown(event) {
      console.log(`${event.event.key} DOWN`);
      if (this.keyTimer == null) {
        var repeat = function () {
          // debugger;
          this.currentFnXValue += 10;
          this.keyTimer = setTimeout(repeat, 200);
          console.log(`current fn x value: ${this.currentFnXValue}`);
        }.bind(this); //è necessario bindare il this alla funzione, altrimenti verrebbe perso il this dell'istanza di Vue all'interno della funzione: https://lusaxweb.github.io/vuesax-blog/tips/scope_this.html#create-var
        repeat();
      }
    },
    handleKeyUp(event) {
      console.log(`${event.event.key} UP`);
      if (this.keyTimer != null) {
        clearTimeout(this.keyTimer);
        this.keyTimer = null;
        this.currentFnXValue = 0;
      }
    },
    updateFunctionChart() {
      // debugger;

      const fnPlotInstance = functionPlot({
        target: "#root",
        width: this.fnContainerWidth,
        height: this.fnContainerHeight * 0.99,
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

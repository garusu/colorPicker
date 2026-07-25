<script setup>
  import { ref, reactive, computed, onMounted, onUnmounted } from 'vue';

  const hue = ref(null);
  const square = ref(null);

  const huePoint = ref(null);
  const squarePoint = ref(null);

  const squareX = ref(0);
  const squareY = ref(0);
  const hueX = ref(0);

  let dragging = false;

  const mouseDown = (elem, event) => {
    event.preventDefault(); 
    dragging = elem; 
    mouseMove(event)
  }
  const mouseUp = () => dragging = false;

  class Color {
    constructor() {
      this.hsv = reactive({
        h: 0,
        s: 0,
        v: 1
      })
      
      this.rgb = computed({
        get: () => {
          return hsvToRgb(this.hsv)
        },
        set: (value) => {
          value = RgbToHsv(value)
          Object.assign(this.hsv, value)

          hueX.value = (value.h / 360) * hue.value.offsetWidth
          squareX.value = value.s * square.value.offsetWidth
          squareY.value = (1 - value.v) * square.value.offsetHeight
        }
      })

      this._hex = computed({
        get: () => {
          return (
            "#" +
            this.rgb.value.r.toString(16).padStart(2, "0") +
            this.rgb.value.g.toString(16).padStart(2, "0") +
            this.rgb.value.b.toString(16).padStart(2, "0")
          ).toUpperCase();
        },
        set: (value) => {
          value = value.trim();
          value = value.replace('#', '');

          if (!/^[0-9A-Fa-f]+$/.test(value)) {value = "ffffff"}
          if (value.length == 3) { value = value.split("").map(e => e + e).join("") }

          if (value.length == 6) {
            this.rgb.value = {
              r: parseInt(value.substring(0, 2), 16),
              g: parseInt(value.substring(2, 4), 16),
              b: parseInt(value.substring(4, 6), 16)
            }
          }
        }
      })
    }

    get cssCleanColor() {
      return `hsl(${this.hsv.h}deg 100% 50%)`
    }
    get r() {
      return this.rgb.value.r 
    }
    get g() {
      return this.rgb.value.g 
    }
    get b() {
      return this.rgb.value.b
    }
    get hex() {
      return this._hex.value
    }
    set r(value) {
      this.changeRgb(value, "r");
    }
    set g(value) {
      this.changeRgb(value, "g");
    }
    set b(value) {
      this.changeRgb(value, "b");
    }
    set hex(value) {
      this._hex.value = value;
    }
    changeRgb(value, letter) {
      this.rgb.value = {
        ...this.rgb.value, 
        [letter]: Math.min(255, parseInt(value) || 0)
      }
    }
  }

  const color = new Color();

  onMounted(() => {
    document.addEventListener('pointerup', mouseUp);
    document.addEventListener("pointermove", mouseMove);
  });
  onUnmounted(() => {
    document.removeEventListener('pointerup', mouseUp);
    document.removeEventListener("pointermove", mouseMove);
  });

  function mouseMove(event) {
    if (dragging == "square") {
      squareX.value = Math.max(
        0,
        Math.min(
          event.clientX - square.value.getBoundingClientRect().left, 
          square.value.offsetWidth
        )
      );
      squareY.value = Math.max(
        0,
        Math.min(
          event.clientY - square.value.getBoundingClientRect().top, 
          square.value.offsetHeight
        )
      );

      color.hsv.s = squareX.value / square.value.offsetWidth;
      color.hsv.v = 1 - squareY.value / square.value.offsetHeight;
    }
    else if (dragging == "hue") {
      hueX.value = Math.max(
        0,
        Math.min(
          event.clientX - hue.value.getBoundingClientRect().left,
          hue.value.offsetWidth
        )
      );

      color.hsv.h = hueX.value / hue.value.offsetWidth * 360;
    }
  }

  function hsvToRgb(hsv) {
    const h = ((hsv.h % 360) + 360) % 360;

    const c = hsv.v * hsv.s;
    const x = c * (1 - Math.abs((h / 60) % 2 - 1));
    const m = hsv.v - c;

    let r = 0, g = 0, b = 0;

    if (h < 60) {
      r = c; g = x; b = 0;
    } else if (h < 120) {
      r = x; g = c; b = 0;
    } else if (h < 180) {
      r = 0; g = c; b = x
    } else if (h < 240) {
      r = 0; g = x; b = c
    } else if (h < 300) {
      r = x; g = 0; b = c;
    } else {
      r = c; g = 0; b = x;
    }

    return {
      r: Math.round((r + m) * 255),
      g: Math.round((g + m) * 255),
      b: Math.round((b + m) * 255)
    }
  }

  function RgbToHsv(rgb) {
    const r = rgb.r / 255;
    const g = rgb.g / 255;
    const b = rgb.b / 255;

    let h = 0;
    const v = Math.max(r, g, b);
    const c = v - Math.min(r, g, b);

    if (c) {
      if (v == r) {
        h = (g - b) / c
      } else if (v == g) {
        h = 2 + (b - r) / c
      } else {
        h = 4 + (r - g) / c
      }
    }
    if (h < 0) {
      h += 6
    }
    h *= 60

    return {
      h: h,
      s: v && c / v,
      v: v
    }
  }
</script>

<template>
  <div id="container" :style="{backgroundColor: `rgb(${color.r} ${color.g} ${color.b})`}">
    <div id="picker">
      <div 
        id="square" 
        ref="square"
        :style="{ backgroundColor: color.cssCleanColor }"
        @pointerdown.left="mouseDown('square', $event)" 
        @dragstart.prevent
      >
        <div 
          class="cursor" 
          ref="squarePoint" 
          :style="{ 
            backgroundColor: `rgb(${color.r} ${color.g} ${color.b})`, 
            left: `${squareX}px`, top: `${squareY}px` 
          }"
          @dragstart.prevent
        ></div>
      </div>
      <div 
        id="hue"
        ref="hue"
        @pointerdown.left="mouseDown('hue', $event)"
        @dragstart.prevent
      >
        <div 
          class="cursor"
          ref="huePoint"
          :style="{ backgroundColor: color.cssCleanColor, left: `${hueX}px` }"
        ></div>
      </div>
      <div id="inputs">
        <div>
          <p>Red</p>
          <input maxlength="3" :value="color.r" @blur="event => {color.r = event.target.value}" >
        </div>
        <div>
          <p>Green</p>
          <input maxlength="3" :value="color.g" @blur="event => {color.g = event.target.value}" >
        </div>
        <div>
          <p>Blue</p>
          <input maxlength="3" :value="color.b" @blur="event => {color.b = event.target.value}" >
        </div>
      </div>
      <div id="hex">
        <p>HEX</p>
        <input maxlength="7" :value="color.hex" @blur="event => {color.hex = event.target.value}" >
      </div>
    </div>
  </div>
</template>

<style scoped>
  p {
    font-size: 0.8rem;
    margin: 0;
  }
  #container {
    touch-action: none;
    display: grid;
    place-items: center;
    height: 100vh;
  }
  #picker {
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 3px 2px grey;
    background-color: whitesmoke;
    width: min(60%, 300px);

  }
  #hex {
    margin-top: 10px;
    width: 100%;
  }
  #inputs {
    display: flex;
    justify-content: space-between;
    gap: 10px;
    width: 100%;

    div {
      margin-top: 20px;
      width: 30%;
    }
  }
  input {
    border: 2px solid #00000080;
    padding: 3px;
    width: 100%;
    box-sizing: border-box;
    font-size: 1rem;
  }
  .cursor {
    box-shadow: 0 0 1px 2px black;
    width: 18px;
    height: 18px;
    border-radius: 100%;
    margin: 0;
    position: relative;
  }
  #square {
    width: 100%;
    height: 300px;
    box-shadow: 0 0 1px 2px black;
    background: 
      linear-gradient(to top, black, transparent), 
      linear-gradient(to right, white, transparent);

    .cursor {
      translate: -50% -50%;
    }
  }
  #hue {
    margin-top: 20px;
    width: 100%;
    height: 10px;
    box-shadow: 0 0 1px 2px black;
    background: linear-gradient(to right, red, yellow, lime, cyan, blue, magenta, red);

    .cursor {
      margin: 0;
      position: relative;
      translate: -50% -4px;
    }
  }
</style>

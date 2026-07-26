<script setup>
  import { ref, reactive, computed, onMounted, onUnmounted } from 'vue';

  const hue = ref(null);
  const square = ref(null);

  const { color } = defineProps({
      color: {type: Object, required: true}
  })

  let dragging = false;

  const mouseDown = (elem, event) => {
    event.preventDefault(); 
    dragging = elem; 
    mouseMove(event)
  }
  const mouseUp = () => dragging = false;

  onMounted(() => {
    color.square.value = square.value;
    color.hue.value = hue.value;
    document.addEventListener('pointerup', mouseUp);
    document.addEventListener("pointermove", mouseMove);
  });
  onUnmounted(() => {
    document.removeEventListener('pointerup', mouseUp);
    document.removeEventListener("pointermove", mouseMove);
  });

  function mouseMove(event) {
    if (dragging == "square") {
      color.squareX.value = Math.max(
        0,
        Math.min(
          event.clientX - color.square.value.getBoundingClientRect().left, 
          color.square.value.offsetWidth
        )
      );
      color.squareY.value = Math.max(
        0,
        Math.min(
          event.clientY - color.square.value.getBoundingClientRect().top, 
          color.square.value.offsetHeight
        )
      );

      color.hsv.s = color.squareX.value / square.value.offsetWidth;
      color.hsv.v = 1 - color.squareY.value / color.square.value.offsetHeight;
    }
    else if (dragging == "hue") {
      color.hueX.value = Math.max(
        0,
        Math.min(
          event.clientX - color.hue.value.getBoundingClientRect().left,
          color.hue.value.offsetWidth
        )
      );

      color.hsv.h = color.hueX.value / color.hue.value.offsetWidth * 360;
    }
  }
</script>

<template>
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
        :style="{ 
          backgroundColor: `rgb(${color.r} ${color.g} ${color.b})`, 
          left: `${color.squareX.value}px`, top: `${color.squareY.value}px` 
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
        :style="{ backgroundColor: color.cssCleanColor, left: `${color.hueX.value}px` }"
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
</template>

<style scoped>
  p {
    font-size: 0.8rem;
    margin: 0;
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

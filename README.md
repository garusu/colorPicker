<p align="center">
	<img src="docs/logo.png">
</p>
<h1 align="center">Color Picker</h1>

This is a minimalistic and fast component allows users to select, fine-tune, and preview colors, with built-in support for converting between HEX and RGB formats

<p align="center">
	<img src="./docs/screenshot.png" alt="Color Picker Screenshot" width="60%">
</p>
<p align="center">
	<img src="./docs/colorpicker.gif" alt="Color Picker GIF" width="60%">
</p>

## Features

* Conversion color to RGB and HEX models
* Direct text input
* Dynamic color change
* Input validation
* Touch optimization

## Use

Import the components:

```js
import { Color } from './components/color.js'
import Picker from './components/Picker.vue'
```

Create a new instance of the "Color" class:

```js
const color = new Color();
```

Add the component to the template:

```vue
<template>
  <Picker :color="color"></Picker>
</template>
```

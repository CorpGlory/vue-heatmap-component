# vue-heatmap-component

## Quick start

1. Install using npm: `npm install CorpGlory/vue-heatmap-component`
2. Import the component: `import VueHeatmapComponent from 'vue-heatmap-component';`
3. [Register](https://vuejs.org/v2/guide/components-registration.html) the component globally or in a separate component

## Usage example
```vue
<template>
  <vue-heatmap-component
    :data="[
      { x: 'x1', y: 'y1',  value: 10 }, 
      { x: 'x1', y: 'y2',  value: 60 },
      { x: 'x2', y: 'y1',  value: 30 },
      { x: 'x2', y: 'y2',  value: 100 },
    ]"
    :axisX=['x1', 'x2']
    :axisY=['y1', 'y2']
  />
</template>
...
```

Result:

![image](https://user-images.githubusercontent.com/39257464/71175019-8a99a880-2277-11ea-947a-dff4771c73a3.png)

## Props
- `data` — (required) array of values as matrix with dimension NxN, e.g:
```js
:data=[
  { x: 'x1', y: 'y1',  value: 10 }, 
  { x: 'x1', y: 'y2',  value: 60 },
  { x: 'x2', y: 'y1',  value: 30 },
  { x: 'x2', y: 'y2',  value: 100 },
]
```
- `axisX` — (required) array of x-axis variables, e.g:
```js
:axisX=['x1', 'x2']
```
- `axisY` - (required) array of y-axis variables, e.g:
```js
:axisY=['y1', 'y2']
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

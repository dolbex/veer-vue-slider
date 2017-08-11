# veer-vue-slider

## About

Veer is a simple Vue Js (v2) slider that attempts to handle whatever you want to throw at it. Please let me know what scenarios throw this thing off so I can adjust.

## Basic Usage

```
<div id="app">
  <veer>
    <img src="/static/davide-cantelli-139887.jpg">
    <img src="/static/eugene-lim-260668.jpg">
    <img src="/static/paul-earle-183430.jpg">
  </veer>
</div>
```

## Options

You can pass your options through in an object through the options prop.

```
  <veer :options="{autoplay:false}">
    ...
  </veer>
```

Below are the available options and the defaults

```
autoplay: true,
mode: 'fade',
initialDelay: 0,
speed: 4000,
showPrevNext: true,
showPages: true,
pageClass: 'veer-pages',
arrowClass: 'veer-arrows',
width: 'auto',
height: 'auto'
```

## Examples

Clone the project and run

```npm run dev```

Modify the index.html file

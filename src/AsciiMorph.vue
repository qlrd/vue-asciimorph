<template>
  <pre v-html="rawHtml" :style="`font-size: ${fontSize};`"/>
</template>

<script setup lang="ts">
/**
 * Ascii Morph
 * @author: Tim Holman (http://tholman.com)
 * 
 * Modified for vue/typescript by @qlrd
 */
import { toRefs, ref, type Ref, onMounted, watch } from 'vue';

interface CanvasDimensions{
  x: number;
  y: number;
}

/**
 * Props
 */
const props = defineProps<{
  list: string[][],
  index: number,
  canvas: CanvasDimensions ,
  timeout: number,
  fontSize: string
}>()

/**
 * Variables
 */
const { list, index, canvas,timeout, fontSize} = toRefs(props)
const rawHtml: Ref<string> = ref('') 
const framesToAnimate: Ref<string[][]> = ref([['']])
const renderedData: Ref<string[]> = ref([''])
const myTimeout: Ref<ReturnType<typeof setTimeout>> = ref(setTimeout(() => null, timeout.value))

/**
 * Methods
 */

/**
 * Render to a new frame
 * @param data
 */
function render (data: string[]): void {
  const ourData = squareOutData(data).slice();
  renderSquareData(ourData);
}

/**
 * Morph between whatever is current, to the new frame
 * @param data
 */
function morph(data: string[]): void {
  clearTimeout(myTimeout.value);
  const frameData = prepareFrames(data.slice());
  animateFrames(frameData);
}

/**
 * Animate frames 
 * @param frameData 
 */
function animateFrames(frameData: string[][]) {
  framesToAnimate.value = frameData;
  animateFrame();
}

/**
 * Animate a frame
 */
function animateFrame() {
  myTimeout.value = setTimeout(() => {
    renderSquareData(framesToAnimate.value[0]);
    framesToAnimate.value.shift();
    if( framesToAnimate.value.length > 0 ) {
      animateFrame();
    }
  }, timeout.value)
  // framesToAnimate
}

/**
 * Render data in a square 
 * @param data 
 */
function renderSquareData(data: string[]): void {
  rawHtml.value = '';
  for( var i = 0; i < data.length; i++ ) {
    rawHtml.value = rawHtml.value + data[i] + '\n';
  }
  renderedData.value = data;
}

/**
 * Replace characters in renderization
 * @param str 
 * @param index 
 * @param character 
 */
function replaceAt(str: string, index: number, character: string): string {
  return str.substring(0, index) + character + str.substring(index+character.length);
}

/**
 * Calculate positions of data in a square
 * @param data 
 */
function squareOutData(data: string[]): string[] {
  let i;
  const renderDimensions: CanvasDimensions = {
    x: 0,
    y: data.length
  };

  // Calculate centering numbers
  for( i = 0; i < data.length; i++ ) {
    if( data[i].length > renderDimensions.x) {
      renderDimensions.x = data[i].length
    }
  }

  // Pad out right side of data to square it out
  for( i = 0; i < data.length; i++ ) {
    if( data[i].length < renderDimensions.x) {
      data[i] = (data[i] + repeat(' ', renderDimensions.x - data[i].length ));
    }
  }

  const paddings: CanvasDimensions = {
    x: Math.floor((canvas.value.x - renderDimensions.x) / 2),
    y: Math.floor((canvas.value.y - renderDimensions.y) / 2)
  }

  // Left Padding
  for( let i = 0; i < data.length; i++ ) {
    data[i] = repeat(' ', paddings.x) + data[i] + repeat(' ', paddings.x);
  }

  // Pad out the rest of everything
  for( let i = 0; i < canvas.value.y; i++ ) {
    if( i < paddings.y) {
      data.unshift(repeat(' ', canvas.value.x));
    } else if (i > (paddings.y + renderDimensions.y)) {
      data.push(repeat(' ', canvas.value.x));
    }
  }
  return data;
}
    
/**
 * Repeat patterns 
 * @param pattern 
 * @param count 
 */
function repeat(pattern: string, count: number): string {
  if (count < 1) return '';
  let result = '';
  while (count > 1) {
    if (count & 1) result += pattern;
    count >>= 1, pattern += pattern;
  }
  return result + pattern;
}

/**
 * Crushes the frame data by 1 unit.
 * @param data 
 */
function getMorphedFrame(data: string[]): string[] | boolean {
  let firstInLine, lastInLine = null;
  let found = false;
  for( let i = 0; i < data.length; i++) {
    let line = data[i];
    firstInLine = line.search(/\S/);
    if( firstInLine === -1) {
      firstInLine = null;
    }

    for( var j = 0; j < line.length; j++) {
      if( line[j] != ' ') {
        lastInLine = j;
      }
    }

    if( firstInLine !== null && lastInLine !== null) {
      data = crushLine(data, i, firstInLine, lastInLine)
      found = true;
    }
    
    firstInLine = null, lastInLine = null;
  }

  if( found ) {
    return data;
  } else {
    return false;
  }
}

/**
 * Crushes the frame data
 * @param data 
 * @param line 
 * @param start 
 * @param end 
 */
function crushLine(data: string[], line: number, start: number, end: number): string[] {
  let centers: CanvasDimensions = {
    x: Math.floor(canvas.value.x / 2),
    y: Math.floor(canvas.value.y / 2)
  }
      
  var crushDirection = 1;
  if( line > centers.y ) {
    crushDirection = -1;
  }

  let charA = data[line][start];
  let charB = data[line][end];

  data[line] = replaceAt(data[line], start, " ");
  data[line] = replaceAt(data[line], end, " ");

  if( !((end - 1) == (start + 1)) && !(start === end) && !((start + 1) === end)) {
    data[line + crushDirection] = replaceAt(
      data[line + crushDirection],
      (start + 1), 
      '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
    );
    
    data[line + crushDirection] = replaceAt(
      data[line + crushDirection],
      (end - 1), 
      '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
    );
  } else if ((((start === end) || (start + 1) === end)) && ((line + 1) !== centers.y && (line - 1) !== centers.y && line !== centers.y)) {
    data[line + crushDirection] = replaceAt(
      data[line + crushDirection], 
      (start), 
      '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
    );
    
    data[line + crushDirection] = replaceAt(
      data[line + crushDirection],
      (end),
      '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
    );
  }

  return data;
}

/**
 * Prepare data frames 
 * @param data 
 */
function prepareFrames(data: string[]): string[][]{
  let deconstructionFrames = [];
  let constructionFrames = [];

  let clonedData = renderedData.value

  // If its taking more than 100 frames, its probably somehow broken
  // Get the deconscrution frames
  for(let i = 0; i < 100; i++) {
    let newData = getMorphedFrame(clonedData);
    if( (newData as boolean)=== false) {
      break;
    }
    deconstructionFrames.push((newData as string[]).slice(0));
    clonedData = newData as string[];
  }

  // Get the constuction frames for the new data
  let squareData = squareOutData(data);
  constructionFrames.unshift(squareData.slice(0));
  for( let i = 0; i < 100; i++ ) {
    let newData = getMorphedFrame(squareData);
    if( newData === false) {
      break;
    }
    constructionFrames.unshift((newData as string[]).slice(0));
    squareData = (newData as string[]);
  }

  return deconstructionFrames.concat(constructionFrames)
}

/**
 * Mounted
 */
onMounted(function () {
  const i = index.value % list.value.length
  render(list.value[i])
})

/**
 * Watchers
 * When index change, morph the frames to desired list
 */
watch(index, function(newValue) {
  if (newValue > list.value.length) {
    morph(list.value[0])
  }
  morph(list.value[newValue % list.value.length])
})
</script>
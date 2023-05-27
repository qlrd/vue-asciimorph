<template>
  <pre v-html="rawHtml" />
</template>

<script lang="ts">
/**
 * Ascii Morph
 * @author: Tim Holman (http://tholman.com)
 * 
 * Modified for vue/typescript by @qlrd
 */
import { defineComponent, type PropType } from 'vue';

interface CanvasDimensions{
  x: number;
  y: number;
}

export default defineComponent({
  name: 'AsciiMorph',
  props: {
    list: {
      type: Object as PropType<string[][]>,
      required: true
    },
    index: {
      type: Object as  PropType<number>,
      default: 0
    },
    canvas: {
      type: Object as PropType<CanvasDimensions>,
      default: {
        x: 16,
        y: 16
      }
    },
    timeout: {
      type: Number,
      default: 20,
    }
  },
  data () {
    return {
      rawHtml: '',
      framesToAnimate: [['']],
      renderedData: [''],
      myTimeout: setTimeout(() => null, this.timeout)
    }
  },
  methods: {
    /**
     * Render to a new frame
     * @param data
     */
    render (data: string[]): void {
      const ourData = this.squareOutData(data).slice();
      this.renderSquareData(ourData);
    },
    /**
     * Morph between whatever is current, to the new frame
     * @param data
     */
    morph(data: string[]): void {
      clearTimeout(this.myTimeout);
      const frameData = this.prepareFrames(data.slice());
      this.animateFrames(frameData);
    },
    /**
     * 
     */
    animateFrames(frameData: string[][]) {
      this.framesToAnimate = frameData;
      this.animateFrame();
    },
    /**
     * 
     */
    animateFrame() {
      this.myTimeout = setTimeout(() => {
        this.renderSquareData(this.framesToAnimate[0]);
        this.framesToAnimate.shift();
        if( this.framesToAnimate.length > 0 ) {
          this.animateFrame();
        }
      }, this.timeout)
      // framesToAnimate
    },
    /**
     * 
     * @param data 
     */
    renderSquareData(data: string[]): void {
      this.rawHtml = '';
      for( var i = 0; i < data.length; i++ ) {
        this.rawHtml = this.rawHtml + data[i] + '\n';
      }
      this.renderedData = data;
    },
    /**
     * 
     * @param str 
     * @param index 
     * @param character 
     */
    replaceAt(str: string, index: number, character: string): string {
      return str.substr(0, index) + character + str.substr(index+character.length);
    },
    /**
     * 
     * @param data 
     */
    squareOutData(data: string[]): string[] {
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
          data[i] = (data[i] + this.repeat(' ', renderDimensions.x - data[i].length ));
        }
      }

      const paddings: CanvasDimensions = {
        x: Math.floor((this.canvas.x - renderDimensions.x) / 2),
        y: Math.floor((this.canvas.y - renderDimensions.y) / 2)
      }

      // Left Padding
      for( let i = 0; i < data.length; i++ ) {
        data[i] = this.repeat(' ', paddings.x) + data[i] + this.repeat(' ', paddings.x);
      }

      // Pad out the rest of everything
      for( let i = 0; i < this.canvas.y; i++ ) {
        if( i < paddings.y) {
          data.unshift(this.repeat(' ', this.canvas.x));
        } else if (i > (paddings.y + renderDimensions.y)) {
          data.push(this.repeat(' ', this.canvas.x));
        }
      }
      return data;
    },
    /**
     * 
     * @param pattern 
     * @param count 
     */
    repeat(pattern: string, count: number): string {
      if (count < 1) return '';
      let result = '';
      while (count > 1) {
        if (count & 1) result += pattern;
        count >>= 1, pattern += pattern;
      }
      return result + pattern;
    },
    /**
     * Crushes the frame data by 1 unit.
     * @param data 
     */
    getMorphedFrame(data: string[]): string[] | boolean {
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
          data = this.crushLine(data, i, firstInLine, lastInLine)
          found = true;
        }
        firstInLine = null, lastInLine = null;
      }

      if( found ) {
        return data;
      } else {
        return false;
      }
    },
    /**
     * 
     * @param data 
     * @param line 
     * @param start 
     * @param end 
     */
    crushLine(data: string[], line: number, start: number, end: number): string[] {
      let centers: CanvasDimensions = {
        x: Math.floor(this.canvas.x / 2),
        y: Math.floor(this.canvas.y / 2)
      }
      
      var crushDirection = 1;
      if( line > centers.y ) {
        crushDirection = -1;
      }

      let charA = data[line][start];
      let charB = data[line][end];

      data[line] = this.replaceAt(data[line], start, " ");
      data[line] = this.replaceAt(data[line], end, " ");

      if( !((end - 1) == (start + 1)) && !(start === end) && !((start + 1) === end)) {
        data[line + crushDirection] = this.replaceAt(
          data[line + crushDirection],
          (start + 1), 
          '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
        );
        data[line + crushDirection] = this.replaceAt(
          data[line + crushDirection],
          (end - 1), 
          '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
        );
      } else if ((((start === end) || (start + 1) === end)) && ((line + 1) !== centers.y && (line - 1) !== centers.y && line !== centers.y)) {
        data[line + crushDirection] = this.replaceAt(
          data[line + crushDirection], 
          (start), 
          '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
        );
        data[line + crushDirection] = this.replaceAt(
          data[line + crushDirection],
          (end),
          '+*/\\'.substr(Math.floor(Math.random()*'+*/\\'.length), 1)
        );
      }

      return data;
    },
    /**
     * 
     * @param data 
     */
    prepareFrames(data: string[]): string[][]{
      let deconstructionFrames = [];
      let constructionFrames = [];

      let clonedData = this.renderedData

      // If its taking more than 100 frames, its probably somehow broken
      // Get the deconscrution frames
      for(let i = 0; i < 100; i++) {
        let newData = this.getMorphedFrame(clonedData);
        if( (newData as boolean)=== false) {
          break;
        }
        deconstructionFrames.push((newData as string[]).slice(0));
        clonedData = newData as string[];
      }

      // Get the constuction frames for the new data
      let squareData = this.squareOutData(data);
      constructionFrames.unshift(squareData.slice(0));
      for( let i = 0; i < 100; i++ ) {
        let newData = this.getMorphedFrame(squareData);
        if( newData === false) {
          break;
        }
        constructionFrames.unshift((newData as string[]).slice(0));
        squareData = (newData as string[]);
      }

      return deconstructionFrames.concat(constructionFrames)
    }
  },
  mounted () {
    const i = this.index % this.list.length
    this.render(this.list[i])
  },
  watch: {
    index (newValue) {
      if (newValue > this.list.length) {
        this.morph(this.list[0])
      }
      this.morph(this.list[newValue % this.list.length])
    }
  }
})
</script>
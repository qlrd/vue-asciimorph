# vue-asciimorph

Vue AsciiMorph is a [tholman/ascii-morph](https://github.com/tholman/ascii-morph) fork. 

It provides the same functionality, but as a configurable vue component: rendering ascii art and creations into elements, allowing for them to be changed out with a morphing transition.

## Play

You can also play with the [sandbox](/sandbox/App.vue), running:

```
git clone https://www.github.com/qlrd/vue-asciimorph.git
cd vue-asciimorph
yarn install
yarn dev
```

## Usage

**Vue 3**

- Install:

```
yarn add vue-asciimorph
```

- Import and register the component;
- define some ascii art as a `list` of string's lists (`string[][]`);
- define the "size" of `canvas`; 
- define the initial `index` of `list`: the component will **render**, by default, the first list item (`index=0`);
- if your change `index` property, it will **morph** to the next list item;
- the morphing time isdefined by a `timeout`, in miliseconds;

### Example

```vue
<template>
  <AsciiMorph :list="list" :index="0" :timeout="20" :canvas="{x: 16, y: 16}"/>
  <button @click.prevent="change">Change</button>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import { AsciiMorph } from 'vue-asciimorph'

export default defineComponent({
  components: {
    AsciiMorph
  },
  methods: {
    change () {
      this.index += 1
    }
  }
  data () {
    return {
      index: 0,
      list: [
        [
          "                             /",
          "                            /",
          "                           /;",
          "                          //",
          "                         ;/",
          "                       ,//",
          "                   _,-' ;_,,",
          "                _,'-_  ;|,'",
          "            _,-'_,..--. |",
          "    ___   .'-'_)'  ) _)\\|      ___",
          "  ,'\"\"\"`'' _  )   ) _)  ''--'''_,-'",
          "-={-o-  /|    )  _)  ) ; '_,--''",
          "  \\ -' ,`.  ) .)  _)_,''|",
          "   `.\"(   `------''     /",
          "     `.\\             _,'",
          "       `-.____....-\\\\",
          "                 || \\\\",
          "                 // ||",
          "                //  ||",
          "            _-.//_ _||_,",
          "              ,'  ,-'/"
        ],
        [
          "         ____",
          "        o8%8888,",
          "      o88%8888888.",
          "     8'-    -:8888b",
          "    8'         8888",
          "   d8.-=. ,==-.:888b",
          "   >8 `~` :`~' d8888",
          "   88         ,88888",
          "   88b. `-~  ':88888",
          "   888b ~==~ .:88888",
          "   88888o--:':::8888",
          "   `88888| :::' 8888b",
          "   8888^^'       8888b",
          "  d888           ,%888b.",
          " d88%            %%%8--'-.",
          "/88:.__ ,       _%-' ---  -",
          "    '''::===..-'   =  --.  `",
        ]
      ]
    }
  }
})
```


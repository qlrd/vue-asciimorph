<!-- sandbox/App.vue -->
<template>
  <h1>{{ title }}</h1>
  <AsciiMorph :list="list" :index="index" :timeout="20" :canvas="{x: 16, y: 16}"/>
  <button @click.prevent="change">Change</button>
</template>


<script lang="ts">
import { defineComponent } from 'vue'
import AsciiMorph from '../src/AsciiMorph.vue'

export default defineComponent({
  components: {
    AsciiMorph
  },
  methods: {
    change () {
      this.index += 1
    }
  },
  data () {
    return {
      title: 'vue-asciimorph',
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
</script>
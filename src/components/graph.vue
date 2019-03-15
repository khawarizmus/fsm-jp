<template>
  <v-card height="100%">
    <v-card-title>Final State Machine Visualisation</v-card-title>
    <v-card-actions>
      <v-layout>
        <v-flex sx3>
          <v-text-field label="English" v-model="input" :error="error" outline></v-text-field>
          <v-text-field
            label="Japanease transliteration"
            v-model="transliteration"
            disabled
            :success="success"
            outline
          ></v-text-field>
        </v-flex>
        <v-flex sx9>
          <div class="d-flex" style="justify-content: center;" ref="state"></div>
        </v-flex>
      </v-layout>
    </v-card-actions>
    <v-card-text class="d-flex" style="justify-content: center;" ref="viz"></v-card-text>
  </v-card>
</template>

<script>
import Viz from 'viz.js'
import { Module, render } from 'viz.js/full.render.js'
import StateMachine from 'javascript-state-machine'
import visualize from 'javascript-state-machine/lib/visualize'
import _ from 'lodash'

export default {
  data() {
    return {
      viz: undefined,
      fsm: undefined,
      states: undefined,
      transitions: undefined,
      error: false,
      success: false,
      input: '',
      transliteration: '',
      enWords: [
        'computer',
        'new york',
        'bill gates',
        'golf ball',
        'television',
        'radio',
        'twin tower',
        'video game',
        'restaurant',
        'story',
        'elevator',
        'ice cream',
      ],
    }
  },
  created() {
    _.debounce(this.debouncedInput, 500)
  },
  mounted() {
    const that = this // capturing the scope
    this.fsm = new StateMachine({
      init: 'initial', // initial state
      transitions: [
        // Computer
        // Konpyūtā
        { name: 'com', from: 'initial', to: 'kon' },
        { name: 'pu', from: ['initial', 'kon'], to: 'pyū' },
        { name: 'ter', from: ['initial', 'pyū'], to: 'tā' },
        // Televisison
        // Terebijion
        { name: 'tele', from: ['initial'], to: 'tere' },
        { name: 'vi', from: ['initial', 'tere'], to: 'bi' },
        { name: 'sion', from: ['initial', 'bi'], to: 'jon' },
        // Radio
        // Rajio
        { name: 'ra', from: ['initial'], to: 'ra' },
        { name: 'dio', from: ['initial', 'ra'], to: 'jio' },
        // Restaurant
        // Resutoran
        { name: 'res', from: ['initial'], to: 'resu' },
        { name: 'tau', from: ['initial', 'resu'], to: 'to' },
        { name: 'rant', from: ['initial', 'to'], to: 'ran' },
        // Story
        // Sutōrī
        { name: 'sto', from: ['initial'], to: 'sutō' },
        { name: 'ry', from: ['initial', 'sutō'], to: 'rī' },
        // Elevator
        // Erebētā
        { name: 'ele', from: ['initial'], to: 'ere' },
        { name: 'va', from: ['initial', 'ere'], to: 'bē' },
        { name: 'tor', from: ['initial', 'bē'], to: 'tā' },
        // New york
        // Nyūyōku
        // Nyū yōku
        { name: 'new', from: ['initial'], to: 'Nyū' },
        { name: 'york', from: ['initial', 'Nyū'], to: 'yōku' },
        // Bill gates
        // Birugeitsu
        // Biru geitsu
        { name: 'bill', from: ['initial'], to: 'biru' },
        { name: 'ga', from: ['initial', 'biru'], to: 'gei' },
        { name: 'tes', from: ['initial', 'gei'], to: 'tsu' },
        // Golf ball
        // Gorufubōru
        // Gorufu bōru
        { name: 'golf', from: ['initial'], to: 'gorufu' },
        { name: 'ball', from: ['initial', 'gorufu'], to: 'bōru' },
        // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'tw', from: ['initial'], to: 'tsu' },
        { name: 'in', from: ['initial', 'tsu'], to: 'in' },
        { name: 'to', from: ['initial', 'in'], to: 'ta' },
        { name: 'wer', from: ['initial', 'ta'], to: 'wā' },
        // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'vi', from: ['initial'], to: 'bi' },
        { name: 'deo', from: ['initial', 'bi'], to: 'deo' },
        { name: 'ga', from: ['initial', 'deo'], to: 'gē' },
        { name: 'me', from: ['initial', 'gē'], to: 'mu' },
        // Ice cream
        // Aisukurīmu
        // Aisu kurīmu
        { name: 'ice', from: ['initial'], to: 'aisu' },
        { name: 'cream', from: ['initial', 'aisu'], to: 'kurīmu' },
      ],
      methods: {},
    })
    this.states = this.fsm.allStates() // all states (transliterations)
    this.transitions = this.fsm.transitions() // alll transitions (syllables)

    const graph = visualize(this.fsm) // getting a .dot file format
    // for more visti: https://github.com/jakesgordon/javascript-state-machine/blob/master/docs/visualization.md

    // creating an svg graph
    // for more visit: https://github.com/mdaines/viz.js/wiki/Usage#using-a-script-tag
    this.viz = new Viz({ Module, render })
    this.viz
      .renderSVGElement(graph)
      .then((result) => {
        // getting the element from the dom
        that.$refs.viz.appendChild(result)
      })
      .catch((error) => {
        // Create a new Viz instance (@see Caveats page for more info)
        that.viz = new Viz({ Module, render })

        // Possibly display the error
        console.error(error)
      })
  },
  methods: {
    debouncedInput() {
      return this.input
    },
  },
  watch: {
    input() {
      this.error = false
      this.success = false
      this.transliteration = ''
      const that = this
      // trimed, removed spaces and lowercased
      const value = _.replace(
        _.lowerCase(_.trim(this.debouncedInput())),
        ' ',
        ''
      ) // Compter Seince ==> computerseince
      // clean the dom from the previos graph
      while (this.$refs.state.hasChildNodes()) {
        this.$refs.state.removeChild(this.$refs.state.lastChild)
      }
      let fsm
      let graph
      if (value === 'com') {
        // one syllabe
        this.success = true
        this.transliteration = 'Kon'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Comp
            // Kon
            { name: 'com', from: 'initial', to: 'kon' },
          ],
        })
        // create the graph
        graph = visualize(fsm, { name: 'fsm', orientation: 'horizontal' })
        // draw with svg
        this.viz
          .renderSVGElement(graph)
          .then((result) => {
            // getting the element from the dom
            that.$refs.state.appendChild(result)
          })
          .catch((error) => {
            // Create a new Viz instance (@see Caveats page for more info)
            that.viz = new Viz({ Module, render })

            // Possibly display the error
            console.error(error)
          })
      } else if (value === 'pu') {
        // normal english word
        this.success = true
        this.transliteration = 'pyū'
        // state machine for computer
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Computer
            // Konpyūtā
            { name: 'pu', from: 'initial', to: 'pyū' },
          ],
        })
        // generate the graph
        graph = visualize(fsm, { name: 'fsm', orientation: 'horizontal' })
        this.viz
          .renderSVGElement(graph)
          .then((result) => {
            // getting the element from the dom
            that.$refs.state.appendChild(result)
          })
          .catch((error) => {
            // Create a new Viz instance (@see Caveats page for more info)
            that.viz = new Viz({ Module, render })

            // Possibly display the error
            console.error(error)
          })
      } else if (value === 'compu') {
        // normal english word
        this.success = true
        this.transliteration = 'Konpyū'
        // state machine for computer
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Computer
            // Konpyūtā
            { name: 'com', from: 'initial', to: 'kon' },
            { name: 'pu', from: 'kon', to: 'pyū' },
          ],
        })
        // generate the graph
        graph = visualize(fsm, { name: 'fsm', orientation: 'horizontal' })
        this.viz
          .renderSVGElement(graph)
          .then((result) => {
            // getting the element from the dom
            that.$refs.state.appendChild(result)
          })
          .catch((error) => {
            // Create a new Viz instance (@see Caveats page for more info)
            that.viz = new Viz({ Module, render })

            // Possibly display the error
            console.error(error)
          })
      } else if (value === 'computer') {
        // normal english word
        this.success = true
        this.transliteration = 'Konpyūtā'
        // state machine for computer
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Computer
            // Konpyūtā
            { name: 'com', from: 'initial', to: 'kon' },
            { name: 'pu', from: 'kon', to: 'pyū' },
            { name: 'ter', from: 'pyū', to: 'tā' },
          ],
        })
        // generate the graph
        graph = visualize(fsm, { name: 'fsm', orientation: 'horizontal' })
        this.viz
          .renderSVGElement(graph)
          .then((result) => {
            // getting the element from the dom
            that.$refs.state.appendChild(result)
          })
          .catch((error) => {
            // Create a new Viz instance (@see Caveats page for more info)
            that.viz = new Viz({ Module, render })

            // Possibly display the error
            console.error(error)
          })
      } else if (value === 'new') {
        // two words will be concatenated
        this.success = true
        this.transliteration = 'Nyū'
        // state machine for new york

        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // new york
            // Nyūyōku
            { name: 'new', from: ['initial'], to: 'Nyū' },
          ],
        })
        // generate the graph
        graph = visualize(fsm, { name: 'fsm', orientation: 'horizontal' })
        // draw in svg
        this.viz
          .renderSVGElement(graph)
          .then((result) => {
            // getting the element from the dom
            that.$refs.state.appendChild(result)
          })
          .catch((error) => {
            // Create a new Viz instance (@see Caveats page for more info)
            that.viz = new Viz({ Module, render })

            // Possibly display the error
            console.error(error)
          })
      } else if (value === 'york') {
        // two words will be concatenated
        this.success = true
        this.transliteration = 'yōku'
        // state machine for new york

        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // new york
            // Nyūyōku
            { name: 'york', from: 'initial', to: 'yōku' },
          ],
        })
        // generate the graph
        graph = visualize(fsm, { name: 'fsm', orientation: 'horizontal' })
        // draw in svg
        this.viz
          .renderSVGElement(graph)
          .then((result) => {
            // getting the element from the dom
            that.$refs.state.appendChild(result)
          })
          .catch((error) => {
            // Create a new Viz instance (@see Caveats page for more info)
            that.viz = new Viz({ Module, render })

            // Possibly display the error
            console.error(error)
          })
      } else if (value === 'newyork') {
        // two words will be concatenated
        this.success = true
        this.transliteration = 'Nyūyōku'
        // state machine for new york

        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // new york
            // Nyūyōku
            { name: 'new', from: ['initial'], to: 'Nyū' },
            { name: 'york', from: 'Nyū', to: 'yōku' },
          ],
        })
        // generate the graph
        graph = visualize(fsm, { name: 'fsm', orientation: 'horizontal' })
        // draw in svg
        this.viz
          .renderSVGElement(graph)
          .then((result) => {
            // getting the element from the dom
            that.$refs.state.appendChild(result)
          })
          .catch((error) => {
            // Create a new Viz instance (@see Caveats page for more info)
            that.viz = new Viz({ Module, render })

            // Possibly display the error
            console.error(error)
          })
      } else if (value === 'bill') {
      } else if (value === 'ga') {
      } else if (value === 'tes') {
      } else if (value === 'billga') {
      } else if (value === 'billgates') {
      } else if (value === 'golf') {
      } else if (value === 'ball') {
      } else if (value === 'golfball') {
      } else if (value === 'tele') {
      } else if (value === 'vi') {
      } else if (value === 'sion') {
      } else if (value === 'televi') {
      } else if (value === 'television') {
      } else if (value === 'ra') {
      } else if (value === 'dio') {
      } else if (value === 'radio') {
      } else if (value === 'tw') {
      } else if (value === 'in') {
      } else if (value === 'to') {
      } else if (value === 'wer') {
      } else if (value === 'twin') {
      } else if (value === 'twinto') {
      } else if (value === 'twintower') {
      } else if (value === 'vi') {
      } else if (value === 'deo') {
      } else if (value === 'ga') {
      } else if (value === 'me') {
      } else if (value === 'video') {
      } else if (value === 'videoga') {
      } else if (value === 'videogame') {
      } else if (value === 'res') {
      } else if (value === 'tau') {
      } else if (value === 'rant') {
      } else if (value === 'restau') {
      } else if (value === 'restaurant') {
      } else if (value === 'sto') {
      } else if (value === 'ry') {
      } else if (value === 'story') {
      } else if (value === 'ele') {
      } else if (value === 'va') {
      } else if (value === 'tor') {
      } else if (value === 'eleva') {
      } else if (value === 'elevator') {
      } else if (value === 'ice') {
      } else if (value === 'cream') {
      } else if (value === 'icecream') {
      }
    },
  },
}
</script>
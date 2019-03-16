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
        this.success = true
        this.transliteration = 'Biru'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Bill gates
        // Birugeitsu
        // Biru geitsu
        { name: 'bill', from: ['initial'], to: 'biru' },
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
      } else if (value === 'ga') {
        this.success = true
        this.transliteration = 'gei'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Bill gates
        // Birugeitsu
        // Biru geitsu
        { name: 'ga', from: 'initial', to: 'gei' },
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
      } else if (value === 'tes') {
        this.success = true
        this.transliteration = 'tsu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Bill gates
        // Birugeitsu
        // Biru geitsu
        { name: 'tes', from: 'initial', to: 'tsu' },
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
      } else if (value === 'billga') {
        this.success = true
        this.transliteration = 'Birugei'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Bill gates
        // Birugeitsu
        // Biru geitsu
        { name: 'bill', from: ['initial'], to: 'biru' },
        { name: 'ga', from: 'biru', to: 'gei' },
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
      } else if (value === 'billgates') {
        this.success = true
        this.transliteration = 'Birugeitsu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
            // Bill gates
        // Birugeitsu
        // Biru geitsu
        { name: 'bill', from: ['initial'], to: 'biru' },
        { name: 'ga', from: ['biru'], to: 'gei' },
        { name: 'tes', from: [ 'gei'], to: 'tsu' },
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
      } else if (value === 'golf') {
         this.success = true
        this.transliteration = 'Gorufu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Golf ball
        // Gorufubōru
        // Gorufu bōru
        { name: 'golf', from: ['initial'], to: 'gorufu' },
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
      } else if (value === 'ball') {
        this.success = true
        this.transliteration = 'bōru'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Golf ball
        // Gorufubōru
        // Gorufu bōru
        { name: 'ball', from: ['initial'], to: 'bōru' },
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
      } else if (value === 'golfball') {
        this.success = true
        this.transliteration = 'Gorufubōru'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Golf ball
        // Gorufubōru
        // Gorufu bōru
        { name: 'golf', from: ['initial'], to: 'gorufu' },
        { name: 'ball', from: ['initial', 'gorufu'], to: 'bōru' },
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
      } else if (value === 'tele') {
         this.success = true
        this.transliteration = 'Tere'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Televisison
        // Terebijion
        { name: 'tele', from: ['initial'], to: 'tere' },
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
      } else if (value === 'vi') {
         this.success = true
        this.transliteration = 'bi'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Televisison
        // Terebijion
        { name: 'vi', from: 'initial', to: 'bi' },
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
      } else if (value === 'sion') {
         this.success = true
        this.transliteration = 'jion'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Televisison
        // Terebijion
        { name: 'sion', from: 'initial', to: 'jion' },
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
      } else if (value === 'televi') {
         this.success = true
        this.transliteration = 'Terebi'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Televisison
        // Terebijion
        { name: 'tele', from: ['initial'], to: 'tere' },
        { name: 'vi', from: 'tere', to: 'bi' },
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
      } else if (value === 'television') {
        this.success = true
        this.transliteration = 'Terebijion'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
           // Televisison
        // Terebijion
        { name: 'tele', from: ['initial'], to: 'tere' },
        { name: 'vi', from: ['initial', 'tere'], to: 'bi' },
        { name: 'sion', from: ['initial', 'bi'], to: 'jion' },
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
      } else if (value === 'ra') {
        this.success = true
        this.transliteration = 'Ra'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
          // Radio
        // Rajio
        { name: 'ra', from: ['initial'], to: 'ra' },
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
      } else if (value === 'dio') {
        this.success = true
        this.transliteration = 'Rajio'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
          // Radio
        // Rajio
        { name: 'dio', from: ['initial'], to: 'jio' },
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
      } else if (value === 'radio') {
        this.success = true
        this.transliteration = 'Rajio'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
          // Radio
        // Rajio
        { name: 'ra', from: ['initial'], to: 'ra' },
        { name: 'dio', from: ['initial', 'ra'], to: 'jio' },
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
      } else if (value === 'tw') {
        this.success = true
        this.transliteration = 'Tsuintawā'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'tw', from: ['initial'], to: 'tsu' },
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
      } else if (value === 'in') {
        this.success = true
        this.transliteration = 'Tsuintawā'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'in', from: ['initial'], to: 'in' },
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
      } else if (value === 'to') {
        this.success = true
        this.transliteration = 'Ta'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'to', from: ['initial', 'in'], to: 'ta' },
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
      } else if (value === 'wer') {
        this.success = true
        this.transliteration = 'Tsuintawā'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'wer', from: ['initial'], to: 'wā' },
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
      } else if (value === 'twin') {
        this.success = true
        this.transliteration = 'Tsuin'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'tw', from: ['initial'], to: 'tsu' },
        { name: 'in', from: ['initial', 'tsu'], to: 'in' },
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
      } else if (value === 'twinto') {
        this.success = true
        this.transliteration = 'Tsuinta'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'tw', from: ['initial'], to: 'tsu' },
        { name: 'in', from: ['initial', 'tsu'], to: 'in' },
        { name: 'to', from: ['initial', 'in'], to: 'ta' },
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
      } else if (value === 'twintower') {
        this.success = true
        this.transliteration = 'Tsuintawā'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Twin tower
        // Tsuintawā
        // Tsuin tawā
        { name: 'tw', from: ['initial'], to: 'tsu' },
        { name: 'in', from: ['initial', 'tsu'], to: 'in' },
        { name: 'to', from: ['initial', 'in'], to: 'ta' },
        { name: 'wer', from: ['initial', 'ta'], to: 'wā' },
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
      } else if (value === 'vi') {
         this.success = true
        this.transliteration = 'bi'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'vi', from: ['initial'], to: 'bi' },
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
      } else if (value === 'deo') {
         this.success = true
        this.transliteration = 'deo'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'deo', from: ['initial'], to: 'deo' },
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
      } else if (value === 'ga') {
         this.success = true
        this.transliteration = 'gē'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'ga', from: ['initial'], to: 'gē' },
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
      } else if (value === 'me') {
         this.success = true
        this.transliteration = 'mu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'me', from: ['initial'], to: 'mu' },
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
      } else if (value === 'video') {
         this.success = true
        this.transliteration = 'Bideo'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'vi', from: ['initial'], to: 'bi' },
        { name: 'deo', from: ['initial', 'bi'], to: 'deo' },
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
      } else if (value === 'videoga') {
         this.success = true
        this.transliteration = 'Bideogē'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'vi', from: ['initial'], to: 'bi' },
        { name: 'deo', from: ['initial', 'bi'], to: 'deo' },
        { name: 'ga', from: ['initial', 'deo'], to: 'gē' },
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
      } else if (value === 'videogame') {
         this.success = true
        this.transliteration = 'Bideogemu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Video game
        // Bideogēmu
        // Bideo gēmu
        { name: 'vi', from: ['initial'], to: 'bi' },
        { name: 'deo', from: ['initial', 'bi'], to: 'deo' },
        { name: 'ga', from: ['initial', 'deo'], to: 'gē' },
        { name: 'me', from: ['initial', 'gē'], to: 'mu' },
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
      } else if (value === 'res') {
        this.success = true
        this.transliteration = 'Resu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Restaurant
        // Resutoran
        { name: 'res', from: ['initial'], to: 'resu' },
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
      } else if (value === 'tau') {
        this.success = true
        this.transliteration = 'to'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Restaurant
        // Resutoran
        { name: 'tau', from: ['initial'], to: 'to' },
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
      } else if (value === 'rant') {
        this.success = true
        this.transliteration = 'ran'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Restaurant
        // Resutoran
        { name: 'rant', from: ['initial'], to: 'ran' },
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
      } else if (value === 'restau') {
        this.success = true
        this.transliteration = 'Resuto'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Restaurant
        // Resutoran
        { name: 'res', from: ['initial'], to: 'resu' },
        { name: 'tau', from: ['initial', 'resu'], to: 'to' },
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
      } else if (value === 'restaurant') {
        this.success = true
        this.transliteration = 'Resutoran'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Restaurant
        // Resutoran
        { name: 'res', from: ['initial'], to: 'resu' },
        { name: 'tau', from: ['initial', 'resu'], to: 'to' },
        { name: 'rant', from: ['initial', 'to'], to: 'ran' },
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
      } else if (value === 'sto') {
        this.success = true
        this.transliteration = 'Sutō'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Story
        // Sutōrī
        { name: 'sto', from: ['initial'], to: 'sutō' },
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
      } else if (value === 'ry') {
        this.success = true
        this.transliteration = 'rī'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Story
        // Sutōrī
        { name: 'ry', from: ['initial'], to: 'rī' },
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
      } else if (value === 'story') {
        this.success = true
        this.transliteration = 'Sutōrī'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Story
        // Sutōrī
        { name: 'sto', from: ['initial'], to: 'sutō' },
        { name: 'ry', from: ['initial', 'sutō'], to: 'rī' },
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
      } else if (value === 'ele') {
        this.success = true
        this.transliteration = 'Ere'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Elevator
        // Erebētā
        { name: 'ele', from: ['initial'], to: 'ere' },
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
      } else if (value === 'va') {
        this.success = true
        this.transliteration = 'bē'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Elevator
        // Erebētā
        { name: 'va', from: ['initial'], to: 'bē' },
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
      } else if (value === 'tor') {
        this.success = true
        this.transliteration = 'tā'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Elevator
        // Erebētā
        { name: 'tor', from: ['initial'], to: 'tā' },
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
      } else if (value === 'eleva') {
        this.success = true
        this.transliteration = 'Erebē'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Elevator
        // Erebētā
        { name: 'ele', from: ['initial'], to: 'ere' },
        { name: 'va', from: ['initial', 'ere'], to: 'bē' },
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
      } else if (value === 'elevator') {
        this.success = true
        this.transliteration = 'Erebētā'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
         // Elevator
        // Erebētā
        { name: 'ele', from: ['initial'], to: 'ere' },
        { name: 'va', from: ['initial', 'ere'], to: 'bē' },
        { name: 'tor', from: ['initial', 'bē'], to: 'tā' },
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
      } else if (value === 'ice') {
        this.success = true
        this.transliteration = 'Aisu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
        // Ice cream
        // Aisukurīmu
        // Aisu kurīmu
        { name: 'ice', from: ['initial'], to: 'aisu' },
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
      } else if (value === 'cream') {
        this.success = true
        this.transliteration = 'kurīmu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
        // Ice cream
        // Aisukurīmu
        // Aisu kurīmu
        { name: 'cream', from: ['initial'], to: 'kurīmu' },
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
      } else if (value === 'icecream') {
        this.success = true
        this.transliteration = 'Aisukurīmu'
        fsm = new StateMachine({
          init: 'initial', // initial state
          transitions: [
        // Ice cream
        // Aisukurīmu
        // Aisu kurīmu
        { name: 'ice', from: ['initial'], to: 'aisu' },
        { name: 'cream', from: ['initial', 'aisu'], to: 'kurīmu' },
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
      }
    },
  },
}
</script>
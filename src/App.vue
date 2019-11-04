<template>
  <div id="app">
    <dropdown @isSelected="updateSelected" ></dropdown>
    <br>
    <br>
    <pure-vue-chart
      :show-y-axis="true"
      :show-x-axis="true"
      :points="dataPoints"
      :width="chartWidth"
      :height="chartHeight"
      :show-values="true"
      :use-month-labels="true"
    />
  </div>
</template>

<script>
import PureVueChart from './components/PureVueChart.vue';
import DPT from './json/dpt.json'
import Vuex from 'vuex'
import Vue from 'vue'
import dropdown from './dropdown'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    values: [],
    avgvalues: [],
    selected: ''
  },
  mutations: {
    addDPT (state, value) {
      state.values.push(value)
    },
    setDPT (state, value) {
      state.values = value
    },
    addAvgDPT (state, value) {
      state.avgvalues.push(value)
    },
    setAvgDPT (state, value) {
      state.avgvalues = value
    },
    addDataObjects (state, value) {
      state.dataobjectvalues.push(value)
    },
    setDataObjects (state, value) {
      state.dataobjectvalues = value
    },
    setSelected(state, selected) {
      state.selected = selected
    }
  }
})

export default {
  name: 'App',
  components: {
    PureVueChart,
    dropdown
  },
  mounted() {
    this.dptcheck()
    this.dataPoints = this.avgdpt()
  },
  updated() {
  },
  data() {
    return {
      dropdown: dropdown,
      dataPoints: [],
      // Keep this property for future pure-vue-chart development
      dataPointObjects: [{label: "labelName", value: "valueName"}],
      chartWidth: 1800,
      chartHeight: 600,
      dpt: DPT,
      selected: 'test',
      ensembleSpot: 0
  }},
  watch : {
  },
  computed: {
  },
  methods: {
    updateSelected(selected) {
      store.commit('setSelected', selected)
      this.selectedCheck()
    },
    avgdpt(value) {
      store.commit('setAvgDPT', [])
      store.state.values.forEach(function(dpt){
        dpt = (dpt / 52)
        var rounded = Math.round( dpt * 10 ) / 10
        rounded.toFixed(1)
        store.commit('addAvgDPT', rounded)
      })
      return store.state.avgvalues
    },
    dptcheck() {
      this.ensembleCheck()
      let tempDpt = {}
      let tempLocation = []
      store.commit('setDPT', [])
      // this.dpt.Data[{{Ensemble/location}}].Ensembles is the location to laod data from
      tempDpt = this.dpt.Data[this.ensembleSpot].Ensembles
      for(var counter = 0; counter < 40; counter++){
        let tempDptValue = 0
        Object.keys(tempDpt).forEach(function(k){
          tempDptValue = tempDptValue + tempDpt[k].Data[counter].Value
          if(k == 51) {
            tempLocation.push(tempDpt[k].LocationCode)
            store.commit('addDPT', tempDptValue)
          }
        })
      }
    },
    selectedCheck() {
      this.dptcheck()
      this.dataPoints = this.avgdpt()
    },
    ensembleCheck() {
      let countEnsembles = this.dpt.Data.length
      for (let cnt = 0; cnt < countEnsembles; cnt++) {
        let tempDpt = this.dpt.Data[cnt].Ensembles
        if (tempDpt[1].LocationCode === store.state.selected) {
          this.ensembleSpot = cnt
        }
      }
    },
    // pure-vue-chart component buggy, keep method for future
    convertDPTObject() {
      var avgdptvalues = this.avgdpt()
      var tempObject = {}
      var tempArray = []
      store.commit('setDataObjects', [])
      avgdptvalues.forEach((value, i) => {
        if (i % 4 === 2) {
          tempObject = {label: '06:00', value: value}
        }
        if (i % 4 === 3) {
          tempObject = {label: '12:00', value: value}
        }
        if (i % 4 === 0) {
          tempObject = {label: '18:00', value: value}
        }
        if (i % 4 === 1) {
          tempObject = {label: '00:00', value: value}
        }
        store.commit('addDataObjects', tempObject)
      })
      return store.state.dataobjectvalues
    }
  },
};
</script>

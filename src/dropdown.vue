<template>
  <div id="dropdown">
    <select v-model="selected" @change="submitSelected(selected)">
      <option v-for="option in locationList.locationList" v-bind:key="option.value">
        {{ option.text }}
      </option>
    </select>
    <!--<button @click="submitSelected(selected)">Confirm selection</button> -->
  </div>
</template>

<script>
import locations from './json/locations.json'
import DPT from './json/dpt.json'
import Vuex from 'vuex'
import Vue from 'vue'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    locationList: [],
    selected: '',
    selectedCode: ''
  },
  mutations: {
    addLocations(state, locationList) {
      state.locationList = locationList
    },
    setSelected(state, selected) {
      state.selected = selected
    }
  }
})

export default {
  name: 'Dropdown',
  components: {

  },
  mounted() {
    this.listOfLocations()
    this.$set(this.locationList, 'locationList', store.state.locationList)
  },
  data() {
    return {
      locations: locations,
      dpt: DPT,
      selected: '',
      locationList: []
  }},
  computed: {
  },
  methods: {
    convertLocationNameToCode(selected) {
      let tempLocations = store.state.locationList
      tempLocations.forEach(textValueObject => {
        if (textValueObject.text === selected) {
          this.selectedCode = textValueObject.value
          }
      })
    },
    listOfLocations() {
      let tempLocationList = []
      let tempLocations = this.locations.Locations
      let tempEnsembles = []
      let countEnsembles = this.dpt.Data.length
      for (let cnt = 0; cnt < countEnsembles; cnt++) {
          let tempDpt = this.dpt.Data[cnt].Ensembles
          tempEnsembles.push(tempDpt)
      }
     
      Object.keys(tempLocations).forEach(function(k){
            for(let x = 0; x < tempEnsembles.length; x++){
                if (tempEnsembles[x][1].LocationCode === tempLocations[k].Code) {
                  tempLocationList.push({"text": tempLocations[k].Name, "value": tempLocations[k].Code})
              }
            }
      })
      store.commit("addLocations", tempLocationList)
   
    },
    setSelected(selected) {
      store.commit("setSelected", selected)
    },
    submitSelected(selected) {
      console.log(selected)
      this.convertLocationNameToCode(selected)
      this.$emit("isSelected", this.selectedCode)
    }
  },
};
</script>

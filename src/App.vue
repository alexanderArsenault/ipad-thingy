<template>
  <v-app>
    <!-- Navigation Bar -->
    <v-navigation-drawer
        v-model="drawer"
        width="700"
        absolute
        temporary
      >
        <v-container>
          <EventTimeline/>
        </v-container>
    </v-navigation-drawer>

    <!-- App bar -->
    <app-bar @openTimeline="drawer = true" />
    <!-- App Content -->
    <v-sheet color="#f5f5f5">
      <home></home>
    </v-sheet>
    <!-- Footer -->
  </v-app>
</template>
<script>
import AppBar from './components/AppBar';
import Home from './components/Home';
import EventTimeline from './components/EventTimeline';
import temperatures from './assets/shipment_data.js'


export default {
  name: 'App',
  components: {
    Home,
    AppBar,
    EventTimeline,
  },
  data: () => ({
    drawer: false,
    bands: [
      { 
        k: "< 2°C",
        min:-1000,
        max: 2,
        hours: 1,
        v: 100
      },
      { 
        k: "2-8°C",
        min: 2,
        max: 8,
        hours: 1500,
        v: 100
      },
      { 
        k: "8-15°C",
        min: 8,
        max: 15,
        hours: 500,
        v: 100
      },
      { 
        k: "> 15°C",
        min: 15,
        max: 1000,
        hours: 1,
        v: 100
      },
    ]
  }),
  computed: {
    stabilityBudget(){
      return this.bands.reduce( (acc, curr) => {
        let formatentry = {
          k: curr.k,
          v: Math.max((100 - parseInt(this.getMillisecondsSpentBetweenTemperatures(curr.min, curr.max) / (curr.hours * 60 * 60 * 1000) * 100)), 0)
        }
        return [...acc, formatentry]
      }, [])
    },
    formattedTemperatureDate(){
      return temperatures.temps.reduce( (acc, curr) => {
        return [
          ...acc,
          {
          x: new Date(curr.x),
          y: curr.y
        }]
      }, [])
    },
    transformedTemperatureSeries(){
      return this.transformations.reduce( (acc, curr) => {
        return this.addTemperatureBetweenIndexs(
          curr.start,
          curr.end,
          curr.amount,
          acc
        )
      }, this.formattedTemperatureDate)
    }
  },
  methods: {
    addTemperatureBetweenIndexs(start, end, amount, array){
      for (let i = start; i < end; i++){
        let halfway = parseInt((start + end) / 2)
        let taperedamount = amount / Math.abs(i - halfway)


        if (taperedamount === Infinity) taperedamount = 1
        array[i].y = array[i].y + taperedamount
      }
      return array
    },
    getMillisecondsSpentBetweenTemperatures(min, max){
      let lasttime = null
      
      return this.formattedTemperatureDate.reduce((acc, curr) => {
        if (curr.y > min && curr.y < max) {
          if (lasttime === null) {
            lasttime = curr.x
            return acc
          } else {
            let timediff = new Date(curr.x).valueOf() - new Date(lasttime).valueOf()
            lasttime = curr.x
            return acc + timediff
          }
        } else {
          lasttime = null
          return acc
        }
      }, 0)
    }
  }
};
</script>
<style lang="scss" scoped>
.app-bar-header {
  padding: 0 30px;
}

.psb-container {
  display: flex;
  align-items: center;
  justify-content: center;
}

.psb-container {
  width: 400px;
  height: 400px;
  margin-top: 30px;
}

.side-tray {
  height: 100%;
  overflow-y: scroll;
}

.tablet-container {
  max-width: 100vw;
  overflow-x: hidden;
}
</style>
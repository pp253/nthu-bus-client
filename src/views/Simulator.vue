<template>
  <v-container fluid>
    <v-card class="mb-3">
      <v-card-text>
        <v-layout wrap>
          <v-flex xs12>
            <v-btn @click="prevStep()" class="mr-3">PREV</v-btn>
            <v-btn @click="nextStep()" class="mr-3">NEXT</v-btn>
            <span class="headline"> {{ nowDateString }} ({{ offset }}) </span>
          </v-flex>
          <v-flex xs12>
            <v-slider v-model="offset" :max="maxOffset" :min="0"></v-slider>
          </v-flex>

          <v-flex
            xs3
            class="text-left"
            v-for="(routing, key) in routes"
            :key="key"
          >
            <h3>{{ translation[key] }}</h3>
            <v-layout wrap v-for="pointId in routing" :key="pointId">
              <v-flex xs5>
                {{ toWaitingTime(prediction[key][pointId]) }}
              </v-flex>
              <v-flex xs7> {{ points[pointId].Name }} ({{ pointId }}) </v-flex>
            </v-layout>
          </v-flex>
        </v-layout>
      </v-card-text>
    </v-card>
    <v-card>
      <v-card-text>
        <v-layout wrap>
          <v-flex>
            <v-simple-table>
              <template v-slot:default>
                <thead>
                  <tr>
                    <th class="text-left">id</th>
                    <th class="text-left">CarNo</th>
                    <th class="text-left">PX</th>
                    <th class="text-left">PY</th>
                    <th class="text-left">PointId</th>
                    <th class="text-left">LastPointId</th>
                    <th class="text-left">GPSTime</th>
                    <th class="text-left">RouteId</th>
                    <th class="text-left">ScheduleId</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, key) in position" :key="key">
                    <td>
                      {{ key }}
                    </td>
                    <td>{{ cars[key].CarNo }}</td>
                    <td>{{ item.PX }}</td>
                    <td>{{ item.PY }}</td>
                    <td>
                      {{ item.PointId }},
                      {{ item.PointId !== 0 ? points[item.PointId].Name : "" }}
                    </td>
                    <td>{{ item.LastPointId }}</td>
                    <td>{{ item.GPSTime }}</td>
                    <td>{{ item.RouteId }}</td>
                    <td>
                      {{ item.ScheduleId }},
                      {{
                        item.ScheduleId
                          ? schedules[item.ScheduleId].DepartureTime
                          : ""
                      }}
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
          </v-flex>
        </v-layout>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import simulation from "./simulation.json";
export default {
  data() {
    return {
      predictions: simulation.predictions,
      positions: simulation.positions,
      cars: simulation.cars,
      routes: simulation.routes,
      points: simulation.points,
      schedules: simulation.schedules,

      translation: {
        "nthu-red-forward": "紅線-上山",
        "nthu-red-backward": "紅線-下山",
        "nthu-green-forward": "綠線-上山",
        "nthu-green-backward": "綠線-下山"
      },

      offset: 0,
      startTime: null,
      endTime: null,
      timeStep: 30 * 1000
    };
  },
  computed: {
    now() {
      if (!this.endTime || !this.startTime) {
        return 0;
      }
      return this.startTime.getTime() + this.timeStep * this.offset;
    },
    nowDay() {
      let t = new Date(this.startTime).setHours(0, 0, 0, 0);
      return t;
    },
    nowTime() {
      return this.now - this.nowDay;
    },
    nowDateString() {
      return new Date(this.now).toLocaleTimeString();
    },
    prediction() {
      return this.predictions[this.offset];
    },
    position() {
      return this.positions[this.offset];
    },
    maxOffset() {
      if (!this.endTime || !this.startTime) {
        return 0;
      }
      return (
        (this.endTime.getTime() - this.startTime.getTime()) / this.timeStep - 1
      );
    }
  },
  methods: {
    nextStep() {
      if (this.offset >= this.maxOffset) {
        return;
      }
      this.offset += 1;
    },
    prevStep() {
      if (this.offset <= 0) {
        return;
      }
      this.offset -= 1;
    },
    toWaitingTime(time) {
      let delta = new Date(time).getTime() - this.nowTime;
      let minString = (delta / (60 * 1000)).toFixed(1);
      if (minString < -500) {
        // return `誤點 ${minString} 分鐘`;
        return `末班已過`;
      } else if (delta < 0) {
        return `誤點 ${minString} 分鐘`;
      } else {
        return `${minString} 分鐘`;
      }
    }
  },
  mounted() {
    this.startTime = new Date(simulation.startTime);
    this.endTime = new Date(simulation.endTime);
    this.timeStep = simulation.timeStep; // in ms
  }
};
</script>

<style lang="scss">
</style>

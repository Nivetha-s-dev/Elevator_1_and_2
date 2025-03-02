<!-- App.vue -->
<template>
  <div id="app">
    <elevator-system
        ref="elevatorSystem"
        :total-floors="totalFloors"
        @elevator-arrived="handleElevatorArrival"
    />
    <div class="floors-section">
      <floor-doors
          v-for="floor in reversedFloors"
          :key="floor"
          :floor-number="floor"
          :total-floors="totalFloors"
          :ref="`floor${floor}`"
          @call-elevator="handleElevatorCall"
          @fire-emergency="handleFireEmergency"
      />
    </div>
  </div>
</template>

<script>
import ElevatorSystem from './components/assignment-components/ElevatorSystem.vue'
import FloorDoors from './components/assignment-components/FloorDoors.vue'

export default {
  name: 'App',
  components: {
    ElevatorSystem,
    FloorDoors
  },
  data() {
    return {
      totalFloors: 6
    }
  },
  computed: {
    reversedFloors() {
      return Array.from({ length: this.totalFloors }, (_, i) => this.totalFloors - i)
    }
  },
  methods: {

    handleElevatorCall(request) {
      this.$refs.elevatorSystem.handleElevatorRequest(request)
    },
    handleFireEmergency(isActive) {
      this.$refs.elevatorSystem.handleFireEmergency(isActive)
    },
    handleElevatorArrival({ floor, elevatorType }) {
      const floorComponent = this.$refs[`floor${floor}`]
      if (floorComponent && floorComponent[0]) {
        floorComponent[0].handleElevatorArrival(elevatorType)
      }
    },

  }
}
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  padding: 20px;
}

.main-container {
  display: flex;
  gap: 40px;
  max-width: 1400px;
  margin: 0 auto;
}

.floors-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}
</style>

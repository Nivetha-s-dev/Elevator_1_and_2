<template>
  <div class="elevator" :class="{ 'express': isExpress }">
    <div class="digital-display">
      <div class="floor-number">{{ currentFloor }}</div>
      <div class="direction-indicator">{{ direction }}</div>
      <div class="status" :class="{ 'emergency': isEmergencyMode }">
        {{ statusMessage }}
      </div>
      <div v-if="isExpress" class="express-indicator">EXPRESS</div>
    </div>

    <div class="elevator-panel">
      <div class="button-grid">
        <button
            v-for="floor in [6, 5, 4, 3, 2, 1]"
            :key="`internal-${floor}`"
            class="floor-button"
            :class="{
            'active': queuedFloors.includes(floor),
            'current': currentFloor === floor,
            'express-floor': isExpress && isExpressFloor(floor)
          }"
            :disabled="isButtonDisabled(floor)"
            @click="handleInternalCall(floor)"
        >
          {{ floor }}
        </button>
      </div>

      <div class="control-buttons">
        <button
            class="emergency-button"
            :class="{ 'active': isEmergencyMode }"
            @click="handleEmergencyAction"
        >
          EMERGENCY
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    id: {
      type: Number,
      required: true,
      default: 1
    },
    isExpress: {
      type: Boolean,
      default: false
    },
    initialFloor: {
      type: Number,
      default: 1
    }
  },

  data() {
    return {
      currentFloor: this.initialFloor,
      targetFloor: null,
      direction: '',
      queuedFloors: [],
      isMoving: false,
      isDoorsOpen: false,
      isEmergencyMode: false,
      statusMessage: 'Ready',
      movementTimeout: null,
      doorTimeout: null
    }
  },

  methods: {
    isExpressFloor(floor) {

      return floor === 1 || floor === 6 || floor === 5
    },
    moveToFloor(floor) {
      if (this.isEmergencyMode || floor === this.currentFloor) return

      this.isMoving = true
      this.targetFloor = floor
      this.direction = floor > this.currentFloor ? 'up' : 'down'
      this.statusMessage = 'Moving'

      const distance = Math.abs(floor - this.currentFloor)
      const timePerFloor = 1000 // 1 second per floor

      setTimeout(() => {
        this.currentFloor = floor
        this.isMoving = false
        this.direction = ''
        this.statusMessage = 'Arrived'
        this.$emit('floor-reached', this.id, floor)
        this.processQueue()
      }, distance * timePerFloor)
    },

    processQueue() {
      if (this.queuedFloors.length > 0 && !this.isEmergencyMode) {
        const nextFloor = this.queuedFloors.shift()
        this.moveToFloor(nextFloor)
      } else {
        this.direction = ''
        this.statusMessage = 'Ready'
      }
    },
    handleInternalCall(floor) {
      if (this.isEmergencyMode) return
      if (this.isExpress && !this.isExpressFloor(floor)) {
        this.statusMessage = 'Express elevator - Limited stops only'
        setTimeout(() => {
          this.statusMessage = this.isMoving ? 'Moving' : 'Ready'
        }, 2000)
        return
      }

      if (!this.isMoving) {
        this.moveElevator(floor)
      } else if (!this.queuedFloors.includes(floor)) {
        this.queuedFloors.push(floor)
        this.queuedFloors.sort((a, b) => {
          if (this.direction === 'up') {
            return a - b
          }
          return b - a
        })
      }
    },

    moveElevator(floor) {
      if (this.isEmergencyMode || floor === this.currentFloor) return

      this.isMoving = true
      this.targetFloor = floor
      this.direction = floor > this.currentFloor ? 'up' : 'down'
      this.statusMessage = 'Moving'
      const distance = Math.abs(floor - this.currentFloor)
      const timePerFloor = 1000
      this.movementTimeout = setTimeout(() => {
        this.currentFloor = floor
        this.direction = ''
        this.isMoving = false
        this.statusMessage = 'Arrived'
        this.$emit('floor-reached', this.id, floor)
        this.openDoors()
        setTimeout(() => {
          if (this.queuedFloors.length > 0 && !this.isEmergencyMode) {
            const nextFloor = this.queuedFloors.shift()
            this.moveElevator(nextFloor)
          } else {
            this.statusMessage = 'Ready'
          }
        }, 2000)
      }, distance * timePerFloor)
    },

    openDoors() {
      this.isDoorsOpen = true
      this.doorTimeout = setTimeout(() => {
        this.closeDoors()
      }, 1500)
    },

    closeDoors() {
      this.isDoorsOpen = false
    },

    handleEmergencyAction() {
      this.isEmergencyMode = !this.isEmergencyMode
      if (this.isEmergencyMode) {
        this.statusMessage = 'Emergency Mode'
        clearTimeout(this.movementTimeout)
        clearTimeout(this.doorTimeout)
        this.isMoving = false
        this.direction = ''
        this.queuedFloors = []
        this.openDoors()
      } else {
        this.statusMessage = 'Ready'
        this.closeDoors()
      }
    },

    isButtonDisabled(floor) {
      return (
          this.isEmergencyMode ||
          (this.isMoving && this.currentFloor === floor) ||
          (this.isExpress && !this.isExpressFloor(floor))
      )
    }
  }
}
</script>

<style scoped>
.elevator {
  border: 1px solid #ccc;
  padding: 20px;
  width: 300px;
  background-color: #f5f5f5;
  border-radius: 8px;
}

.elevator.express {
  border: 2px solid #ffd700;
  background-color: #fff8dc;
}

.digital-display {
  background-color: #000;
  color: #00ff00;
  padding: 15px;
  margin-bottom: 20px;
  border-radius: 4px;
  text-align: center;
}

.floor-number {
  font-size: 36px;
  font-weight: bold;
}

.direction-indicator {
  font-size: 18px;
  margin: 5px 0;
}

.status {
  font-size: 14px;
  color: #00ff00;
}

.status.emergency {
  color: #ff0000;
  animation: blink 1s infinite;
}

.express-indicator {
  color: #ffd700;
  font-weight: bold;
  margin-top: 5px;
  font-size: 16px;
}

.elevator-panel {
  background-color: #ddd;
  padding: 15px;
  border-radius: 4px;
}

.button-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin-bottom: 15px;
}

.floor-button {
  padding: 15px;
  font-size: 18px;
  border: none;
  border-radius: 4px;
  background-color: #fff;
  cursor: pointer;
  transition: background-color 0.3s;
}

.floor-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.floor-button.active {
  background-color: #4CAF50;
  color: white;
}

.floor-button.current {
  background-color: #2196F3;
  color: white;
}

.floor-button.express-floor {
  border: 2px solid #ffd700;
}

.control-buttons {
  text-align: center;
}

.emergency-button {
  background-color: #ff0000;
  color: white;
  padding: 15px 30px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.emergency-button.active {
  animation: blink 1s infinite;
}

@keyframes blink {
  50% {
    opacity: 0.5;
  }
}
</style>

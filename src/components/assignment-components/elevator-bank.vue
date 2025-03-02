<template>
  <div class="elevator-bank">
    <!-- Floor Panel (External Buttons) -->
    <div class="floor-panel">
      <div v-for="floor in [6, 5, 4, 3, 2, 1]" :key="`floor-${floor}`" class="floor-controls">
        <span>Floor {{ floor }}</span>
        <div class="direction-buttons">
          <button
              class="up-button"
              @click="handleFloorCall(floor)"
              :disabled="isButtonDisabled(floor)"
          >
            ▲
          </button>
          <button
              class="down-button"
              @click="handleFloorCall(floor)"
              :disabled="isButtonDisabled(floor)"
          >
            ▼
          </button>
        </div>
      </div>
    </div>

    <Elevator
        ref="elevator"
        :current-floor="elevatorState.currentFloor"
        :queued-floors="elevatorState.queuedFloors"
        :is-moving="elevatorState.isMoving"
        :direction="elevatorState.direction"
        @floor-reached="handleFloorReached"
        @internal-call="handleInternalCall"
    />
    <Floor
        v-for="floor in [6, 5, 4, 3, 2, 1]"
        :key="floor"
        :floor="floor"
        :current-floor="elevatorState.currentFloor"
        :door-status="elevatorState.doorStatus"
        :is-moving="elevatorState.isMoving"
        :is-emergency-mode="elevatorState.isEmergencyMode"
        @floor-call="handleFloorCall"
        @emergency-toggle="handleEmergencyToggle"
        @clear-queue="clearQueue"
    />
  </div>
</template>

<script>
import Elevator from './elevator.vue'

export default {
  name: 'ElevatorBank',
  components: {
    Elevator
  },

  data() {
    return {
      elevatorState: {
        currentFloor: 1,
        targetFloor: null,
        queuedFloors: [],
        isMoving: false,
        direction: 'IDLE',
        sameDirectionQueue: [],
        oppositeDirectionQueue: []
      }
    }
  },

  methods: {
    handleFloorCall(floor, direction) {
      if (!this.disableInputs) {
        if (direction === this.elevatorState.direction) {
          this.elevatorState.sameDirectionQueue.push(floor);
        } else {
          this.elevatorState.oppositeDirectionQueue.push(floor);
        }
        this.processQueues();
      }
    },
    clearQueue() {
      this.elevatorState.queuedFloors = [];
    },
    handleInternalCall(floor) {
      if (floor > this.elevatorState.currentFloor) {
        this.elevatorState.sameDirectionQueue.push(floor);
      } else {
        this.elevatorState.oppositeDirectionQueue.push(floor);
      }
      this.processQueues();
    },
    moveElevator(floor) {

      const distance = Math.abs(floor - this.currentFloor);
      const time = distance * this.speed;
      this.$refs.elevatorCar.style.transform = `translateY(${(floor - this.currentFloor) * this.floorHeight}px)`;
      this.currentFloor = floor;
      this.stopElevator();
      this.openDoors();
      setTimeout(() => {
        this.closeDoors();
        if (this.queuedFloors.length > 0) {
          const nextFloor = this.queuedFloors[0];

          this.moveElevator(nextFloor);

          this.queuedFloors.shift();
        }
      }, 3000);
    },


      handleFloorReached(floor)
      {
        this.elevatorState.currentFloor = floor;
        this.elevatorState.queuedFloors = this.elevatorState.queuedFloors.filter(f => f !== floor);
        this.elevatorState.isMoving = this.elevatorState.queuedFloors.length > 0;
        this.elevatorState.direction = this.getNextDirection();
        setTimeout(() => {
          if (this.elevatorState.queuedFloors.length > 0) {
            this.processQueues();
          }
        }, 3000);
      }

    },

    getNextDirection() {
      if (this.elevatorState.queuedFloors.length === 0) return 'IDLE';
      const nextFloor = this.elevatorState.queuedFloors[0];
      return nextFloor > this.elevatorState.currentFloor ? 'UP' : 'DOWN';
    },
  processQueues() {

    if (this.elevatorState.sameDirectionQueue.length > 0) {
      const nextFloor = this.elevatorState.sameDirectionQueue.shift();
      this.elevatorState.queuedFloors.push(nextFloor);
      this.$refs.elevator.moveElevator(nextFloor);
    } else if (this.elevatorState.oppositeDirectionQueue.length > 0) {

      const nextFloor = this.elevatorState.oppositeDirectionQueue.shift();
      this.elevatorState.queuedFloors.push(nextFloor);
      this.$refs.elevator.moveElevator(nextFloor);
    }
  }

}
</script>


<style scoped>
.elevator-bank {
  display: flex;
  gap: 2rem;
  padding: 2rem;
  background-color: #f5f5f5;
  min-height: 100vh;
}

.floor-panel-section {
  flex: 0 0 300px;
}

.floor-panel {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border: 1px solid #ddd;
  margin-bottom: 0.5rem;
  background-color: white;
}

.floor-number {
  font-weight: bold;
}

.floor-buttons {
  display: flex;
  gap: 0.5rem;
}

.direction-button {
  padding: 0.5rem 1rem;
  border: 1px solid #ccc;
  background-color: #fff;
  cursor: pointer;
}

.direction-button.active {
  background-color: #4CAF50;
  color: white;
  border-color: #45a049;
}

.direction-button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

.elevator-container {
  flex: 1;
}
</style>

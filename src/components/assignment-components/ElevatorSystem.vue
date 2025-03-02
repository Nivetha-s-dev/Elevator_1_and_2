<!-- ElevatorSystem.vue -->
<template>
  <div class="elevator-system" :class="{ 'fire-mode': isFireMode }">
    <div class="system-status" v-if="isFireMode">
      FIRE EMERGENCY MODE ACTIVE
    </div>
    <div class="elevators-container">
      <elevator
          ref="regularElevator"
          :id="1"
          :initial-floor="1"
          :is-express="false"
          :is-fire-mode="isFireMode"
          :total-floors="totalFloors"
          @floor-reached="handleFloorReached"
      />
      <elevator
          ref="expressElevator"
          :id="2"
          :initial-floor="1"
          :is-express="true"
          :is-fire-mode="isFireMode"
          :total-floors="totalFloors"
          @floor-reached="handleFloorReached"
      />
    </div>
  </div>
</template>
<script>
import Elevator from './Elevator.vue'

export default {
  name: 'ElevatorSystem',
  components: {
    Elevator
  },
  props: {
    totalFloors: {
      type: Number,
      default: 6
    }
  },
  data() {
    return {
      regular: null,
      express: null,
      isFireMode: false,
      elevatorStates: {
        regular: {
          currentFloor: 1,
          targetFloor: null,
          direction: null,
          isMoving: false,
          queue: []
        },
        express: {
          currentFloor: 1,
          targetFloor: null,
          direction: null,
          isMoving: false,
          queue: []
        }
      }
    }
  },
  mounted() {
    this.regular = this.$refs.regularElevator
    this.express = this.$refs.expressElevator
  },
  methods: {
    isExpressFloor(floor) {
      return floor === 1 ||
          floor === this.totalFloors ||
          floor === (this.totalFloors - 1)
    },
    handleFireEmergency(isActive) {
      this.isFireMode = isActive

      if (isActive) {

        this.clearAllQueues()


        this.sendElevatorsToFirstFloor()
      }
    },
    clearAllQueues() {
      const regular = this.$refs.regularElevator
      const express = this.$refs.expressElevator

      regular.queuedFloors = []
      express.queuedFloors = []
    },
    handleElevatorRequest({ floor, direction, isExpressFloor }) {

      if (this.isFireMode) {
        console.log('Elevator system in fire emergency mode')
        return
      }
      console.log(`Request: Floor ${floor}, Direction ${direction}, Express: ${isExpressFloor}`)


      this.updateElevatorStates()


      if (isExpressFloor) {
        const express = this.$refs.expressElevator
        if (!express.isMoving) {
          express.moveToFloor(floor)
        } else {
          this.addToDirectionalQueue(express, floor, direction)
        }
        return
      }


      const regular = this.$refs.regularElevator
      const express = this.$refs.expressElevator


      const sameDirectionElevators = []

      if (regular.direction === direction || !regular.isMoving) {
        sameDirectionElevators.push(regular)
      }

      if ((express.direction === direction || !express.isMoving) && !express.isExpressOnly) {
        sameDirectionElevators.push(express)
      }

      if (sameDirectionElevators.length > 0) {

        const closest = this.findClosestElevator(sameDirectionElevators, floor)
        this.assignRequest(closest, floor, direction)
      } else {

        const regularWaitTime = this.calculateWaitTime(regular)
        const expressWaitTime = this.calculateWaitTime(express)

        const soonestAvailable = regularWaitTime <= expressWaitTime ? regular : express
        this.assignRequest(soonestAvailable, floor, direction)
      }
    },
    sendElevatorsToFirstFloor() {
      const regular = this.$refs.regularElevator
      const express = this.$refs.expressElevator

      if (regular.currentFloor !== 1) {
        regular.moveToFloor(1)
      }

      if (express.currentFloor !== 1) {
        express.moveToFloor(1)
      }
    },


    updateElevatorStates() {
      this.elevatorStates.regular = {
        currentFloor: this.regular.currentFloor,
        targetFloor: this.regular.targetFloor,
        direction: this.regular.direction,
        isMoving: this.regular.isMoving,
        queue: [...this.regular.queuedFloors]
      }

      this.elevatorStates.express = {
        currentFloor: this.express.currentFloor,
        targetFloor: this.express.targetFloor,
        direction: this.express.direction,
        isMoving: this.express.isMoving,
        queue: [...this.express.queuedFloors]
      }
    },

    handleExpressRequest(floor, direction) {
      const express = this.$refs.expressElevator
      if (!express.isMoving) {
        express.moveToFloor(floor)
      } else {

        this.addToDirectionalQueue(express, floor, direction)
      }
      this.$emit('request-assigned', { floor, elevatorId: 2 })
    },

    handleRegularRequest(floor, direction) {
      const regular = this.$refs.regularElevator
      const express = this.$refs.expressElevator


      const sameDirectionElevators = this.findElevatorsInDirection(direction)

      if (sameDirectionElevators.length > 0) {

        const closest = this.findClosestElevator(sameDirectionElevators, floor)
        this.assignRequest(closest, floor, direction)
      } else {

        const soonestAvailable = this.findSoonestAvailable(regular, express)
        this.assignRequest(soonestAvailable, floor, direction)
      }
    },

    findElevatorsInDirection(direction) {
      const elevators = []
      const regular = this.$refs.regularElevator
      const express = this.$refs.expressElevator

      if (regular.direction === direction) elevators.push(regular)
      if (express.direction === direction && !express.isExpressOnly) elevators.push(express)

      return elevators
    },

    findClosestElevator(elevators, targetFloor) {
      return elevators.reduce((closest, current) => {
        const closestDistance = this.calculateDistance(closest, targetFloor)
        const currentDistance = this.calculateDistance(current, targetFloor)
        return currentDistance < closestDistance ? current : closest
      })
    },

    calculateDistance(elevator, targetFloor) {
      if (!elevator.isMoving) {
        return Math.abs(elevator.currentFloor - targetFloor)
      }


      const distanceToTarget = Math.abs(elevator.currentFloor - targetFloor)
      const remainingCurrentJourney = Math.abs(elevator.currentFloor - elevator.targetFloor)

      return distanceToTarget + remainingCurrentJourney
    },

    findSoonestAvailable(regular, express) {

      if (!regular.isMoving) return regular
      if (!express.isMoving && !express.isExpressOnly) return express


      const regularWaitTime = this.calculateWaitTime(regular)
      const expressWaitTime = this.calculateWaitTime(express)

      return regularWaitTime <= expressWaitTime ? regular : express
    },

    calculateWaitTime(elevator) {
      const currentJourneyTime = Math.abs(elevator.currentFloor - elevator.targetFloor)
      const queueTime = elevator.queuedFloors.reduce((total, floor) => {
        return total + Math.abs(floor - elevator.targetFloor)
      }, 0)

      return currentJourneyTime + queueTime
    },

    assignRequest(elevator, floor, direction) {
      if (!elevator.isMoving) {
        elevator.moveToFloor(floor)
      } else {
        this.addToDirectionalQueue(elevator, floor, direction)
      }

      this.$emit('request-assigned', {
        floor,
        elevatorId: elevator === this.regular ? 1 : 2
      })
    },
    handleFloorReached(elevatorId, floor) {
      console.log(`Elevator ${elevatorId} reached floor ${floor}`);
      this.$emit('elevator-arrived', {
        floor,
        elevatorType: elevatorId === 1 ? 'regular' : 'express'
      });
    }
  },

    addToDirectionalQueue(elevator, floor, direction) {
      const queue = [...elevator.queuedFloors]

      // Insert floor in queue based on direction
      if (direction === 'up') {
        queue.push(floor)
        queue.sort((a, b) => a - b)
      } else {
        queue.push(floor)
        queue.sort((a, b) => b - a)
      }

      elevator.queuedFloors = queue
    },

    handleFloorReached(elevatorId, floor) {
      console.log(`Elevator ${elevatorId} reached floor ${floor}`)


      this.updateElevatorStates()


      this.$emit('elevator-arrived', {
        floor,
        elevatorType: elevatorId === 1 ? 'regular' : 'express'
      })
    }

}
</script>

<style scoped>
.elevator-system {
  display: flex;
  justify-content: center;
  padding: 20px;
  background-color: #f0f0f0;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.system-status {
  background-color: #ff3b30;
  color: white;
  text-align: center;
  padding: 10px;
  font-weight: bold;
  margin-bottom: 10px;
  border-radius: 4px;
  animation: flash 1s infinite;
}

.fire-mode {
  border: 2px solid #ff3b30;
}

@keyframes flash {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}
.elevators-container {
  display: flex;
  gap: 40px;
  padding: 20px;
  background-color: white;
  border-radius: 4px;
  box-shadow: inset 0 0 4px rgba(0, 0, 0, 0.1);
}
.system-status {
  background-color: #ff3b30;
  color: white;
  text-align: center;
  padding: 10px;
  font-weight: bold;
  margin-bottom: 10px;
  border-radius: 4px;
  animation: flash 1s infinite;
}

.fire-mode {
  border: 2px solid #ff3b30;
}

@keyframes flash {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}
.elevator-status {
  position: absolute;
  top: -25px;
  left: 0;
  right: 0;
  text-align: center;
  font-size: 12px;
  color: #666;
}

@media (max-width: 768px) {
  .elevators-container {
    gap: 20px;
    padding: 10px;
  }
}
</style>

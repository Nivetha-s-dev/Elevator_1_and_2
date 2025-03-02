<!-- FloorDoors.vue -->
<template>
  <div class="floor-doors-container">
    <div class="floor-info">

      <div class="floor-number-panel">
        <span class="floor-number">{{ floorNumber }}</span>
      </div>


      <div class="direction-buttons">
        <button
            class="direction-btn up"
            :class="{ 'active': upButtonPressed }"
            @click="callElevatorWithDirection('up')"
            :disabled="isTopFloor || anyDoorsOpen"
        >
          ▲
        </button>
        <button
            class="direction-btn down"
            :class="{ 'active': downButtonPressed }"
            @click="callElevatorWithDirection('down')"
            :disabled="isBottomFloor || anyDoorsOpen"
        >
          ▼
        </button>
      </div>

      <div class="doors-container">
        <!-- Regular Elevator -->
        <div class="elevator-unit regular">
          <div class="elevator-label">REGULAR</div>
          <div class="elevator-doors">
            <div class="doors" :class="{ 'open': regularDoorsOpen }">
              <div class="door left"></div>
              <div class="door right"></div>
            </div>
            <div class="door-status">
              {{ regularDoorsOpen ? 'OPEN' : 'CLOSED' }}
            </div>
          </div>
        </div>


        <div class="elevator-unit express" :class="{ 'disabled': !isExpressFloor }">
          <div class="elevator-label">EXPRESS</div>
          <div class="elevator-doors">
            <div class="doors" :class="{ 'open': expressDoorsOpen }">
              <div class="door left"></div>
              <div class="door right"></div>
            </div>
            <div class="door-status">
              {{ isExpressFloor
                ? (expressDoorsOpen ? 'OPEN' : 'CLOSED')
                : 'NO ACCESS'
              }}
            </div>
          </div>
        </div>
      </div>


      <div v-if="floorNumber === 1" class="fire-emergency-container">
        <button
            class="fire-button"
            :class="{ 'active': isFireMode }"
            @click="handleFireEmergency"
        >

          <span class="fire-text">IN CASE OF FIRE</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FloorDoors',
  props: {
    floorNumber: {
      type: Number,
      required: true
    },
    totalFloors: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      regularDoorsOpen: false,
      expressDoorsOpen: false,
      upButtonPressed: false,
      downButtonPressed: false,
      isFireMode: false
    }
  },
  computed: {
    isExpressFloor() {
      return this.floorNumber === 1 ||
          this.floorNumber === this.totalFloors ||
          this.floorNumber === (this.totalFloors - 1)
    },
    isTopFloor() {
      return this.floorNumber === this.totalFloors
    },
    isBottomFloor() {
      return this.floorNumber === 1
    },
    anyDoorsOpen() {
      return this.regularDoorsOpen || this.expressDoorsOpen
    }
  },
  methods: {
    callElevatorWithDirection(direction) {
      if (direction === 'up') {
        this.upButtonPressed = true
      } else {
        this.downButtonPressed = true
      }

      this.$emit('call-elevator', {
        floor: this.floorNumber,
        direction: direction,
        isExpressFloor: this.isExpressFloor
      })
    },
    openRegularDoors() {
      console.log(`Opening regular doors on floor ${this.floorNumber}`);
      this.regularDoorsOpen = true;
      this.resetDirectionButtons();

      // Ensure the door closes after a delay
      setTimeout(() => {
        this.closeRegularDoors();
      }, 3000);
    },

    closeRegularDoors() {
      console.log(`Closing regular doors on floor ${this.floorNumber}`);
      this.regularDoorsOpen = false;
    },

    openExpressDoors() {
      if (this.isExpressFloor) {
        console.log(`Opening express doors on floor ${this.floorNumber}`);
        this.expressDoorsOpen = true;
        this.resetDirectionButtons();

        // Ensure the door closes after a delay
        setTimeout(() => {
          this.closeExpressDoors();
        }, 3000);
      }
    },

    closeExpressDoors() {
      console.log(`Closing express doors on floor ${this.floorNumber}`);
      this.expressDoorsOpen = false;
    },


    handleElevatorArrival(elevatorType) {
      console.log(`Handling ${elevatorType} elevator arrival at floor ${this.floorNumber}`);
      if (elevatorType === 'regular') {
        this.openRegularDoors();
      } else if (elevatorType === 'express' && this.isExpressFloor) {
        this.openExpressDoors();
      }
    }
  },
    handleFireEmergency() {
      this.isFireMode = !this.isFireMode
      this.$emit('fire-emergency', this.isFireMode)
    },

    resetDirectionButtons() {
      this.upButtonPressed = false
      this.downButtonPressed = false
    },

    openRegularDoors() {
      this.regularDoorsOpen = true
      this.resetDirectionButtons()
      setTimeout(() => {
        this.closeRegularDoors()
      }, 3000)
    },

    closeRegularDoors() {
      this.regularDoorsOpen = false
    },

    openExpressDoors() {
      if (this.isExpressFloor) {
        this.expressDoorsOpen = true
        this.resetDirectionButtons()
        setTimeout(() => {
          this.closeExpressDoors()
        }, 3000)
      }
    },

    closeExpressDoors() {
      this.expressDoorsOpen = false
    }
}
</script>

<style scoped>
.floor-doors-container {
  border: 1px solid #ccc;
  padding: 12px;
  margin: 5px 0;
  border-radius: 4px;
  background-color: #f5f5f5;
  max-height: 140px;
}

.floor-info {
  display: flex;
  align-items: center;
  gap: 20px;
}

.floor-number-panel {
  background-color: #000;
  color: #00ff00;
  padding: 8px 12px;
  border-radius: 4px;
  min-width: 40px;
  text-align: center;
}

.floor-number {
  font-size: 24px;
  font-weight: bold;
  font-family: 'Digital-7', monospace;
}

.direction-buttons {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.direction-btn {
  width: 34px;
  height: 34px;
  border: none;
  border-radius: 4px;
  background-color: #ddd;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  transition: all 0.3s ease;
}

.direction-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.direction-btn.active {
  background-color: #4CAF50;
  color: white;
}

.doors-container {
  display: flex;
  gap: 30px;
}

.elevator-unit {
  width: 100px;
  text-align: center;
}

.elevator-label {
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 4px;
  padding: 2px 4px;
  border-radius: 2px;
}

.elevator-unit.regular .elevator-label {
  background-color: #2196F3;
  color: white;
}

.elevator-unit.express .elevator-label {
  background-color: #ffd700;
  color: black;
}

.elevator-doors {
  position: relative;
}

.doors {
  height: 60px;
  background: #333;
  position: relative;
  display: flex;
  overflow: hidden;
  border-radius: 2px;
}

.door {
  width: 50%;
  height: 100%;
  background: #666;
  transition: transform 0.5s ease-in-out;
  position: absolute;
}

.door.left {
  left: 0;
  transform-origin: left;
}

.door.right {
  right: 0;
  transform-origin: right;
}

.doors.open .door.left {
  transform: translateX(-100%);
}

.doors.open .door.right {
  transform: translateX(100%);
}

.elevator-unit.regular .doors {
  border: 2px solid #2196F3;
}

.elevator-unit.express .doors {
  border: 2px solid #ffd700;
}

.door-status {
  font-size: 10px;
  font-weight: bold;
  color: #666;
  margin-top: 4px;
  text-transform: uppercase;
}

.elevator-unit.disabled {
  opacity: 0.6;
}

.fire-emergency-container {
  margin-left: auto;
  padding: 0 15px;
}

.fire-button {
  background-color: #ff3b30;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px 20px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  min-width: 180px;
  height: 44px;
  justify-content: center;
}

.fire-button:hover {
  background-color: #ff1a1a;
  transform: scale(1.05);
}

.fire-button.active {
  background-color: #cc0000;
  animation: pulse 2s infinite;
}

.fire-icon {
  font-size: 20px;
}

.fire-text {
  font-size: 14px;
  letter-spacing: 0.5px;
}

/* Door animation effect */
.doors::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.3);
  pointer-events: none;
  z-index: 1;
}

@keyframes pulse {
  0% {
    opacity: 1;
    transform: scale(1);
  }
  50% {
    opacity: 0.8;
    transform: scale(1.05);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@media (max-width: 768px) {
  .floor-doors-container {
    padding: 8px;
  }

  .floor-info {
    gap: 10px;
    flex-wrap: wrap;
  }

  .fire-emergency-container {
    width: 100%;
    margin: 10px 0;
    padding: 0;
  }

  .fire-button {
    width: 100%;
  }
}
</style>

<template>
  <div class="floor-panel">
    <div v-for="floor in [1,2,3,4,5,6]"
         :key="floor"
         class="floor-level"
         :class="{ 'current': currentFloor === floor }">

      <div class="floor-info">
        <span class="floor-number">Floor {{ floor }}</span>
        <!-- Emergency button on first floor -->
        <button
            v-if="floor === 1"
            class="emergency-btn"
            :class="{ 'active': isEmergencyMode }"
            @click="toggleEmergency"
        >
          In Case of Fire
        </button>
      </div>

      <div class="call-buttons">
        <button
            v-if="floor !== 6"
            class="direction-btn up"
            :disabled="isButtonPressed(floor)"
            :class="{ 'pressed': isButtonPressed(floor)}"
            @click="callElevator(floor, $event)"
        >
          ▲
        </button>
        <button
            v-if="floor !== 1"
            class="direction-btn down"
            :disabled="isButtonPressed(floor)"
            :class="{ 'pressed': isButtonPressed(floor) }"
            @click="callElevator(floor, $event)"
        >
          ▼
        </button>
      </div>

      <div class="door-indicator"
           :class="{
             'open': isDoorOpen && currentFloor === floor,
             'moving': isDoorMoving && currentFloor === floor
           }">
        <div class="door left"></div>
        <div class="door right"></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Floor',
  props: {
    currentFloor: {
      type: Number,
      required: true
    },
    doorStatus: {
      type: String,
      required: true
    },
    isMoving: {
      type: Boolean,
      default: false
    },
    isEmergencyMode: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      pressedFloor: null,


    }
  },
  computed: {
    isDoorOpen() {
      return this.doorStatus === 'OPEN';
    },
    isDoorMoving() {
      return this.doorStatus === 'OPENING' || this.doorStatus === 'CLOSING';
    }
  },
  methods: {
    isButtonPressed(floor, direction) {
      return this.pressedFloor === floor && this.pressedDirection === direction;
    },
    toggleFireMode() {
      this.isFireMode = !this.isFireMode;
      if (this.isFireMode) {
        this.$emit('emergency-toggle', true);
        this.$emit('clear-queue');
        this.requestFloor(1);
      } else {
        this.$emit('emergency-toggle', false);
      }
    },

    callElevator(floor, event) {
      event.stopPropagation();
      if (this.isEmergencyMode) {
        return;
      }
      if (!this.isButtonPressed(floor)) {
        this.pressedFloor = floor;

        this.$emit('floor-call', floor);
        this.$emit('elevator-call', floor);
        // Adjust the delay as needed
      }
    },
    toggleEmergency() {
      this.$emit('toggle-emergency');
      if (!this.isEmergencyMode) {
        // When activating emergency mode, immediately call elevator to first floor
        this.$emit('floor-call', 1);
      }
    },
    resetButton(floor) {
      if (this.pressedFloor === floor) {
        this.pressedFloor = null;
      }
    }
  }
}
</script>

<style scoped>
.floor-panel {
  display: flex;
  flex-direction: column-reverse;
  gap: 15px;
  padding: 10px;
  background: #f8f9fa;
  border-radius: 8px;
  height: 600px;
  overflow-y: auto;
}

.floor-level {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  min-height: 80px;
}

.floor-level.current {
  background: #e8f5e9;
  border-left: 4px solid #4CAF50;
}

.floor-info {
  width: 100px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.floor-number {
  font-size: 16px;
  font-weight: bold;
  color: #2c3e50;
}
.emergency-btn.active {
  background-color: #c0392b;
  animation: pulse 1s infinite;
}
.call-buttons {
  display: flex;
  gap: 8px;
}

.direction-btn {
  width: 40px;
  height: 40px;
  border: none;
  border-radius: 50%;
  background: #34495e;
  color: white;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.direction-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}


.direction-btn.bright {
  background-color: #33CC33;
}

.emergency-btn {
  margin-top: 8px;
  padding: 8px 16px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.emergency-btn:hover {
  background-color: #c82333;
}

.emergency-btn.active {
  background-color: #721c24;
  animation: pulse 2s infinite;
}


@keyframes pulse {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    opacity: 1;
  }
}

.door-indicator {
  width: 60px;
  height: 60px;
  background: #34495e;
  position: relative;
  overflow: hidden;
  border-radius: 4px;
}

.door {
  position: absolute;
  width: 50%;
  height: 100%;
  background: #95a5a6;
  transition: transform 1.5s ease-in-out;
}

.door.left {
  left: 0;
}

.door.right {
  right: 0;
}

.door-indicator.open .left {
  transform: translateX(-100%);
}

.door-indicator.open .right {
  transform: translateX(100%);
}

@media (max-width: 768px) {
  .floor-panel {
    height: 500px;
  }

  .floor-level {
    min-height: 70px;
  }
}
</style>

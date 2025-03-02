<script>
export default {
  name: "door",
  props: {
    panelType: {
      type: String,
      default: ""
    },
    doorStatus: {
      type: String,
      default: "CLOSED"
    },
    isCurrentFloor: {
      type: Boolean,
      default: false
    },
    isInQueue: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      isDoorOpen: false,
      isDoorClosing: false,
      isDoorOpening: false,
    }
  },
  watch: {
    doorStatus: {
      immediate: true,
      handler(newStatus) {
        this.updateDoorState(newStatus);
      }
    }
  },
  methods: {
    updateDoorState(status) {
      switch (status) {
        case 'OPENING':
          this.isDoorOpening = true;
          this.isDoorClosing = false;
          this.isDoorOpen = false;
          break;
        case 'OPEN':
          this.isDoorOpen = true;
          this.isDoorOpening = false;
          this.isDoorClosing = false;
          break;
        case 'CLOSING':
          this.isDoorClosing = true;
          this.isDoorOpening = false;
          this.isDoorOpen = false;
          break;
        case 'CLOSED':
          this.isDoorOpen = false;
          this.isDoorOpening = false;
          this.isDoorClosing = false;
          break;
      }
    }
  }
}
</script>

<template>
  <div class="elevator-door">
    <div class="door-container" :class="{
            'current-floor': isCurrentFloor,
            'in-queue': isInQueue
        }">
      <div class="door-frame">
        <div class="door left-door" :class="{
                    'open': isDoorOpen,
                    'opening': isDoorOpening,
                    'closing': isDoorClosing
                }"></div>
        <div class="door right-door" :class="{
                    'open': isDoorOpen,
                    'opening': isDoorOpening,
                    'closing': isDoorClosing
                }"></div>
      </div>
      <div class="door-status">{{ doorStatus }}</div>
    </div>
  </div>
</template>

<style scoped>
.elevator-door {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
}

.door-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.door-frame {
  width: 60px;
  height: 80px;
  background: #444;
  position: relative;
  overflow: hidden;
  border-radius: 2px;
}

.door {
  position: absolute;
  width: 50%;
  height: 100%;
  background: #71797E;
  transition: transform 0.5s ease-in-out;
}

.left-door {
  left: 0;
  transform-origin: left;
}

.right-door {
  right: 0;
  transform-origin: right;
}

/* Door States */
.left-door.open {
  transform: translateX(-100%);
}

.right-door.open {
  transform: translateX(100%);
}

.left-door.opening {
  animation: leftDoorOpen 1.5s ease-in-out;
}

.right-door.opening {
  animation: rightDoorOpen 1.5s ease-in-out;
}

.left-door.closing {
  animation: leftDoorClose 1.5s ease-in-out;
}

.right-door.closing {
  animation: rightDoorClose 1.5s ease-in-out;
}

@keyframes leftDoorOpen {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(-100%);
  }
}

@keyframes rightDoorOpen {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(100%);
  }
}

@keyframes leftDoorClose {
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(0);
  }
}

@keyframes rightDoorClose {
  from {
    transform: translateX(100%);
  }
  to {
    transform: translateX(0);
  }
}

.door-status {
  font-size: 12px;
  color: #666;
}

/* Floor States */
.current-floor .door-frame {
  border: 2px solid #4CAF50;
}

.in-queue .door-frame {
  border: 2px solid #FFC107;
}
</style>

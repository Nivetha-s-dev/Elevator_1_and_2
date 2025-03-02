<!-- src/components/Assignment.vue -->
<template>
  <div class="assignment">
    <h2>Elevator System</h2>
    <ElevatorSystem />
  </div>
</template>

<script>
import ElevatorSystem from './ElevatorSystem.vue'

export default {
  name: 'Assignment',
  components: {
    ElevatorSystem
  }
}
</script>

<style scoped>
.assignment {
  padding: 20px;
  max-width: 800px;
  margin: 0 auto;
}
</style>

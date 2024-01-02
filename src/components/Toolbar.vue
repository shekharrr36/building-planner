<template>
  <div>
    <button @click="addRectangle">Add Rectangle</button>
    <!-- Add buttons for other shapes -->
    <button @click="enableSelectTool">Select Tool</button>
    <button @click="toggleAnnotations">Toggle Annotations</button>
  </div>
</template>

<script>
export default {
  methods: {
    addRectangle() {
      // Implement logic to add rectangles to the drawing area
    },
    // Implement methods for other shapes
    enableSelectTool() {
      // Implement logic to enable the select tool
    },
    toggleAnnotations() {
      // Implement logic to show/hide annotations
    }
  }
};
</script>

<style scoped>
/* Add styling as needed */
</style>

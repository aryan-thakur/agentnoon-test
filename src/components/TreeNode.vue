// components/TreeNode.vue
<script setup>
import { computed, ref } from "vue";

const props = defineProps({
  node: {
    type: Object,
    required: true,
  },
});

// Helper to recursively close descendants
const closeDescendants = (node) => {
  if (node.children) {
    node.children.forEach((child) => {
      if (child._displayState) {
        child._displayState.isOpen = false;
      }
      closeDescendants(child);
    });
  }
};

// Method to toggle children visibility
const toggleChildren = () => {
  if (!hasChildren.value) return; // Don't toggle if no children

  const currentState = props.node._displayState.isOpen;
  const newState = !currentState;
  props.node._displayState.isOpen = newState;

  // If we are closing the node, recursively close all descendants
  if (!newState) {
    closeDescendants(props.node);
  }
};

const getRandomColor = () => {
  const colors = ["#2c3e50", "#34495e", "#2b4865", "#3c4c65"];
  return colors[Math.floor(Math.random() * colors.length)];
};

const getDarkerVersion = (color) => {
  switch (color) {
    case "#2c3e50":
      return "#1f2a38"; // darker version of #2c3e50
    case "#34495e":
      return "#263545"; // darker version of #34495e
    case "#2b4865":
      return "#1e3247"; // darker version of #2b4865
    case "#3c4c65":
      return "#2b384a"; // darker version of #3c4c65
    default:
      return "#2c3e50";
  }
};

function formatNumber(num) {
  if (num >= 1_000_000_000) {
    return (num / 1_000_000_000).toFixed(2) + "B";
  } else if (num >= 1_000_000) {
    return (num / 1_000_000).toFixed(2) + "M";
  } else if (num >= 1_000) {
    return (num / 1_000).toFixed(2) + "K";
  } else {
    return num.toString();
  }
}

const cardColor = ref(getRandomColor());
const tagColor = getDarkerVersion(cardColor);

// Computed property to check if the node has children
const hasChildren = computed(() => {
  return props.node.children && props.node.children.length > 0;
});

// Computed property for indentation based on depth
const indentStyle = computed(() => {
  return {
    marginLeft: `${props.node.depth * 6}px`,
    marginRight: `${props.node.depth * 6}px`,
  }; // Adjust 24px as needed
});

// Computed property for the toggle icon
const toggleIcon = computed(() => {
  if (!hasChildren.value) return ""; // No icon if no children
  return props.node._displayState.isOpen ? "▼" : "▶"; // Simple text icons
});
</script>

<template>
  <div class="flex flex-col gap-6">
    <div class="flex justify-center relative">
      <div
        class="p-4 pt-6 rounded-lg shadow-md hover:shadow-lg transition-shadow duration-300 cursor-pointer flex flex-col w-64 min-h-fit bg-opacity-75"
        :style="[indentStyle, { backgroundColor: cardColor }]"
        @click="toggleChildren"
      >
        <img
          :src="node.data.Photo"
          alt="Profile"
          class="w-12 h-12 rounded-full object-cover absolute -top-6 left-1/2 transform -translate-x-1/2 border-4 border-white shadow-md"
        />
        <div class="pt-2 text-white font-semibold text-lg text-center">
          {{ node.data.Name }}
        </div>

        <!-- Position -->
        <div class="text-white text-sm text-center opacity-80">
          {{ node.data["Job Title"] }}
        </div>

        <!-- Tags (Individual Properties) -->
        <div class="pt-6 flex flex-wrap gap-2 justify-center">
          <span
            class="px-2 py-1 rounded-full text-xs font-medium"
            :style="{
              backgroundColor: tagColor,
              color: '#f0f0f0',
            }"
          >
            {{ node.data.Department || "Department N/A" }}
          </span>
          <span
            class="px-2 py-1 rounded-full text-xs font-medium"
            :style="{
              backgroundColor: tagColor,
              color: '#f0f0f0',
            }"
          >
            {{ node.data.Location || "Location N/A" }}
          </span>
          <span
            class="px-2 py-1 rounded-full text-xs font-medium"
            :style="{
              backgroundColor: tagColor,
              color: '#f0f0f0',
            }"
          >
            Level:{{ node.data.level || "N/A" }}
          </span>
          <span
            class="px-2 py-1 rounded-full text-xs font-medium"
            :style="{
              backgroundColor: tagColor,
              color: '#f0f0f0',
            }"
          >
            Management cost: ${{ formatNumber(node.data.managerCost) || "0" }}
          </span>
          <span
            class="px-2 py-1 rounded-full text-xs font-medium"
            :style="{
              backgroundColor: tagColor,
              color: '#f0f0f0',
            }"
          >
            IC cost: ${{ formatNumber(node.data.individualCost) || "0" }}
          </span>
          <span
            class="px-2 py-1 rounded-full text-xs font-medium"
            :style="{
              backgroundColor: tagColor,
              color: '#f0f0f0',
            }"
          >
            Total cost: ${{ formatNumber(node.data.totalCost) || "0" }}
          </span>
          <span
            class="px-2 py-1 rounded-full text-xs font-medium"
            :style="{
              backgroundColor: tagColor,
              color: '#f0f0f0',
            }"
          >
            Management cost ratio:
            {{ node.data.managementCostRatio.toFixed(2) || "0" }}
          </span>
        </div>
        <div class="pt-2">
          <span class="text-center text-gray-500 mr-1 shrink-0">
            {{ toggleIcon }} ({{ node.children ? node.children.length : 0 }}/{{
              node.data.recursiveChildren
            }})
          </span>
        </div>
      </div>
    </div>
    <!-- Children Container -->
    <!-- Recursively render children -->
    <div
      v-if="hasChildren && node._displayState.isOpen"
      class="transition-all duration-300 ease-in-out w-fit flex flex-row flex-nowrap"
    >
      <TreeNode
        v-for="child in node.children"
        :key="child['Employee ID']"
        :node="child"
      />
    </div>
  </div>
</template>

<style scoped>
/* Optional: Add transition effects for opening/closing if needed */
/* Tailwind handles transitions via classes, but animating height requires more complex setups */
/* Simple fade/opacity is easier if desired */
</style>

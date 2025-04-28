// components/TreeNode.vue
<script setup>
import {
  computed,
  ref,
  onMounted,
  onBeforeUnmount,
  watch,
  nextTick,
} from "vue";

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
const lines = ref([]);

const myKey = computed(() => props.node.data["Employee Id"]);
const childKeys = computed(() => {
  return (props.node.children || []).map((child) => child.data["Employee Id"]);
});

function drawLines() {
  clearLines();

  // Check if should draw
  if (!props.node._displayState?.isOpen) {
    return;
  }

  // Bottom of working card
  const fromElem = document.querySelector(
    `[data-key="${myKey.value}"] .starter`
  );

  // to top of profile picture
  childKeys.value.forEach((childKey) => {
    const toElem = document.querySelector(`[data-key="${childKey}"] .connect`);
    if (fromElem && toElem) {
      const line = new LeaderLine(fromElem, toElem, {
        startPlug: "behind",
        endPlug: "arrow1",
        path: "grid",
        startSocket: "bottom",
        endSocket: "top",
        color: "black",
        size: 2,
      });
      lines.value.push(line);
    }
  });
}

// Clear all active LeaderLines
function clearLines() {
  lines.value.forEach((line) => {
    line.remove();
  });
  lines.value = [];
}

// Get a random color for the background of a card
const getRandomColor = () => {
  const colors = ["#367C8D", "#2E7D32", "#5E35B1", "#455A64"];
  return colors[Math.floor(Math.random() * colors.length)];
};

// Darken a set color
const getDarkerVersion = (color) => {
  switch (color) {
    case "#367C8D":
      return "#1E525E";
    case "#2E7D32":
      return "#1B5E20";
    case "#5E35B1":
      return "#311B92";
    case "#455A64":
      return "#263238";
    default:
      return "#263238";
  }
};

// Formatiing helper
const formatNumber = (num) => {
  if (num >= 1_000_000_000) {
    return (num / 1_000_000_000).toFixed(2) + "B";
  } else if (num >= 1_000_000) {
    return (num / 1_000_000).toFixed(2) + "M";
  } else if (num >= 1_000) {
    return (num / 1_000).toFixed(2) + "K";
  } else {
    return num.toString();
  }
};

const cardColor = ref(getRandomColor());
const tagColor = computed(() => getDarkerVersion(cardColor.value));

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

// Watch for displayState changes
watch(
  () => props.node._displayState?.isOpen,
  (newVal, oldVal) => {
    nextTick(() => {
      if (newVal) {
        drawLines();
      } else {
        clearLines();
      }
    });
  }
);

// Watch immediate children's open states
watch(
  () => (props.node.children || []).map((child) => child._displayState?.isOpen),
  (newValues, oldValues) => {
    nextTick(() => {
      // Whenever any immediate child opens/closes
      if (props.node._displayState?.isOpen) {
        drawLines();
      }
    });
  },
  { deep: true }
);

// On mount draw lines
onMounted(() => {
  nextTick(() => {
    if (props.node._displayState?.isOpen) {
      drawLines();
    }
    window.addEventListener("resize", handleResize);
  });
});

// Clean up on unmount
onBeforeUnmount(() => {
  clearLines();
  window.removeEventListener("resize", handleResize);
});

// Handle window resize
function handleResize() {
  nextTick(() => {
    if (props.node._displayState?.isOpen) {
      drawLines();
    }
  });
}
</script>

<template>
  <div class="flex flex-col gap-6 items-center">
    <div
      :data-key="props.node.data['Employee Id'] + 'start'"
      class="starter flex justify-center relative"
    >
      <div
        class="p-4 pt-6 h-10 rounded-lg shadow-md hover:shadow-lg transition-shadow duration-300 cursor-pointer flex flex-col w-64 min-h-fit bg-opacity-75"
        :style="[indentStyle, { backgroundColor: cardColor }]"
        @click="toggleChildren"
      >
        <img
          :src="node.data.Photo"
          alt="Profile"
          class="connect w-12 h-12 rounded-full object-cover absolute -top-6 left-1/2 transform -translate-x-1/2 border-4 border-white shadow-md"
        />
        <div class="pt-2 text-white font-semibold text-lg text-center">
          {{ node.data.Name }}
        </div>

        <div class="text-white text-sm text-center opacity-80">
          {{ node.data["Job Title"] }}
        </div>

        <div
          class="pt-6 h-60 flex flex-wrap gap-2 justify-center items-center overflow-scroll"
        >
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
          <span class="text-center text-gray-300 mr-1 shrink-0">
            {{ toggleIcon }} ({{ node.children ? node.children.length : 0 }}/{{
              node.data.recursiveChildren
            }})
          </span>
        </div>
      </div>
    </div>
    <!-- Recursively render children -->
    <div
      v-if="hasChildren && node._displayState.isOpen"
      class="transition-all duration-300 ease-in-out mt-16 w-fit flex flex-row flex-nowrap"
    >
      <TreeNode
        class="tree-node"
        v-for="child in node.children"
        :key="child.data['Employee Id']"
        :node="child"
        :data-key="child.data['Employee Id']"
      />
    </div>
  </div>
</template>

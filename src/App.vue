<script>
import * as d3 from "https://cdn.jsdelivr.net/npm/d3@7/+esm";
import { reactive } from "vue";
import TreeNode from "./components/TreeNode.vue";

export default {
  name: "D3 heirarchy",

  components: {
    TreeNode,
  },

  data() {
    return {
      rootNode: null,
    };
  },

  methods: {
    pullData() {
      fetch("/gigacorp.csv")
        .then((response) => {
          if (!response.ok) {
            throw new Error("File does not exist!");
          }
          return response.text();
        })
        .then((csvText) => {
          const parsedData = d3.csvParse(csvText);
          const stratify = d3
            .stratify()
            .id((d) => d["Employee Id"]) // Use "Employee Id" as the node identifier
            .parentId((d) => d["Manager"]); // Use "Manager" as the parent reference

          // Generate the hierarchy from the parsed data
          const rootNode = stratify(parsedData);
          this.rootNode = rootNode;
          this.processData();
        })
        .catch((error) => {
          console.error("Error fetching or processing the CSV data:", error);
        });
    },
    replaceNameAfterQuestionMark(s, shortenedName) {
      const questionMarkIndex = s.indexOf("?");
      if (questionMarkIndex === -1) return s;

      const afterQuestionMark = s.slice(questionMarkIndex + 1);

      const nameRegex = /name=([^&]*)/;
      const match = afterQuestionMark.match(nameRegex);

      if (!match) return s;

      const newAfterQuestionMark = afterQuestionMark.replace(
        nameRegex,
        `name=${shortenedName}`
      );
      return s.slice(0, questionMarkIndex + 1) + newAfterQuestionMark;
    },
    processData() {
      if (!this.rootNode) return;

      this.rootNode.eachAfter((node) => {
        let managerCost = 0;
        let individualCost = 0;
        let recursiveChildren = 0;

        // Only process children's costs
        if (node.children) {
          node.children.forEach((child) => {
            if (child.children && child.children.length > 0) {
              // If child is a manager, add to manager cost
              managerCost += Number(child.data.Salary) || 0;
            } else {
              // If child is a leaf (individual contributor), add to individual cost
              individualCost += Number(child.data.Salary) || 0;
            }
            managerCost += child.data.managerCost; // recursive cost
            individualCost += child.data.individualCost; // recursive cost
            recursiveChildren += 1 + child.data.recursiveChildren;
          });
        }

        // Store calculated values
        node.data.managerCost = managerCost;
        node.data.individualCost = individualCost;
        node.data.managementCostRatio = managerCost
          ? managerCost / individualCost
          : 0;
        node.data.totalCost = managerCost + individualCost;
        node.data.recursiveChildren = recursiveChildren;

        // Correct link for photo src
        let nameArray = node.data.Name.split(" ");
        let shortenedName = nameArray[0].charAt(0) + nameArray[1].charAt(0);
        node.data.Photo = this.replaceNameAfterQuestionMark(
          node.data.Photo,
          shortenedName
        );

        // reactive works on objects
        node._displayState = reactive({
          isOpen: node.depth === 0,
        });
      });
    },
  },

  mounted() {
    this.pullData();
  },
};
</script>

<template>
  <div class="p-4 font-sans">
    <h1 class="text-2xl font-bold mb-4 pb-10 top-4 text-black">
      Organization Tree
    </h1>
    <div v-if="rootNode">
      <TreeNode
        :node="rootNode"
        class="tree-node"
        :data-key="rootNode.data['Employee Id']"
        :key="rootNode.data['Employee Id']"
      />
    </div>
    <div class="text-black" v-else>Loading...</div>
  </div>
</template>

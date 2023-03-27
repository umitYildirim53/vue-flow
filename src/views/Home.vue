<script setup>
import {
  Panel,
  PanelPosition,
  VueFlow,
  isNode,
  useVueFlow,
} from "@vue-flow/core";
import { Background } from "@vue-flow/background";
import { Controls } from "@vue-flow/controls";
import { MiniMap } from "@vue-flow/minimap";
import { ref, computed, nextTick, watch } from "vue";
import useStore from "@/store.js";
import SaveRestoreControls from "@/components/SaveRestoreControls.vue";
import Sidebar from "@/components/Sidebar.vue";

/**
 * useVueFlow provides all event handlers and store properties
 * You can pass the composable an object that has the same properties as the VueFlow component props
 */

let id = 0;
const getId = () => `dndnode_${id++}`;

const store = useStore();

const dark = ref(false);

const changeTheme = (theme) => {
  dark.value = theme;
};

const {
  onPaneReady,
  onNodeDragStop,
  onConnect,
  addEdges,
  addNodes,
  setTransform,
  toObject,
  findNode,
  nodes,
  edges,
  viewport,
  project,
  vueFlowRef,
} = useVueFlow({ nodes: store.elements });

/**
 * Our elements
 */

const onDragOver = (event) => {
  event.preventDefault();

  if (event.dataTransfer) {
    event.dataTransfer.dropEffect = "move";
  }
};

onConnect((params) => addEdges([params]));

const onDrop = (event) => {
  const type = event.dataTransfer?.getData("application/vueflow");

  const { left, top } = vueFlowRef.value.getBoundingClientRect();

  const position = project({
    x: event.clientX - left,
    y: event.clientY - top,
  });

  const newNode = {
    id: getId(),
    type,
    position,
    label: `${type} node`,
  };

  addNodes([newNode]);

  // align node position after drop, so it's centered to the mouse
  nextTick(() => {
    const node = findNode(newNode.id);
    const stop = watch(
      () => node.dimensions,
      (dimensions) => {
        if (dimensions.width > 0 && dimensions.height > 0) {
          node.position = {
            x: node.position.x - node.dimensions.width / 2,
            y: node.position.y - node.dimensions.height / 2,
          };
          stop();
        }
      },
      { deep: true, flush: "post" }
    );
  });
};

/**
 * This is a Vue Flow event-hook which can be listened to from anywhere you call the composable, instead of only on the main component
 *
 * onPaneReady is called when viewpane & nodes have visible dimensions
 */
onPaneReady(({ fitView }) => {
  fitView();
});

onNodeDragStop((e) => console.log("drag stop", e));

/**
 * onConnect is called when a new connection is created.
 * You can add additional properties to your new edge (like a type or label) or block the creation altogether
 */
onConnect((params) => addEdges([params]));

/**
 * To update node properties you can simply use your elements v-model and mutate the elements directly
 * Changes should always be reflected on the graph reactively, without the need to overwrite the elements
 */
</script>

<template>
  <div class="dndflow" @drop="onDrop">
    <VueFlow
      v-model="store.elements"
      :class="{ dark }"
      class="basicflow"
      :default-viewport="{ zoom: 1.5 }"
      :min-zoom="0.2"
      :max-zoom="4"
      :fit-view-on-init="true"
      @dragover="onDragOver"
    >
      <Background :pattern-color="dark ? '#FFFFFB' : '#aaa'" gap="8" />
      <MiniMap />
      <Controls />
      <SaveRestoreControls @theme="changeTheme($event)" />
    </VueFlow>
    <Sidebar />
  </div>
</template>
<style></style>

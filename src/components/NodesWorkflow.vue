<template>
  <canvas ref="canvas"></canvas>
</template>

<script>
import { fabric } from 'fabric';
import { renderIcon } from '../utils.js';
import { arrowIcon, deleteIcon } from '../icons.js';

const SEARCH_FIELD_HEIGHT = 80;
const NODE_WIDTH = 100;
const NODE_HEIGHT = 100;
let nodeCanvas = null;

// improve: config svg-loader, import and use svg images
const deleteImg = document.createElement('img');
deleteImg.src = deleteIcon;
const arrowImg = document.createElement('img');
arrowImg.src = arrowIcon;

export default {
  name: 'NodesWorkflow',
  props: {
    activeNodes: Array,
    onDeleteNode: Function,
  },
  watch: {
    // improve: create function which will render nodes from the server
    activeNodes(newVal, oldVal) {
      if (newVal?.length > oldVal?.length) {
        this.createNode(newVal[newVal.length - 1]);
      }
    },
  },
  mounted() {
    nodeCanvas = new fabric.Canvas(this.$refs.canvas, {
      isDrawingMode: false,
      width: window.innerWidth,
      height: window.innerHeight - SEARCH_FIELD_HEIGHT,
      selection: false,
    });
    nodeCanvas.on({
      'object:moving': this.onNodeMoving,
    });
  },
  // improve: add correct formulas and don't use "magic" numbers (left, top, x1, y1, etc..)
  methods: {
    createNode(node) {
      const rect = new fabric.Rect({
        fill: node.color,
        width: NODE_WIDTH,
        height: NODE_HEIGHT,
        originX: 'center',
        originY: 'center',
      });

      const text = new fabric.Text(node.label, {
        fontFamily: 'Avenir',
        fontSize: 12,
        originX: 'center',
        originY: 'center',
      });

      const group = new fabric.Group([rect, text], {
        left: 100,
        top: 50,
        width: NODE_WIDTH,
        height: NODE_HEIGHT,
        id: node.id,
        controls: {
          deleteControl: new fabric.Control({
            x: -1.3,
            y: -0.4,
            offsetY: -20,
            offsetX: 60,
            cursorStyle: 'pointer',
            mouseUpHandler: (event, transform) =>
              this.deleteNode(event, transform, node),
            render: renderIcon(deleteImg),
            cornerSize: 20,
          }),
          createArrowControl: new fabric.Control({
            x: 1,
            y: -1,
            offsetY: 95,
            offsetX: -50,
            cursorStyle: 'pointer',
            mouseUpHandler: this.drawArrow,
            render: renderIcon(arrowImg),
            cornerSize: 24,
          }),
        },
      });

      nodeCanvas.add(group);
      nodeCanvas.setActiveObject(group);
    },

    deleteNode(eventData, transform, node) {
      const activeNode = nodeCanvas.getActiveObject();
      const inputLine = activeNode.get('input');
      const outputLine = activeNode.get('output');

      nodeCanvas.remove(activeNode, inputLine, outputLine);
      nodeCanvas.requestRenderAll();

      this.onDeleteNode(node.id);
    },

    drawArrow(event, target, left, top) {
      const activeNode = nodeCanvas.getActiveObject();
      const currentOutput = activeNode.get('output');

      if (currentOutput) {
        nodeCanvas.remove(currentOutput);
      }

      // improve: draw correct an arrow
      const newLine = new fabric.Line([0, 0, 0, 0], {
        left: left,
        top: top,
        stroke: 'black',
      });

      activeNode.set('output', newLine);

      nodeCanvas.add(newLine);
      nodeCanvas.sendToBack(newLine);

      nodeCanvas.on('mouse:move', function (event) {
        newLine.set({
          x1: left,
          y1: top,
          x2: event.e.layerX,
          y2: event.e.layerY,
        });

        nodeCanvas.renderAll();
      });

      nodeCanvas.on('mouse:down', function (event) {
        if (event.target) {
          const activeNode = nodeCanvas.getActiveObject();
          const currentInputLine = activeNode.get('input');
          const currentOutputLine = activeNode.get('output');

          if (currentInputLine) {
            nodeCanvas.remove(currentInputLine);
          }

          newLine.set({
            x2: event.target.left,
            y2: event.target.top + NODE_HEIGHT / 2,
          });

          activeNode.set('input', newLine);

          nodeCanvas.renderAll();
        } else {
          nodeCanvas.remove(newLine);
        }

        nodeCanvas.off('mouse:move');
        nodeCanvas.off('mouse:down');
      });
    },

    onNodeMoving(event) {
      const activeNode = nodeCanvas.getActiveObject();
      const activeInput = activeNode.get('input');
      const activeOutput = activeNode.get('output');

      if (activeOutput) {
        activeOutput.set({
          x1: activeNode.left,
          y1: activeNode.top + NODE_HEIGHT / 2 + 10,
        });
      }

      if (activeInput) {
        activeInput.set({
          x2: activeNode.left,
          y2: activeNode.top + NODE_HEIGHT / 2,
        });
      }
    },
  },
};
</script>

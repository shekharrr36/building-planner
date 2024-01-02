<template>
  <v-stage ref="stage" :config="stageSize" @mousedown="handleStageMouseDown" @touchstart="handleStageMouseDown"
    @mousemove="handleMouseMove" @mouseUp="handleMouseUp">
    <v-layer ref="layer">
      <v-rect v-for="item in rectangles" :key="item.id" :config="item" @transformend="handleTransformEnd" />
      <v-transformer ref="transformer" />
    </v-layer>
  </v-stage>
</template>

<script>
import Konva from 'konva';
const width = window.innerWidth;
const height = window.innerHeight;

export default {
  data() {
    return {
      stageSize: {
        width: width,
        height: height,
      },
      rectangles: [
        {
          rotation: 0,
          x: 10,
          y: 10,
          width: 100,
          height: 100,
          scaleX: 1,
          scaleY: 1,
          fill: 'red',
          name: 'rect1',
          draggable: true,
        },
        {
          rotation: 0,
          x: 150,
          y: 150,
          width: 100,
          height: 100,
          scaleX: 1,
          scaleY: 1,
          fill: 'green',
          name: 'rect2',
          draggable: true,
        },
      ],
      isDrawing: false,
      selectedShapeName: '',
    };
  },
  methods: {
    handleTransformEnd(e) {
      // shape is transformed, let us save new attrs back to the node
      // find element in our state
      const rect = this.rectangles.find(
        (r) => r.name === this.selectedShapeName
      );
      // update the state
      rect.x = e.target.x();
      rect.y = e.target.y();
      rect.rotation = e.target.rotation();
      rect.scaleX = e.target.scaleX();
      rect.scaleY = e.target.scaleY();

      // change fill
      rect.fill = Konva.Util.getRandomColor();
    },
    handleStageMouseDown(e) {
      // console.log('drawing ')
      // clicked on stage - clear selection
      if (e.target === e.target.getStage()) {

        this.selectedShapeName = '';
        this.updateTransformer();

        this.isDrawing = true


        const pos = this.$refs.stage.getNode().getPointerPosition();
        // if (this.currentShape.type === "rectangle") {
        this.rectangles.push({
          rotation: 0,
          scaleX: 1,
          scaleY: 1,
          fill: 'green',
          name: 'rect3',
          draggable: true, ...pos
        })
        console.log('rectagljldsf')
        // } else if (this.currentShape.type === "circle") {
        //   this.circles.push({ ...this.currentShape, ...pos })
        // } else if (this.currentShape.type.includes("line")) {
        //   let newLine = { ...this.currentShape, points: [pos.x, pos.y] }
        //   this.lines.push(newLine)
        // } else if (this.currentShape.type === "ellipse") {
        //   this.ellipses.push({ ...this.currentShape, ...pos })
        // } else if (this.currentShape.type.includes("wedge")) {
        //   this.wedges.push({ ...this.currentShape, ...pos })
        // }

        return;
      }

      // clicked on transformer - do nothing
      const clickedOnTransformer =
        e.target.getParent().className === 'Transformer';
      if (clickedOnTransformer) {
        return;
      }

      // find clicked rect by its name
      const name = e.target.name();
      const rect = this.rectangles.find((r) => r.name === name);
      if (rect) {
        this.selectedShapeName = name;
      } else {
        this.selectedShapeName = '';
      }
      this.updateTransformer();
    },
    handleMouseMove(event) {
      // no drawing - skipping
      // if (!this.isDrawing || this.selectShape.value || !this.currentShape.type) {
      // if (!this.isDrawing || !this.currentShape.type) {
      console.log('moving...')
      if (!this.isDrawing) {
        // console.log('drawing not set')
        return;
      }
      const point = this.$refs.stage.getNode().getPointerPosition();
      // if (this.currentShape.type === "rectangle") {
      let curRec = this.rectangles[this.rectangles.length - 1];
      curRec.width = Math.abs(point.x - curRec.x)
      curRec.height = Math.abs(point.y - curRec.y)
      // } else if (this.currentShape.type === "circle") {
      //   let curCircle = this.circles[this.circles.length - 1]
      //   curCircle.radius = Math.abs(point.x - curCircle.x)
      // } else if (this.currentShape.type === "line") {
      //   let curLine = this.lines[this.lines.length - 1]
      //   curLine.points = [...curLine.points, point.x, point.y]
      // } else if (this.currentShape.type === "dashed-line") {
      //   let curDashedLine = this.lines[this.lines.length - 1]
      //   curDashedLine.points = [...curDashedLine.points, point.x, point.y]
      //   curDashedLine.dash = [33, 10]
      //   curDashedLine.tension = 0.5
      // } else if (this.currentShape.type === "dashed-dotted-line") {
      //   let curDashedDottedLine = this.lines[this.lines.length - 1]
      //   curDashedDottedLine.points = [...curDashedDottedLine.points, point.x, point.y]
      //   curDashedDottedLine.dash = [29, 20, 0.001, 20]
      //   curDashedDottedLine.tension = 0.7
      // } else if (this.currentShape.type === "ellipse") {
      //   let curEllipse = this.ellipses[this.ellipses.length - 1]
      //   curEllipse.radiusX = Math.abs(point.x - curEllipse.x)
      //   curEllipse.radiusY = Math.abs(point.y - curEllipse.y)
      // } else if (this.currentShape.type === "wedge60") {
      //   let curWedge = this.wedges[this.wedges.length - 1]
      //   curWedge.radius = Math.abs(point.x - curWedge.x)
      //   curWedge.angle = 60
      // } else if (this.currentShape.type === "wedge180") {
      //   let curWedge = this.wedges[this.wedges.length - 1]
      //   curWedge.radius = Math.abs(point.x - curWedge.x)
      //   curWedge.angle = 180
      // }
    },
    handleMouseUp() {
      console.log('up')
      this.isDrawing = false;
    },
    updateTransformer() {
      // here we need to manually attach or detach Transformer node
      const transformerNode = this.$refs.transformer.getNode();
      const stage = transformerNode.getStage();
      const { selectedShapeName } = this;

      const selectedNode = stage.findOne('.' + selectedShapeName);
      // do nothing if selected node is already attached
      if (selectedNode === transformerNode.node()) {
        return;
      }

      if (selectedNode) {
        // attach to another node
        transformerNode.nodes([selectedNode]);
      } else {
        // remove transformer
        transformerNode.nodes([]);
      }
    },
  },
};
</script>

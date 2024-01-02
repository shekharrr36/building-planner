<template>
  <div>
    <select v-model="currentShape.type">
      <option value="">Please select one</option>
      <option value="rectangle">rectangle</option>
      <option value="circle">circle</option>
      <option value="line">line</option>
      <option value="dashed-line">dashed-line</option>
      <option value="dashed-dotted-line">dashed-dotted-line</option>
      <option value="ellipse">ellipse</option>
      <option value="wedge60">wedge 60</option>
      <option value="wedge180">wedge 180</option>
      <!-- <option v-for="type in types" v-bind:value="type.value">{{ type.value }}</option> -->
    </select>

    <button @click="enableSelectTool" :style="selectShape.value ? 'background-color: yellow;' : ''">Select
      Tool</button>
    <button @click="removeShape" style="background-color: red">Remove</button>
    <button @click="toggleAnnotations" :style="isAnnotating ? 'background-color: blue;' : ''">Toggle
      Annotations</button>
  </div>
  <div>
    <v-stage :config="stageConfig" ref="stage" @mousemove="handleMouseMove" @mouseDown="handleMouseDown"
      @touchstart="handleMouseDown" @mouseUp="handleMouseUp">
      <v-layer ref="layer">
        <v-text ref="text" :config="{
          x: 10,
          y: 10,
          fontSize: 20,
          text: text,
          fill: 'black',
        }" />
        <!-- shapes here -->
        <v-circle v-for="(cir, index) in circles" :key="index" :draggable="selectShape.value"
          @transformend="handleTransformEnd($event, 'circles', index)" :name="cir.name" :config="{
            ...cir
          }" />

        <v-ellipse v-for="(ell, index) in ellipses" :key="index" :draggable="selectShape.value"
          @transformend="handleTransformEnd($event, 'ellipses', index)" :name="ell.name" :config="{
            ...ell
          }" />

        <v-rect v-for="(rec, index) in rectangles" :key="index" :draggable="selectShape.value"
          @mouseDown="handleSelectAnnotations($event, 'rectangles', index)"
          @transformend="handleTransformEnd($event, 'rectangles', index)" :name="rec.name" :config="{ ...rec }" />

        <v-line v-for="(line, index) in lines" :key="index" :draggable="selectShape.value"
          @transformend="handleTransformEnd($event, 'lines', index)" :name="line.name" :config="{
            ...line
          }" />

        <v-wedge v-for="(wedge, index) in wedges" :key="index" :draggable="selectShape.value"
          @transformend="handleTransformEnd($event, 'wedges', index)" :name="wedge.name" :config="{
            ...wedge
          }" />

        <v-transformer ref="transformer" />
      </v-layer>
    </v-stage>
  </div>
</template>

<script>
import { VStage, VLayer, VRect, VText } from 'vue-konva';
import Konva from 'konva';

const width = window.innerWidth;
const height = window.innerHeight;

export default {
  data() {
    return {
      stageConfig: {
        width: width,
        height: height,
      },
      shapes: [
        {
          "id": "1",
          "value": "rectangle"
        },
        {
          "id": "2",
          "value": "circle"
        },
        {
          "id": "3",
          "value": "line"
        },
        {
          "id": "4",
          "value": "dashed line"
        }
      ],
      text: "Try to draw some shapes",
      isDrawing: false,
      isDragging: false,
      rectangles: [],
      circles: [],
      lines: [],
      ellipses: [],
      wedges: [],
      currentShape: {
        fill: 'rgb(0,0,0,0)',
        stroke: 'black',
        strokeWidth: 3,
        // draggable: true,
        lineCap: 'round',
        lineJoin: 'round',
        type: '',
        show: true,
        annotated: false,
        state: 'show'
      },
      isAnnotating: false,
      selectShape: {
        value: false,
        type: '',
        index: -1,
        name: '',
      },
      annotations: []
    };
  },
  methods: {
    // 1. Draw tools → lines, rectangles, circles & other shapes which let the user create their
    // masterpiece.
    // 2. Select tool → Allows the created shapes to be moved, resized or deleted.
    // 3. View tool → Show or hide annotations.
    // Implement methods for other shapes
    enableSelectTool() {
      // Implement logic to enable the select tool
      if (this.selectShape.value) {
        this.selectShape.type = ''
        this.selectShape.index = -1
        this.selectShape.value = false
      } else {
        this.selectShape.value = true
      }
    },
    handleSelectAnnotations(e, shapeName, index) {
      console.log('select annotations')
      const shape = this?.[shapeName]?.[index]
      if (this.isAnnotating) {
        const shapeInAnnotation = this.annotations.find(i => i.name === `${shapeName}-${index}`)
        if (shapeInAnnotation) {
          shape.stroke = "black"
          shape.state = "show"
        } else {
          shape.stroke = "gray"
          shape.state = "hide"
          this.annotations.push(shape)
        }

        // if (shape.state === "show") {
        //   shape.state = "hide"
        // } else {
        //   shape.state = 'show'
        // }
        // this.annotations.push(shape)
        console.log('annotatinons', this.annotations)
      } else {
        console.log('not annotating')
      }
    },
    toggleAnnotations() {
      // Implement logic to show/hide annotations
      // toggle annotations to show/ hide

      if (this.isAnnotating) {
        console.log('band kro')
        for (let i = 0; i < this.annotations.length; i++) {
          const annotation = this.annotations[i];
          const [shape, index] = annotation.name.split('-')


          console.log('ye band ho rha h', annotation, shape, index)
          if (annotation.state === "hide") {
            const shapeAlreadyInAnnotation = this?.[shape].find(i => i.name === `${shape}-${index}`)
            console.log('shapeAlready', shapeAlreadyInAnnotation)
            if (shapeAlreadyInAnnotation) {
              console.log('select hue ko chupao', annotation, shape, index)
              this?.[shape].splice(index, 1)
            }
          }
          // else if (annotation.state === 'show') {
          //   this?.[shape].push(annotation)
          // }
        }
        this.isAnnotating = false
      } else {
        console.log('chalu karo')
        for (let i = 0; i < this.annotations.length; i++) {
          const annotation = this.annotations[i];
          const [shape, index] = annotation.name.split('-')
          if (annotation.state === "hide") {
            console.log('chupe hue ko dikhao', shape, index)
            // this?.[shape].splice(index, 1)
            this?.[shape].push(annotation)
          }
          // else if (annotation.state === 'show') {
          //   console.log('kya ya dikh gaya', annotation)
          //   this?.[shape].push(annotation)
          // }
        }
        this.isAnnotating = true
      }
      console.log('current annotations', this.annotations)
    },
    handleMouseDown(e) {
      // if (!this.currentShape.type || this.selectShape.value) {
      if (!this.currentShape.type) {
        console.log('please select shape first to draw')
        return;
      }
      if (this.isAnnotating) {
        console.log('annotating')
        return
      }

      if (!this.selectShape.value || e.target === e.target.getStage()) {
        // clicked on the plane
        if (this.selectShape.value) {

          this.selectShape.name = '';
          this.selectShape.value = false
          this.updateTransformer();
          return
        }
        // draw
        this.isDrawing = true;

        const pos = this.$refs.stage.getNode().getPointerPosition();
        if (this.currentShape.type === "rectangle") {
          this.rectangles.push({ ...this.currentShape, name: 'rectangles-' + this.rectangles.length, ...pos })
        } else if (this.currentShape.type === "circle") {
          this.circles.push({ ...this.currentShape, name: 'circles-' + this.circles.length, ...pos })
        } else if (this.currentShape.type.includes("line")) {
          let newLine = { ...this.currentShape, name: 'lines-' + this.lines.length, points: [pos.x, pos.y] }
          this.lines.push(newLine)
        } else if (this.currentShape.type === "ellipse") {
          this.ellipses.push({ ...this.currentShape, name: 'ellipses-' + this.ellipses.length, ...pos })
        } else if (this.currentShape.type.includes("wedge")) {
          this.wedges.push({ ...this.currentShape, name: 'wedges-' + this.wedges.length, ...pos })
        }
        return
      } else {

        // clicked on transformer do nothing
        const clickedOnTransformer =
          e.target.getParent().className === 'Transformer';
        if (clickedOnTransformer) {
          return;
        }

        // select shape
        if (this.selectShape.value) {
          const name = e.target.name();
          this.selectShape.name = name
          this.selectShape.type = name.split('-')[0]
          this.updateTransformer();
        }
      }
    },
    handleMouseUp() {
      this.isDrawing = false;
    },
    handleMouseMove(event) {
      // no drawing - skipping
      // if (!this.isDrawing || this.selectShape.value || !this.currentShape.type) {
      if (!this.isDrawing || !this.currentShape.type) {
        return;
      }
      const point = this.$refs.stage.getNode().getPointerPosition();
      if (this.currentShape.type === "rectangle") {
        let curRec = this.rectangles[this.rectangles.length - 1];
        curRec.width = Math.abs(point.x - curRec.x)
        curRec.height = Math.abs(point.y - curRec.y)
      } else if (this.currentShape.type === "circle") {
        let curCircle = this.circles[this.circles.length - 1]
        curCircle.radius = Math.abs(point.x - curCircle.x)
      } else if (this.currentShape.type === "line") {
        let curLine = this.lines[this.lines.length - 1]
        curLine.points = [...curLine.points, point.x, point.y]
      } else if (this.currentShape.type === "dashed-line") {
        let curDashedLine = this.lines[this.lines.length - 1]
        curDashedLine.points = [...curDashedLine.points, point.x, point.y]
        curDashedLine.dash = [33, 10]
        curDashedLine.tension = 0.5
      } else if (this.currentShape.type === "dashed-dotted-line") {
        let curDashedDottedLine = this.lines[this.lines.length - 1]
        curDashedDottedLine.points = [...curDashedDottedLine.points, point.x, point.y]
        curDashedDottedLine.dash = [29, 20, 0.001, 20]
        curDashedDottedLine.tension = 0.7
      } else if (this.currentShape.type === "ellipse") {
        let curEllipse = this.ellipses[this.ellipses.length - 1]
        curEllipse.radiusX = Math.abs(point.x - curEllipse.x)
        curEllipse.radiusY = Math.abs(point.y - curEllipse.y)
      } else if (this.currentShape.type === "wedge60") {
        let curWedge = this.wedges[this.wedges.length - 1]
        curWedge.radius = Math.abs(point.x - curWedge.x)
        curWedge.angle = 60
      } else if (this.currentShape.type === "wedge180") {
        let curWedge = this.wedges[this.wedges.length - 1]
        curWedge.radius = Math.abs(point.x - curWedge.x)
        curWedge.angle = 180
      }
    },
    removeShape() {
      if (this.selectShape?.name) {
        const [shape, index] = this.selectShape.name.split('-')
        this[shape].splice(index, 1)

        this.selectShape.name = ""
        this.selectShape.type = ''
        this.selectShape.index = -1

        this.updateTransformer();
      } else {
        console.log('please select shape first to remove')
      }
    },
    handleTransformEnd(e, shapeName, index) {
      const shape = this?.[shapeName]?.[index]
      // update the state
      shape.x = e.target?.x();
      shape.y = e.target?.y();
      shape.rotation = e.target?.rotation();
      shape.scaleX = e.target?.scaleX();
      shape.scaleY = e.target?.scaleY();
      shape.strokeWidth = 3
    },
    updateTransformer() {
      const transformerNode = this.$refs.transformer.getNode();
      const stage = transformerNode.getStage();
      const { name } = this.selectShape;


      const selectedNode = stage.findOne('.' + name);
      if (selectedNode === transformerNode.node()) {
        return;
      }

      if (nodes) {
        transformerNode.nodes([...nodes]);
      } else {
        transformerNode.nodes([]);
      }
    }
  },
  components: {
    VStage,
    VLayer,
    VRect,
  }
}
</script>

<style scoped>
/* Add styling as needed */
</style>

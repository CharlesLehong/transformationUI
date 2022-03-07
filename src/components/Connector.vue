<template>
  <g @mouseover="handleMouseOver" @mouseleave="handleMouseLeave">
    <path :d="dAttr" :style="pathStyle"></path>
    <a v-if="show.delete" @click="deleteConnector">
      <text text-anchor="middle" :transform="arrowTransform" font-size="44"
        >&times;</text
      >
    </a>
  </g>
</template>

<script>
export default {
  name: "FieldConnector",
  props: {
    start: {
      type: Array,
      default() {
        return [0, 0];
      },
    },
    end: {
      type: Array,
      default() {
        return [0, 0];
      },
    },
    id: Number,
  },
  data() {
    return {
      show: {
        delete: false,
      },
    };
  },
  computed: {
    pathStyle() {
      return {
        stroke: "rgb(12, 156, 173)",
        strokeWidth: 2.73205,
        fill: "none",
      };
    },
    arrowStyle() {
      return {
        stroke: "rgb(12, 156, 173)",
        strokeWidth: 5.73205,
        fill: "none",
      };
    },
    arrowTransform() {
      const [arrowX, arrowY] = this.caculateCenterPoint();
      const degree = this.caculateRotation();
      return `translate(${arrowX}, ${arrowY}) rotate(${degree})`;
    },
    dAttr() {
      let x1 = this.start[0],
        y1 = this.start[1],
        x2 = this.end[0],
        y2 = this.end[1];
      let dist = this.distance(x1, y1, x2, y2) * 0.25;

      //let x1 = cx, y1 = cy + 50, x2 = ex, y2 = ey - 50;
      return `M ${x1}, ${y1} C ${x1 + dist}, ${y1}, ${
        x2 - dist
      }, ${y2}, ${x2}, ${y2}`;
    },
  },
  methods: {
    handleMouseOver() {
      if (this.id) {
        this.show.delete = true;
      }
    },
    handleMouseLeave() {
      this.show.delete = false;
    },
    caculateCenterPoint() {
      const dx = (this.end[0] - this.start[0]) / 2;
      const dy = (this.end[1] - this.start[1]) / 2;
      return [this.start[0] + dx, this.start[1] + dy];
    },
    distance(x1, y1, x2, y2) {
      return Math.sqrt((x2 - x1) * (x2 - x1) + (y2 - y1) * (y2 - y1));
    },
    caculateRotation() {
      const angle = -Math.atan2(
        this.end[0] - this.start[0],
        this.end[1] - this.start[1]
      );
      const degree = (angle * 180) / Math.PI;
      return degree < 0 ? degree + 360 : degree;
    },
    deleteConnector() {
      this.$emit("deleteConnector");
    },
  },
};
</script>

<style scoped lang="scss">
g {
  cursor: pointer;
}
</style>

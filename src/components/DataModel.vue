<template>
  <div class="data-model" :style="nodeStyle">
    <div
      v-for="(inputField, index) in inputFields"
      :key="index"
      class="data-model-port data-model-input"
      :class="{ 'data-model-port-active': portIsActive(index, false) }"
      :style="portStyle(index)"
      @mousedown="inputMouseDown"
      @mouseup="inputMouseUp($event, index)"
    >
      <p class="field-name">
        {{ inputField.name + ":" + inputField.dataType }}
      </p>
    </div>
    <div class="data-model-name">
      {{ name }}
    </div>
    <div
      v-for="(outputField, index) in outputFields"
      :key="index"
      class="data-model-port data-model-output"
      :class="{ 'data-model-port-active': portIsActive(index) }"
      :style="portStyle(index)"
      @mousedown="outputMouseDown($event, index)"
    >
      <p class="field-name">
        {{ outputField.name + ":" + outputField.dataType }}
      </p>
    </div>
  </div>
</template>
<script>
export default {
  name: "DataModel",
  props: {
    x: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === "number";
      },
    },
    y: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === "number";
      },
    },
    name: {
      type: String,
      default: "input name",
    },
    outputFields: {
      type: Array,
      default() {
        return [];
      },
    },
    inputFields: {
      type: Array,
      default() {
        return [];
      },
    },
    options: {
      type: Object,
      default() {
        return {
          centerX: 0,
          scale: 1,
          centerY: 0,
        };
      },
    },
    connectors: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  data() {
    return {
      hover: false,
      modelType: "",
      port: -1,
      functionIOHovered: false,
      show: {
        delete: false,
      },
    };
  },
  computed: {
    nodeStyle() {
      let largestNumberOfFields = Math.max(
        this.inputFields.length,
        this.outputFields.length
      );
      return {
        top: this.options.centerY + this.y * this.options.scale + "px",
        left: this.options.centerX + this.x * this.options.scale + "px",
        transform: `scale(${this.options.scale})`,
        height: 72 + 60 * (largestNumberOfFields + 1) + "px",
        minHeight: "200px",
        width: "350px",
      };
    },
    portIsActive() {
      return function (portNumber, source = true) {
        let connector = null;

        if (source)
          connector = this.connectors.find((c) => c.sourcePortId == portNumber);
        else
          connector = this.connectors.find((c) => c.targetPortId == portNumber);

        if (connector) return true;
        return false;
      };
    },
  },
  methods: {
    handleMousedown(e) {
      const target = e.target || e.srcElement;
      if (
        target.className.indexOf("data-model-input") < 0 &&
        target.className.indexOf("data-model-output") < 0
      ) {
        this.$emit("nodeSelected", e);
      }
      e.preventDefault();
    },
    outputMouseDown(e, sourcePortId) {
      this.$emit("connectionStart", sourcePortId);
      e.preventDefault();
    },
    inputMouseDown(e) {
      e.preventDefault();
    },
    inputMouseUp(e, targetPortId) {
      this.$emit("connectionStop", targetPortId);
      e.preventDefault();
    },
    portStyle(index) {
      return {
        top: 130 + index * 60 + "px",
      };
    },

    shortenLabel(name) {
      if (name.length > 30) return name.substr(0, 27) + "...";
      return name;
    },
  },
};
</script>
<style scoped>
.data-model {
  margin: 0;
  width: 350px;
  position: absolute;
  box-sizing: border-box;
  border: none;
  border-radius: 2px;
  background: #f5f5f5;
  z-index: 1;
  opacity: 0.9;
  transform-origin: top left;
  box-shadow: 0 0 0 2px #0c9cad;
}
.data-model-name {
  background: #0c9cad;
  text-align: center;
  max-height: 90px;
  min-height: 90px;
  font-size: 20px;
  font-weight: bold;
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
}
.data-model-port {
  position: absolute;
  width: 24px;
  height: 24px;
  transform: translate(-50%);
  border: 1px solid #ccc;
  border-radius: 100px;
  background: #f5f5f5;
  box-shadow: 0 0 0 2px #0c9cad;
}
.data-model-port-active {
  background-color: #0c9cad;
}
.data-model-port:hover {
  background-color: #0c9cad;
}
.data-model-output {
  top: 110px;
  left: 350px;
  text-align: right;
  cursor: pointer;
}
.field-name {
  font-size: 17px;
  font-weight: bolder;
}
.data-model-input .field-name {
  position: inherit;
  left: 35px;
  width: 300px;
}
.data-model-output .field-name {
  position: inherit;
  right: 35px;
  width: 300px;
}
</style>

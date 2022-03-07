<template>
  <div
    class="mapping-container"
    @mousemove="handleMove"
    :style="dynamicContainerStyle"
  >
    <svg width="100%" :height="`${height}px`">
      <connector
        v-for="(connector, index) in connectors"
        v-bind.sync="connectors[index]"
        :key="index"
        @deleteConnector="deleteConnector(connector.id)"
      />
    </svg>
    <data-model
      v-for="(dataModel, index) in localMapProperties.dataModels"
      :key="index"
      v-bind.sync="localMapProperties.dataModels[index]"
      :options="mapOptions"
      :connectors="localMapProperties.connectors"
      @connectionStart="connectionStart(dataModel.id, $event)"
      @connectionStop="connectionStop(dataModel.id, $event)"
    >
    </data-model>
  </div>
</template>

<script>
import Connector from "./Connector.vue";
import DataModel from "./DataModel.vue";
import { getMousePosition } from "../helpers/position";
import { dataModelTypes } from "./../constants/dataModelTypes";

export default {
  name: "MappigContainer",
  components: {
    Connector,
    DataModel,
  },
  props: {
    mapProprties: {
      type: Object,
      default() {
        return {
          centerX: 0,
          scale: 1,
          centerY: 0,
          dataModels: [],
          connectors: [],
        };
      },
    },
    height: {
      type: Number,
      default: 400,
    },
    screenWidth: {
      type: Number,
      default: 400,
    },
  },
  data() {
    return {
      action: {
        connecting: false,
        dragging: false,
        scrolling: false,
        selected: 0,
      },
      mouse: {
        x: 0,
        y: 0,
        lastX: 0,
        lastY: 0,
      },
      newConnector: null,
      rootDivOffset: {
        top: 0,
        left: 0,
      },
    };
  },
  computed: {
    dynamicContainerStyle() {
      return {
        width: this.screenWidth,
      };
    },
    localMapProperties: {
      get: function () {
        return this.mapProprties;
      },
      set: function (value) {
        this.$emit("updateScene", value);
      },
    },
    mapOptions() {
      return {
        centerY: this.localMapProperties.centerY,
        centerX: this.localMapProperties.centerX,
        scale: this.localMapProperties.scale,
      };
    },
    connectors() {
      const connectors = this.localMapProperties.connectors.map((connector) => {
        const sourceModel = this.getModelById(connector.sourceDataModelId);
        const targetModel = this.getModelById(connector.targetDataModelId);

        let x, y, cy, cx, ex, ey;

        x = this.localMapProperties.centerX + sourceModel.x;
        y = this.localMapProperties.centerY + sourceModel.y;
        [cx, cy] = this.getPortPosition(
          sourceModel.type,
          x,
          y,
          connector.sourcePortId
        );

        x = this.localMapProperties.centerX + targetModel.x;
        y = this.localMapProperties.centerY + targetModel.y;
        [ex, ey] = this.getPortPosition(
          targetModel.type,
          x,
          y,
          connector.targetPortId,
          true
        );

        return {
          start: [cx, cy],
          end: [ex, ey],
          id: connector.id,
        };
      });
      if (this.newConnector) {
        let x, y, cy, cx;
        const sourceModel = this.getModelById(
          this.newConnector.sourceDataModelId
        );

        x = this.localMapProperties.centerX + sourceModel.x;
        y = this.localMapProperties.centerY + sourceModel.y;
        [cx, cy] = this.getPortPosition(
          sourceModel.type,
          x,
          y,
          this.newConnector.sourcePortId
        );

        connectors.push({
          start: [cx, cy],
          end: [this.newConnector.mx, this.newConnector.my],
        });
      }

      return connectors;
    },
  },
  methods: {
    getModelById(id) {
      return this.localMapProperties.dataModels.find((model) => {
        return id === model.id;
      });
    },
    getPortPosition(type, x, y, portNumber = 0) {
      y += 142;
      if (type === dataModelTypes.Target) return [x, y + 60 * portNumber];
      if (type === dataModelTypes.Source) return [x + 350, y + 60 * portNumber];
    },
    connectionStart(sourceDataModelId, sourcePortId) {
      this.action.connecting = true;
      this.newConnector = {
        sourceDataModelId: sourceDataModelId,
        sourcePortId: sourcePortId,
        mx: 0,
        my: 0,
      };
    },
    connectionStop(targetDataModelId, targetPortId) {
      if (
        this.newConnector &&
        this.newConnector.sourceDataModelId !== targetDataModelId
      ) {
        const existingConnector = this.localMapProperties.connectors.find(
          (connector) => {
            return (
              connector.sourceDataModelId ===
                this.newConnector.sourceDataModelId &&
              connector.targetDataModelId === targetDataModelId &&
              connector.sourcePortId === this.newConnector.sourcePortId
            );
          }
        );

        if (!existingConnector) {
          let maxId = Math.max(
            0,
            ...this.localMapProperties.connectors.map((connector) => {
              return connector.id;
            })
          );

          const newLink = {
            id: maxId + 1,
            sourceDataModelId: this.newConnector.sourceDataModelId,
            sourcePortId: this.newConnector.sourcePortId,
            targetDataModelId: targetDataModelId,
            targetPortId: targetPortId,
          };
          this.localMapProperties.connectors.push(newLink);
        }
      }
      this.newConnector = null;
    },
    deleteConnector(id) {
      const deletedLink = this.localMapProperties.connectors.find(
        (connector) => {
          return connector.id === id;
        }
      );

      if (deletedLink) {
        this.localMapProperties.connectors =
          this.localMapProperties.connectors.filter((item) => {
            return item.id !== id;
          });
      }
    },
    handleMove(e) {
      if (this.action.connecting) {
        [this.mouse.x, this.mouse.y] = getMousePosition(this.$el, e);
        if (this.newConnector) {
          [this.newConnector.mx, this.newConnector.my] = [
            this.mouse.x,
            this.mouse.y,
          ];
        }
      }
    },
  },
  mounted() {
    this.rootDivOffset.top = this.$el ? this.$el.offsetTop : 0;
    this.rootDivOffset.left = this.$el ? this.$el.offsetLeft : 0;
  },
};
</script>

<style scoped lang="scss">
.mapping-container {
  margin: 0;
  background: white;
  position: relative;
  overflow: hidden;
}
</style>

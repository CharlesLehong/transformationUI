<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-card>
          <v-card-title>Transformation Map</v-card-title>
          <v-card-subtitle>The transformation map description</v-card-subtitle>
          <v-divider></v-divider>
          <v-card-actions>
            <v-btn>EDIT</v-btn>
            <v-btn>PUBLISH</v-btn>
            <v-spacer></v-spacer>
            <v-btn>DELETE</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12">
        <v-card>
          <v-card-title>Map Fields</v-card-title>
          <v-divider></v-divider>
          <v-card-text>
            <mapping-container
              :mapProprties.sync="mapProprties"
              :screenWidth="width"
              @updateMapProperties="($even) => (mapProprties = $event)"
              @addStep="addStep()"
              @nodeDelete="deleteStep()"
              :height="mapScreenHeight"
            />
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import MappingContainer from "./components/MappingContainer.vue";
import { dataModelTypes } from "./constants/dataModelTypes";

export default {
  name: "app",
  components: {
    MappingContainer,
  },
  data() {
    return {
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight,
      mapProprties: {
        centerX: 0,
        centerY: 0,
        scale: 1,
        dataModels: [
          {
            id: 1,
            x: 0,
            y: 100,
            type: dataModelTypes.Source,
            name: "Source",
            outputFields: [
              { name: "Name", dataType: "String" },
              { name: "Surname", dataType: "String" },
              { name: "Age", dataType: "Integer" },
            ],
          },
          {
            id: 2,
            x: this.targetModelXPosition,
            y: 100,
            type: dataModelTypes.Target,
            name: "Destination",
            inputFields: [
              { name: "FirstName", dataType: "String" },
              { name: "LastName", dataType: "String" },
              { name: "Age", dataType: "Integer" },
            ],
          },
        ],
        connectors: [
          {
            id: 1,
            sourceDataModelId: 1,
            sourcePortId: 0,
            targetDataModelId: 2,
            targetPortId: 0,
          },
          {
            id: 2,
            sourceDataModelId: 1,
            sourcePortId: 1,
            targetDataModelId: 2,
            targetPortId: 1,
          },
        ],
      },
    };
  },
  watch: {
    width() {
      this.setDataModelPositions();
    },
  },
  computed: {
    dataModelTypes() {
      return dataModelTypes;
    },
    mapScreenHeight() {
      let sourceFields = this.mapProprties.dataModels[0].outputFields;
      let targetFields = this.mapProprties.dataModels[1].inputFields;
      return 60 * Math.max(sourceFields.length, targetFields.length) + 340;
    },
  },
  methods: {
    getDimensions() {
      this.width = document.documentElement.clientWidth;
      this.height = document.documentElement.clientHeight;
    },
    setDataModelPositions() {
      let sourceModel = this.mapProprties.dataModels[0];
      let targetModel = this.mapProprties.dataModels[1];
      let sourcePercentage = 0.06;
      let targetPercentage = 0.3;

      if (!sourceModel || !targetModel) return;
      if (this.width > 1900) {
        targetPercentage = 0.2;
        sourcePercentage = 0.12;
      }

      sourceModel.x = this.width - this.width + this.width * sourcePercentage;
      targetModel.x = this.width - this.width * targetPercentage - 400;
    },
  },
  mounted() {
    window.addEventListener("resize", this.getDimensions);
    this.setDataModelPositions();
  },
  unmounted() {
    window.removeEventListener("resize", this.getDimensions);
  },
};
</script>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0;
  overflow: hidden;
  height: 800px;
  .tool-wrapper {
    position: relative;
  }
}
</style>

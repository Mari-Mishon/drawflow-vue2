<template>
  <div id="app">
    <div class="ma-0">
      <v-row>
        <v-col cols="2" class="column">
          <v-list>
            <v-list-item
              v-for="n in listNodes"
              :key="n.name"
              draggable="true"
              :data-node="n.item"
              @dragstart="drag($event)"
              class="drag-drawflow"
            >
              <div class="node" :style="`background: ${n.color}`">
                {{ n.name }}
              </div>
            </v-list-item>
          </v-list>
        </v-col>
        <v-col cols="10">
          <div id="drawflow" @drop="drop($event)" @dragover="allowDrop($event)">
            <div class="menu">
              <v-btn @click="homeTab" class="selected">Home</v-btn>
              <v-btn @click="otherTab">Other Module</v-btn>
            </div>
            <div class="bar-zoom">
              <v-btn @click="zoomOut">Уменьшить</v-btn>
              <v-btn @click="zoomReset">Нач.позиция</v-btn>
              <v-btn @click="zoomIn">Увеличить</v-btn>
            </div>
          </div>
        </v-col>
      </v-row>
    </div>
  </div>
</template>

<script>
/*eslint-disable */
import Drawflow from "drawflow";
import styleDrawflow from "drawflow/dist/drawflow.min.css";
import Vue from "vue";
import Node1 from "./components/nodes/node1.vue";
import Node2 from "./components/nodes/node2.vue";
import Node3 from "./components/nodes/node3.vue";

export default {
  name: "App",

  components: {
    Node1,
    Node2,
    Node3,
  },
  data() {
    return {
      editor: null,
      exportValue: null,
      mobile_item_selec: "",
      mobile_last_move: null,

      listNodes: [
        {
          name: "Get/Post",
          color: "#49494970",
          item: "Node1",
          input: 0,
          output: 1,
        },
        {
          name: "Script",
          color: "blue",
          item: "Node2",
          input: 1,
          output: 2,
        },
        {
          name: "console.log",
          color: "#ff9900",
          item: "Node3",
          input: 1,
          output: 0,
        },
      ],
    };
  },
  mounted() {
    const id = document.getElementById("drawflow");
    this.editor = new Drawflow(id, Vue, this);
    this.editor.start();
    this.editor.registerNode("Node1", Node1, {}, {});
    this.editor.registerNode("Node2", Node2, {}, {});
    this.editor.registerNode("Node3", Node3, {}, {});

    this.editor.import({
      drawflow: {
        Home: {
          data: {
            5: {
              id: 5,
              name: "Node2",
              data: { script: "(req,res) => {\n console.log(req);\n}" },
              class: "Node2",
              html: "Node2",
              typenode: "vue",
              inputs: {
                input_1: { connections: [{ node: "6", input: "output_1" }] },
              },
              outputs: {
                output_1: { connections: [] },
                output_2: { connections: [] },
              },
              pos_x: 500,
              pos_y: 117,
            },
            6: {
              id: 6,
              name: "Node1",
              data: { url: "localhost/add", method: "post" },
              class: "Node1",
              html: "Node1",
              typenode: "vue",
              inputs: {},
              outputs: {
                output_1: { connections: [{ node: "5", output: "input_1" }] },
              },
              pos_x: 137,
              pos_y: 89,
            },
          },
        },
        Other: {
          data: {
            5: {
              id: 5,
              name: "Node2",
              data: { script: "(req,res) => {\n console.log(req);\n}" },
              class: "Node2",
              html: "Node2",
              typenode: "vue",
              inputs: {
                input_1: { connections: [{ node: "6", input: "output_1" }] },
              },
              outputs: {
                output_1: { connections: [] },
                output_2: { connections: [] },
              },
              pos_x: 380,
              pos_y: 217,
            },
            6: {
              id: 6,
              name: "Node1",
              data: { url: "localhost/add", method: "post" },
              class: "Node1",
              html: "Node1",
              typenode: "vue",
              inputs: {},
              outputs: {
                output_1: { connections: [{ node: "5", output: "input_1" }] },
              },
              pos_x: 137,
              pos_y: 189,
            },
          },
        },
      },
    });
  },

  methods: {
    changeModule(event) {
      var all = document.querySelectorAll(".menu ul li");
      for (var i = 0; i < all.length; i++) {
        all[i].classList.remove("selected");
      }
      event.target.classList.add("selected");
    },

    positionMobile(ev) {
      mobile_last_move = ev;
    },
    addNodeToDrawFlow(name, pos_x, pos_y) {
      pos_x =
        pos_x *
          (this.editor.precanvas.clientWidth /
            (this.editor.precanvas.clientWidth * this.editor.zoom)) -
        this.editor.precanvas.getBoundingClientRect().x *
          (this.editor.precanvas.clientWidth /
            (this.editor.precanvas.clientWidth * this.editor.zoom));

      pos_y =
        pos_y *
          (this.editor.precanvas.clientHeight /
            (this.editor.precanvas.clientHeight * this.editor.zoom)) -
        this.editor.precanvas.getBoundingClientRect().y *
          (this.editor.precanvas.clientHeight /
            (this.editor.precanvas.clientHeight * this.editor.zoom));

      const nodeSelected = this.listNodes.find((ele) => ele.item == name);

      this.editor.addNode(
        name,
        nodeSelected.input,
        nodeSelected.output,
        pos_x,
        pos_y,
        name,
        {},
        name,
        "vue"
      );
    },

    allowDrop(e) {
      e.preventDefault();
    },

    drag(ev) {
      if (ev.type === "touchstart") {
        mobile_item_selec = ev.target
          .closest(".drag-drawflow")
          .getAttribute("data-node");
      } else {
        ev.dataTransfer.setData("node", ev.target.getAttribute("data-node"));
      }
    },

    drop(ev) {
      if (ev.type === "touchend") {
        var parentdrawflow = document
          .elementFromPoint(
            mobile_last_move.touches[0].clientX,
            mobile_last_move.touches[0].clientY
          )
          .closest("#drawflow");
        if (parentdrawflow != null) {
          this.addNodeToDrawFlow(
            mobile_item_selec,
            mobile_last_move.touches[0].clientX,
            mobile_last_move.touches[0].clientY
          );
        }
      } else {
        ev.preventDefault();
        var data = ev.dataTransfer.getData("node");
        this.addNodeToDrawFlow(data, ev.clientX, ev.clientY);
      }
    },
    zoomOut() {
      this.editor.zoom_out();
    },
    zoomIn() {
      this.editor.zoom_in();
    },
    zoomReset() {
      this.editor.zoom_reset();
    },
    homeTab() {
      this.editor.changeModule("Home");
      this.changeModule(event);
    },
    otherTab() {
      this.editor.changeModule("Other");
      this.changeModule(event);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.column {
  border-right: 1px solid #494949;
}
.column ul {
  padding-inline-start: 0px;
}
.column li {
  background: transparent;
}
.node {
  border-radius: 8px;
  border: 2px solid #494949;
  display: block;
  width: 100%;
  height: 60px;
  line-height: 40px;
  padding: 10px;
  margin: 10px 0px;
  cursor: move;
}

#drawflow {
  text-align: initial;
  position: relative;
  height: 670px;
  border: 1px solid red;
  width: 100%;
  text-align: initial;
  background: #2b2c30;
  background-size: 20px 20px;
  background-image: radial-gradient(#494949 1px, transparent 1px);
}

.bar-zoom {
  position: absolute;
  right: 0;
  bottom: 0;
  z-index: 9999;
}
.menu {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 9999;
}
</style>
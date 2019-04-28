<template>
  <div id="app">
    <div class="result"></div>
    <div class="svg-container">
      <svg transform="translate(0.5, 0.5)">
        <defs>
          <marker
            id="arrow"
            markerWidth="10"
            markerHeight="10"
            refX="10"
            refY="3"
            orient="auto"
            markerUnits="strokeWidth"
          >
            <path d="M0,0 L0,6 L9,3 z" fill="#000"></path>
          </marker>
        </defs>
        <g v-for="(node, i) in graph.nodes" :key="`node-${i}`" @mousedown="handleMove($event,node)">
          <circle
            :cx="node.x"
            :cy="node.y"
            :r="radius"
            :class="{node, source: node.label == graph.source, sink: node.label === graph.sink}"
          ></circle>
          <text :x="node.x" :y="node.y">{{node.label}}</text>
        </g>
        <g :key="`edge-${i}`" v-for="(c,i) in connections">
          <line
            :x1="c.from[0]"
            :y1="c.from[1]"
            :x2="c.to[0]"
            :y2="c.to[1]"
            marker-end="url(#arrow)"
          ></line>
          <text :x="c.center[0]" :y="c.center[1]">{{c.label}}</text>
        </g>
      </svg>
    </div>
    <div class="menu-container">
      <div class="wrapper">
        <label for="verticle">
          Wierzchołek:
          <input class="input" type="text" v-model="label" name="verticle">
        </label>
        <button @click="addNode">Dodaj wierzchołek</button>
      </div>
      <div class="wrapper">
        <label for="source">
          Od:
          <input class="input" type="text" v-model="source" name="source">
        </label>
        <label for="destination">
          Do:
          <input class="input" type="text" v-model="destination" name="destination">
        </label>
        <label for="weight">
          Waga:
          <input class="input" type="text" v-model="weight" name="weight">
        </label>
        <button @click="addEdge">Dodaj krawędź</button>
      </div>
      <div class="wrapper">
        <label for="source">
          Źródło:
          <input class="input" type="text" v-model="graph.source" name="source">
        </label>
        <label for="sink">
          Ujście:
          <input class="input" type="text" v-model="graph.sink" name="sink">
        </label>
        <button @click="runAlgorithm">Uruchom algorytm</button>
      </div>
      <div class="wrapper">
        <ul class="list-container">
          <li class="list-item" v-for="(node, i) in graph.nodes" :key="`edit-${i}`">
            <input class="input" v-model="node.label">
            <input class="input" v-model.number="node.x">
            <input class="input" v-model.number="node.y">
            <button @click="removeNode(node.label, i)">X</button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
let node = (label, x, y) => {
  return {
    label,
    x,
    y
  };
};
let edge = (source, destination, weight) => {
  return { source, destination, weight };
};
let graph = {
  source: "",
  sink: "",
  nodes: [],
  edges: []
};
export default {
  name: "app",
  components: {},
  data() {
    return {
      graph,
      radius: 30,
      label: "",
      weight: "",
      source: "",
      destination: ""
    };
  },
  computed: {
    connections() {
      return this.graph.edges.map(({ source, destination, weight }) => {
        const from = this.getNode(source);
        const to = this.getNode(destination);
        const angle = Math.atan2(from.y - to.y, from.x - to.x);
        const pathFrom = [
          from.x - Math.cos(angle) * this.radius,
          from.y - Math.sin(angle) * this.radius
        ];
        const pathTo = [
          to.x + Math.cos(angle) * this.radius,
          to.y + Math.sin(angle) * this.radius
        ];
        const center = [
          (pathFrom[0] + pathTo[0]) / 2,
          (pathFrom[1] + pathTo[1]) / 2
        ];
        const label = this.labelPos(angle);
        return {
          from: pathFrom,
          to: pathTo,
          center: [center[0] + label[0], center[1] + label[1]],
          label: weight
        };
      });
    },
    transformToArray() {
      let graph = [];
      let length = this.graph.nodes.length;
      this.graph.nodes.forEach(node => {
        let array = new Array(length);
        array.fill(0);
        this.graph.edges.forEach(edge => {
          if (edge.source === node.label) {
            array[edge.destination] = edge.weight;
          }
        });
        graph.push(array);
      });
      return graph;
    }
  },
  mounted() {
    let move = ev => {
      if (this.last != null) {
        this.dragging.x -= this.last.x - ev.clientX;
        this.dragging.y -= this.last.y - ev.clientY;
        this.last = { x: ev.clientX, y: ev.clientY };
      }
    };
    let up = ev => {
      this.last = null;
      window.document.removeEventListener("onmousemove", move);
      window.document.removeEventListener("onmouseup", up);
    };
    this.up = up.bind(this);
    this.move = move.bind(this);
  },
  methods: {
    labelPos(angle) {
      const abs = Math.abs(angle);
      if (abs > Math.PI / 4 && abs < (Math.PI * 3) / 4) {
        return [6, 0];
      }
      return [0, -3];
    },
    getNode(label) {
      return this.graph.nodes.find(i => i.label === label);
    },
    handleMove(ev, node) {
      this.dragging = node;
      this.last = { x: ev.clientX, y: ev.clientY };
      window.document.addEventListener("mousemove", this.move);
      window.document.addEventListener("mouseup", this.up);
    },
    removeNode(label, i) {
      this.graph.nodes.splice(i, 1);
      this.graph.edges = this.graph.edges.filter(
        x => !(x.source === label || x.destination === label)
      );
    },
    addNode() {
      if (this.label !== "") {
        this.graph.nodes.push(node(this.label, 50, 50));
      }
      this.label = "";
    },
    addEdge() {
      if (this.source !== "" && this.destination !== "" && this.weight !== "") {
        this.graph.edges.push(edge(this.source, this.destination, this.weight));
      }
      this.source = "";
      this.destination = "";
      this.weight = "";
    },
    bfs(residualGraph, parent) {
      const graphLength = residualGraph.length;
      const source = this.graph.source;
      const sink = this.graph.sink;
      //tablica już odwiedzonych wierzchołków
      let visited = new Array(graphLength);
      visited.fill(false);
      //tworzymy kolejkę
      let queue = [];
      //wrzucamy na nią nasze źródło
      queue.push(source);
      //zaznaczamy, że zostało odwiedozne
      visited[source] = true;
      parent[source] = -1;
      //dopóki kolejka nie jest pusta
      while (queue.length != 0) {
        //zdejmujemy z niej element
        let u = queue.shift();
        //i lecimy pętlą po długości grafu
        for (let v = 0; v < graphLength; v++) {
          //jeżeli jeszcze nie odwiedzono
          if (!visited[v] && residualGraph[u][v] > 0) {
            //wrzucamy na kolejkę
            queue.push(v);
            //oznaczamy, że parentem tego wierzchołka jest wierzchołek U
            parent[v] = u;
            //zaznaczamy, że wierzchołek V został odwiedzony
            visited[v] = true;
          }
        }
      }
      //jeżeli przeszliśmy cały BFS, i osiągnęliśmy ujście startując ze źródła to zwracamy true, inaczej false.
      return visited[sink] === true;
    },
    runAlgorithm() {
      const parentDiv = document.querySelector(".result");
      //przekształcam pomocniczo nasz graf do tablic sąsiedztwa z wagami i prypisuję przy okazji do zmiennej sieci rezydualnej
      const residualGraph = this.transformToArray;
      const length = residualGraph.length;
      //do uzupełniania parentów w BFS
      let parent = new Array(length);
      //ustawiamy startowy maksymalny przepływ na 0
      let maxFlow = 0;
      const source = this.graph.source;
      const sink = this.graph.sink;
      //dopóki jest ścieżka ze źródła do ujścia
      while (this.bfs(residualGraph, parent)) {
        let pathFlow = Number.MAX_VALUE;
        const step = document.createElement("div");
        step.classList.add("step-container");
        const path = document.createElement("span");
        path.classList.add("path");
        const bandWidthContainer = document.createElement("div");
        bandWidthContainer.classList.add("bandwidth-container");
        let pathText = "";
        //od ujścia do źródła, idziemy po ścieżce i szukamy najmniejszej wartości (najmniejszej wagi)
        for (let v = sink; v != source; v = parent[v]) {
          let u = parent[v];
          pathText += `${v}---`;
          pathFlow = Math.min(pathFlow, residualGraph[u][v]);
        }
        pathText += "0";
        path.textContent +=
          "Ścieżka: " +
          pathText
            .split("")
            .reverse()
            .join("") +
          " najmniejsza waga: " +
          pathFlow;

        bandWidthContainer.textContent = "Przepustowość:";
        //od ujścia do źródła aktualizujemy każdą krawędź, zwiększając krawędź v-u a zmniejszając krawędź u-v
        // zmniejszamy przepustowość rezydualną krawędzi u-v. Pojawia się kanał przeciwny o wartości równej pathFlow
        for (let v = sink; v != source; v = parent[v]) {
          let u = parent[v];
          const bandwidth = document.createElement("span");
          bandwidth.classList.add("bandwidth-element");
          residualGraph[u][v] -= pathFlow;
          bandwidth.textContent += ` [${u},${v}] - ${pathFlow} = ${
            residualGraph[u][v]
          } w przeciwną stronę`;
          residualGraph[v][u] += pathFlow;
          bandwidth.textContent += ` [${v},${u}] + ${pathFlow} = ${
            residualGraph[v][u]
          }\n`;
          this.graph.edges.find(
            edge => edge.source == u && edge.destination == v
          ).weight = `${residualGraph[v][u]} / ${residualGraph[u][v]}`;
          bandWidthContainer.appendChild(bandwidth);
        }
        //zwiększamy nasz maksymalny przepływ
        maxFlow += pathFlow;
        const actualBandWidth = document.createElement("span");
        actualBandWidth.classList.add("bandwidth-element");
        actualBandWidth.textContent = "Aktualna przepustowość: " + maxFlow;
        step.appendChild(path);
        step.appendChild(bandWidthContainer);
        step.appendChild(actualBandWidth);
        pathText = "";
        parentDiv.appendChild(step);
      }
      const element = document.createElement("span");
      element.classList.add("maxbandwidth");
      element.textContent = "Maksymalny przepływ: " + maxFlow;
      parentDiv.appendChild(element);
    }
  }
};
</script>

<style lang="scss">
*,
*::after,
*::before {
  box-sizing: border-box;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  display: flex;
  justify-content: center;
  width: 100%;
}
ul {
  list-style: none;
}
.result {
  display: flex;
  flex-direction: column;
  padding: 10px;
  .step-container {
    margin: 10px;
    .path {
      display: block;
    }
    .bandwidth-container {
      .bandwidth-element {
        display: block;
        padding: 3px;
      }
    }
  }
  .maxbandwidth {
    display: block;
    margin: 10px;
  }
}
.menu-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.svg-container {
  display: flex;
  width: 800px;
  svg {
    flex: 1 1 100%;
    height: 600px;
    border: solid black 1px;
    circle.node {
      user-select: none;
      stroke-width: 3;
      stroke: black;
      fill: #ccc;
      transition: fill 1s;
      &.source {
        fill: rgb(206, 88, 33);
      }
      &.sink {
        fill: rgba(86, 144, 231, 0.795);
      }
    }
    circle.hl {
      fill: #aaa;
      transition: transform 1s;
    }
    text {
      user-select: none;
      fill: black;
      text-anchor: middle;
      font-weight: 600;
    }
    line {
      stroke: black;
      stroke-width: 1;
    }
    .disable {
      display: none;
    }
  }
}
</style>

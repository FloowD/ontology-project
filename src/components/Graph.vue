<script setup lang="ts">
import ontologyJson from '../data/ontology.json'
import { ref } from "vue"
import * as vNG from "v-network-graph"
import {
  ForceLayout,
  ForceNodeDatum,
  ForceEdgeDatum,
} from "v-network-graph/lib/force-layout"


const textRecherche = ref('')

const nodes = ref({});
const edges = ref({});
const nbNodes = ref(0);
const nbEdges = ref(0);
const layouts = ref({
  nodes: {
    node0: {
      x: 0,
      y: 0,
      fixed: true, // Unaffected by force
    },
  },
});
const depth = ref(1);
const father = ref(false);

const configs = ref(
  vNG.defineConfigs({
    view: {
      layoutHandler: new ForceLayout({
        positionFixedByDrag: false,
        positionFixedByClickWithAltKey: true,
        createSimulation: (d3, nodes, edges) => {
          // d3-force parameters
          const forceLink = d3.forceLink<ForceNodeDatum, ForceEdgeDatum>(edges).id(d => d.id)
          return d3
            .forceSimulation(nodes)
            .force("edge", forceLink.distance(30).strength(1))
            .force("charge", d3.forceManyBody().strength(-5000))
            .alphaMin(0.000000001)
        }
      }),
    },
    node: {
      normal: {
        // Change the color of the node according to the level
        color: node => node.level,
      },
      label: {
        visible: true,
        color: "black",
      },
      focusring: {
        color: "hotpink",
      },
    },
  })
);


function extractNodesAndEdges(dict) {
  let nodes = {};
  let relations = {};
  console.log("dict: " + dict.name)
  nodes[dict.name] = {name: dict.name, level: dict.level};
  if(dict.subs){
    dict.subs.forEach(sub => {
      let subnodesAndEdges = extractNodesAndEdges(sub);
      nodes = {...nodes, ...subnodesAndEdges[0]};
      relations = {...relations, ...subnodesAndEdges[1]};
      relations[dict.name + sub.name] = {source: dict.name, target: sub.name};
    })
  }
  return [nodes, relations];
}

function extractNodesAndEdgesToLevel(dict, stopLevel) {
  let nodes = {};
  let relations = {};
  nodes[dict.name] = { name: dict.name, level: dict.level };

  if (dict.subs && dict.level < stopLevel) {
    dict.subs.forEach(sub => {
      let subnodesAndEdges = extractNodesAndEdgesToLevel(sub, stopLevel);
      nodes = { ...nodes, ...subnodesAndEdges[0] };
      relations = { ...relations, ...subnodesAndEdges[1] };
      relations[dict.name + sub.name] = { source: dict.name, target: sub.name };
    });
  }

  return [nodes, relations];
}

function changeLevelToColor(nodes){
  let newNodes = {};
  for (const [key, value] of Object.entries(nodes)) {
    newNodes[key] = {name: value.name, level: levelToColor(value)};
  }
  return newNodes;
}

function levelToColor(node){
  if(node.level == 0){
      return "red"
    } else if(node.level == 1){
      return "blue"
    } else if(node.level == 2){
      return "green"
    } else if(node.level == 3){
      return "yellow"
    } else if(node.level == 4){
      return "orange"
    } else if(node.level == 5){
      return "purple"
    } else if(node.level == 6){
      return "pink"
    } else if(node.level == 7){
      return "brown"
    } else if(node.level == 8){
      return "grey"
    } else if(node.level == 9){
      return "black"
    } else {
      return "white"
    }
}

let nodeTest = {
    "name": "test",
    "level": "0",
    "subs": [
      {
        "name": "sub1",
        "level": "1",
        "subs": [
          {
            "name": "subsub1",
            "level": "2"
          },
          {
            "name": "subsub2",
            "level": "2"
          }
        ]
      },
      {
        "name": "sub2",
        "level": "1",
      }
    ]
  }

let nodesAndRelations = extractNodesAndEdges(ontologyJson);
// console.log("NodeTest : ", nodeTest);
// console.log("TypeNodeTest ", typeof nodeTest);
// let nodesAndRelations = extractNodesAndEdges(nodeTest);
nodesAndRelations[0] = changeLevelToColor(nodesAndRelations[0]);
console.log(nodesAndRelations[0]);
console.log(nodesAndRelations[1]);

// Initialiser les propriétés de données
nodes.value = nodesAndRelations[0];
edges.value = nodesAndRelations[1];
nbNodes.value = Object.keys(nodes.value).length;
nbEdges.value = Object.keys(edges.value).length;


function AfficheNiv(level){
  //Affiche seulement le level 1 du graph
  let nodesAndRelations = extractNodesAndEdgesToLevel(ontologyJson, level+1);
  updateGraph(nodesAndRelations[0], nodesAndRelations[1]);
}

function updateGraph(newNodes, newEdges) {
  nodes.value = changeLevelToColor(newNodes);
  edges.value = newEdges;
  nbNodes.value = Object.keys(nodes.value).length;
  nbEdges.value = Object.keys(edges.value).length;
}

function onInput(){
    const newNodesAndRelations = extractNodesAndEdgesByName(ontologyJson, textRecherche.value.toLowerCase(), depth.value)
    console.log(newNodesAndRelations[0]);
    console.log(newNodesAndRelations[1]);
    
    //If father, add the super concepts of textRecherche
    console.log("Father : ", father.value)
    
    
    updateGraph(newNodesAndRelations[0], newNodesAndRelations[1]);


}

function extractNodesAndEdgesByName(dict, name, depth) {
  let nodes = {};
  let relations = {};

  if (dict.name.toLowerCase().includes(name)) {
    nodes[dict.name] = { name: dict.name, level: dict.level };
    if(!father.value){
      const superConcepts = findSuperConcepts(ontologyJson, { name: dict.name, level: dict.level });
      const superConceptsNodesAndEdges = extractNodesAndEdgesToDepth(superConcepts, depth);
      nodes = { ...nodes, ...superConceptsNodesAndEdges[0] };
      relations = { ...relations, ...superConceptsNodesAndEdges[1] };
    }
  }

  if (dict.subs) {
    dict.subs.forEach(sub => {
      const subNodesAndEdges = extractNodesAndEdgesByName(sub, name, depth);
      nodes = { ...nodes, ...subNodesAndEdges[0] };
      relations = { ...relations, ...subNodesAndEdges[1] };
      if (dict.name.toLowerCase().includes(name)) {
        const subNodesAndEdges = extractNodesAndEdgesToDepth(dict, depth);
        nodes = { ...nodes, ...subNodesAndEdges[0] };
        relations = { ...relations, ...subNodesAndEdges[1] };
      }
    });
  }

  return [nodes, relations];
}

function extractNodesAndEdgesToDepth(dict, depth) {
  let nodes = {};
  let relations = {};
  nodes[dict.name] = { name: dict.name, level: dict.level };

  if (dict.subs && depth > 0) {
    dict.subs.forEach(sub => {
      const subNodesAndEdges = extractNodesAndEdgesToDepth(sub, depth - 1);
      nodes = { ...nodes, ...subNodesAndEdges[0] };
      relations = { ...relations, ...subNodesAndEdges[1] };
      relations[dict.name + sub.name] = { source: dict.name, target: sub.name };
    });
  }
  return [nodes, relations];
}

function findSuperConcepts(graph, node) {
  let superConcepts = {};

  function findSuperConceptsRec(currentNode, path = []) {
    if (!currentNode) {
      return;
    }

    if (currentNode.name === node.name && currentNode.level === node.level) {
      //Get the info of the current node from the graph

      // Found the target node, add its super concepts to the result
      superConcepts[currentNode.name] = { name: currentNode.name, level: currentNode.level, subs: path };
    } else if (currentNode.subs && currentNode.subs.length > 0 ) {
      // Continue searching in the sub-nodes
      for (const subNode of currentNode.subs) {
        findSuperConceptsRec(subNode, [{ name: currentNode.name, level: currentNode.level, subs: path }]);
      }
    }
  }

  // Start the recursive search from the root of the graph
  findSuperConceptsRec(graph);
  //return the first element of the object
  return superConcepts[Object.keys(superConcepts)[0]];
}

function getLevelFromNameInGraph(graph, nodeName){
  //Get the level of a nodeName in the graph
  let level = 0;
  let found = false;

  function getLevelFromNameInGraphRec(currentNode, path = []) {
    if (!currentNode) {
      return;
    }

    if (currentNode.name === nodeName.name) {
      //Get the info of the current node from the graph
      level = currentNode.level;
      found = true;
    } else if (currentNode.subs && currentNode.subs.length > 0) {
      // Continue searching in the sub-nodes
      for (const subNode of currentNode.subs) {
        getLevelFromNameInGraphRec(subNode, [{ name: currentNode.name, level: currentNode.level, subs: path }]);
      }
    }
  }

  // Start the recursive search from the root of the graph
  getLevelFromNameInGraphRec(graph);
  return level;
}

function test(){
  let testLevel = getLevelFromNameInGraph(nodeTest, { name: "subsub2"});
  console.log("testLevel: " + testLevel);
  let testNodes = findSuperConcepts(nodeTest, {name: "subsub1", level: getLevelFromNameInGraph(nodeTest, { name: "subsub1"})}, 1);
  console.log(testNodes)
  console.log("SuperConcept : ",findSuperConcepts(nodeTest, {name: "subsub2", level: "2"}));
  let test = extractNodesAndEdges(testNodes);
  test[0] = changeLevelToColor(test[0]);
  nodes.value = test[0];
  edges.value = test[1];
  nbNodes.value = Object.keys(nodes.value).length;
  nbEdges.value = Object.keys(edges.value).length;
  console.log("Test : ",test);
}


</script>

<template>
  <div class="container">
    <div class="nav">
      <div class="buttons">
        <button @click="AfficheNiv(1)">Affiche Niveau 1</button>
        <button @click="AfficheNiv(2)">Affiche Niveau 2</button>
        <button @click="AfficheNiv(3)">Affiche Niveau 3</button>
        <button @click="AfficheNiv(4)">Affiche tous le Graph</button>
        // <button @click="test">Test</button>
      </div>
      
      <div class="recherche">
        <h2>Rechercher un instrument</h2>
        <input type="text" id="search" name="search" placeholder="Recherche" @input="onInput" v-model="textRecherche">
      </div>
      <div class="sub-depth">
        <h2>Profondeur de recherche : {{ depth }}</h2>
        <input type="range" id="depth" name="depth" min="0" max="3" value="1" @input="onInput" v-model="depth">
      </div>
      <div class="see-father">
        <h2>Voir les super concepts ?</h2>
        <input type="checkbox" id="father" name="father" value="father" @input="onInput" v-model="father">
      </div>
    </div>

    <v-network-graph
      class="graph"
      :zoom-level="0.5"
      :nodes="nodes"
      :edges="edges"
      :layouts="layouts"
      :configs="configs"
    />
    <div class="data">
      <h2>Données</h2>
      <p>Nombre de Noeuds : {{ nbNodes }}</p>
      <p>Nombre d'Edges : {{ nbEdges }}</p>
    </div>
  </div>
</template>

<style>
.graph {
  width: 1200px;
  height: 800px;
  border: 1px solid #000;
}

.container {
  display: flex;
  flex-direction: row;
  justify-items: center;

}

.buttons {
  margin: 1rem;
  display: flex;
  flex-direction: column;
}

.buttons button {
  margin-bottom: 0.5rem; /* Ajustez la marge en fonction de vos besoins */
}

.recherche {
  margin: 2rem;
  display: flex;
  flex-direction: column;
  input {
    margin-bottom: 0.5rem; /* Ajustez la marge en fonction de vos besoins */
  }
}

h2{
  color: black;
  margin: 1rem;
}
p {
  color: black;
  margin: 1rem;
}
</style>

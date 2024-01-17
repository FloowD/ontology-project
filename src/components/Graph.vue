<script setup lang="ts">
import ontologyJson from '../data/ontology.json'
import { reactive, ref } from "vue"
import * as vNG from "v-network-graph"
import {
  ForceLayout,
  ForceNodeDatum,
  ForceEdgeDatum,
} from "v-network-graph/lib/force-layout"

const NODE_COUNT = 20


// The fixed position of the node can be specified.
const layouts = ref({
  nodes: {
    node0: {
      x: 0,
      y: 0,
      fixed: true, // Unaffected by force
    },
  },
})

const configs = reactive(
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
            .force("edge", forceLink.distance(60).strength(0.04))
            .force("charge", d3.forceManyBody().strength(-250))
            .alphaMin(0.001)
        }
      }),
    },
    node: {
      normal: {
        //Change the color of the node according to the level
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
)

function extractNodesAndEdges(dict) {
      let nodes = {}
      let relations = {}
      nodes[dict.name] = {name: dict.name, level: dict.level}
      if(dict.subs){
        dict.subs.forEach(sub => {
            let subnodesAndEdges = extractNodesAndEdges(sub)
            nodes = {...nodes, ...subnodesAndEdges[0]}
            relations = {...relations, ...subnodesAndEdges[1]}
            relations[dict.name + sub.name] = {source: dict.name, target: sub.name}
        })
    }
      return [nodes, relations]
  }

  function changeLevelToColor(nodes){
    let newNodes = {}
    for (const [key, value] of Object.entries(nodes)) {
      newNodes[key] = {name: value.name, level: levelToColor(value)}
    }
    return newNodes
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

  let nodesAndRelations = extractNodesAndEdges(ontologyJson)
  
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

  // nodesAndRelations = extractNodesAndEdges(nodeTest)
  nodesAndRelations[0] = changeLevelToColor(nodesAndRelations[0])
  console.log(nodesAndRelations[0])
  console.log(nodesAndRelations[1])
  
  const nodes = nodesAndRelations[0]
  const edges = nodesAndRelations[1]


</script>

<template>
  <v-network-graph
    class="graph"
    :zoom-level="0.5"
    :nodes="nodes"
    :edges="edges"
    :layouts="layouts"
    :configs="configs"
  />
</template>

<style>
.graph {
  width: 1200px;
  height: 800px;
  border: 1px solid #000;
}
</style>

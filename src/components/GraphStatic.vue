<script setup >
  import ontologyJson from '../data/ontology.json'
  import {
  ForceLayout,
} from "v-network-graph/lib/force-layout"

  function extractNodesAndEdges(dict) {
      let nodes = {}
      let relations = {}
      nodes[dict.name] = {name: dict.name}
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

  let nodesAndRelations = extractNodesAndEdges(ontologyJson)
  console.log(nodesAndRelations[0])
  console.log(nodesAndRelations[1])
  
  const nodes = nodesAndRelations[0]
  const edges = nodesAndRelations[1]


</script>

<template>
  <v-network-graph
    class="graph"
    :nodes="nodes"
    :edges="edges"
    :configs="configs"
  />
</template>

<style>
.graph {
  width: 800px;
  height: 600px;
  border: 1px solid #000;
}
</style>
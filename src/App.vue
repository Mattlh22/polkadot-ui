<template>
  <div id="app">
   <h3>{{genesis}}</h3>
    <HelloWorld msg="New Blocks"/>
    <div>
        <div v-for="item in newblocks" :key="item.hash.toHex()">
            Block Number: {{ item.number }}<br/>
            Author: {{ item.author }}<br/>
            Time: {{ item.time }}
            <hr/>
        </div>
    </div>
  </div>
</template>
<script>
import HelloWorld from './components/HelloWorld.vue'
import { ApiPromise, WsProvider } from '@polkadot/api';
import moment from 'moment'
const wsProvider = new WsProvider('wss://rpc.polkadot.io');

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  methods:{
      attach:async function()
        {
      this.api = await ApiPromise.create({ provider: wsProvider });
        // Retrieve the chain & node information information via rpc calls
        const [chain, nodeName, nodeVersion] = await Promise.all([
            this.api.rpc.system.chain(),
            this.api.rpc.system.name(),
            this.api.rpc.system.version()
        ]);

        this.genesis=`You are connected to chain ${chain} using ${nodeName} v${nodeVersion}`;
        this.api.derive.chain.subscribeNewHeads(async (lastHeader) => {
        let m =await this.api.query.timestamp.now();
        lastHeader.time= moment(parseInt(m)) .format("DD MMM YYYY hh:mm:ss a") 
        this.newblocks.unshift(lastHeader);
    });
  }},
  data: function () {
  return {api:ApiPromise,
      genesis:"          Loading......",
newblocks:[]
  }},
  mounted:function()
  {
      this.attach();
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

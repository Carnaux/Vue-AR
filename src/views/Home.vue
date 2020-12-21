<template>
  <div class="home">
    <ArButton :engine="'Arjs'" :tracking="'Marker'" :model="model" :coords="coords"></ArButton>
  </div>
</template>

<script>
import * as THREE from 'three'
import ArButton from '../components/ArButton.vue'
// @ is an alias to /src
export default {
  name: 'Home',
  components: {
    ArButton
  },
  data(){
    return {
      model: null,
      coords: {
        lat: -5.826232135866558,
        lng: -35.201280229372166
      }
    }
  },
  methods:{
    setModel(){
      const geometry = new THREE.BoxGeometry(0.2,0.2,0.2);
      const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
      const cube = new THREE.Mesh( geometry, material );

      this.model = cube;
    }
  },
  mounted(){
    this.setModel();
  }
}
</script>

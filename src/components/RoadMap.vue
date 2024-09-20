<template>
  <v-container fluid class="container">
    <div class="text-center">
    <v-btn
      color="red-darken-2"
      @click="snackbar = true"
    >
      Open Snackbar
    </v-btn>

    <v-snackbar
      v-model="snackbar"
      multi-line
    >
      {{ text }}

      <template v-slot:actions>
        <v-btn
          color="red"
          variant="text"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </div>
    <v-row justify="center">
      <v-col cols="12">
        <div ref="threeContainer" class="three-container"></div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'

const snackbar = ref(false)

const text = `I am a multi-line snackbar.\nI can have more than one line. This is another line that is quite long.`
</script>

<script>
import * as THREE from 'three';

export default {
  data() {
    return {
      snackbar: false,
      text: `I am a multi-line snackbar.\nI can have more than one line. This is another line that is quite long.`,
    }
  },
  name: 'ThreeDRoadmap',
  mounted() {
    this.initThreeJS();
  },
  methods: {
    openSnackbar() {
      this.snackbar.show = true; // Open the Snackbar when button is clicked
    },
    closeSnackbar() {
      this.snackbar.show = false; // Close the Snackbar
    },
    initThreeJS() {
      // Basic Scene Setup
      const scene = new THREE.Scene();

      // Set background color to purple
      scene.background = new THREE.Color(0x800080);

      const camera = new THREE.PerspectiveCamera(
        75,
        this.$refs.threeContainer.offsetWidth / this.$refs.threeContainer.offsetHeight,
        0.1,
        1000
      );
      const renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(this.$refs.threeContainer.offsetWidth, this.$refs.threeContainer.offsetHeight);
      this.$refs.threeContainer.appendChild(renderer.domElement);

      // Lighting
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
      scene.add(ambientLight);

      const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight.position.set(10, 10, 10).normalize();
      scene.add(directionalLight);

      // Create Road
      const roadShape = new THREE.Shape();
      roadShape.moveTo(-5, -1);
      roadShape.lineTo(5, -1);
      roadShape.lineTo(5, 1);
      roadShape.lineTo(-5, 1);
      roadShape.lineTo(-5, -1);

      const extrudeSettings = {
        steps: 2,
        depth: 0.2,
        bevelEnabled: false,
      };

      const roadGeometry = new THREE.ExtrudeGeometry(roadShape, extrudeSettings);
      const roadMaterial = new THREE.MeshLambertMaterial({ color: 0x666666 });
      const roadMesh = new THREE.Mesh(roadGeometry, roadMaterial);
      roadMesh.rotation.x = Math.PI / 2;
      roadMesh.position.y = 0;
      scene.add(roadMesh);

      // Create Milestones
      const milestoneGeometry = new THREE.CylinderGeometry(0.3, 0.3, 0.5, 32);
      const milestoneMaterial = new THREE.MeshLambertMaterial({ color: 0xff0000 });

      const milestones = [];
      for (let i = 0; i < 4; i++) {
        const milestoneMesh = new THREE.Mesh(milestoneGeometry, milestoneMaterial);
        milestoneMesh.position.set(-4 + i * 3, 0.5, 0);
        scene.add(milestoneMesh);
        milestones.push(milestoneMesh);
      }

      // Interactivity: Move camera with mouse
      const onMouseMove = (event) => {
        const mouseX = (event.clientX / window.innerWidth) * 2 - 1;
        const mouseY = -(event.clientY / window.innerHeight) * 2 + 1;

        camera.position.x = mouseX * 2;
        camera.position.y = mouseY * 2 + 2; // Maintain some height above the road
        camera.lookAt(scene.position);
      };

      window.addEventListener('mousemove', onMouseMove);

      // Camera Position
      camera.position.z = 7;
      camera.position.y = 2;
      camera.lookAt(0, 0, 0);

      // Animation Loop
      const animate = () => {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
      };

      animate();

      // Handle resize
      window.addEventListener('resize', () => {
        camera.aspect = this.$refs.threeContainer.offsetWidth / this.$refs.threeContainer.offsetHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(this.$refs.threeContainer.offsetWidth, this.$refs.threeContainer.offsetHeight);
      });
    },
  },
};
</script>

<style scoped>
.container {
  background-color: #800080; /* Adjust the container's background to match the scene */
}
</style>

<style>
.three-container {
  width: 100%;
  height: 500px;
}
</style>

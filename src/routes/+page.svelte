<script>
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import * as YUKA from 'yuka';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

	if (browser) {
		const renderer = new THREE.WebGLRenderer({ antialias: true });
		renderer.setSize(window.innerWidth, window.innerHeight);
		document.body.appendChild(renderer.domElement);

		// Sets the color of the background
		renderer.setClearColor(0xfefefe);

		const scene = new THREE.Scene();
		const camera = new THREE.PerspectiveCamera(
			45,
			window.innerWidth / window.innerHeight,
			0.1,
			1000
		);

		// Sets orbit control to move the camera around
		const orbit = new OrbitControls(camera, renderer.domElement);

		// Camera positioning
		camera.position.set(6, 8, 14);
		orbit.update();

		function animate() {
			renderer.render(scene, camera);
		}
		// add lighting
		const directionalLight = new THREE.DirectionalLight(0xffffff, 2);
		scene.add(directionalLight);

		// add RaceTrack
		const loader = new GLTFLoader();
		loader.load('MonacoTrackScaled.glb', function (glb) {
			const model = glb.scene;
			scene.add(model);
		});

		// add navMesh
		const navmeshLoader = new YUKA.NavMeshLoader();
		navmeshLoader.load('MonacoNavMesh.glb').then((navigationMesh) => {});

		// add vehicle
		const vehicleGeometry = new THREE.ConeGeometry(0.125, 0.5, 16);
		vehicleGeometry.rotateX(Math.PI * 0.5);
		vehicleGeometry.translate(0, 0.25, 0);
		const vehicleMaterial = new THREE.MeshNormalMaterial();

		const vehicleMesh = new THREE.Mesh(vehicleGeometry, vehicleMaterial);
		vehicleMesh.matrixAutoUpdate = false;
		scene.add(vehicleMesh);

		renderer.setAnimationLoop(animate);

		window.addEventListener('resize', function () {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		});
	}
</script>

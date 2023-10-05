<script>
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import * as YUKA from 'yuka';
	import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
	import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
	import { createGraphHelper } from '$lib/graph_helper.js';
	import { createConvexRegionHelper } from '$lib/navmesh_helper.js';

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
		const controls = new OrbitControls(camera, renderer.domElement);

		// Camera positioning
		camera.position.set(6, 8, 14);
		camera.lookAt(scene.position);

		// add lighting
		const amientLight = new THREE.AmbientLight(0x9999999);
		scene.add(amientLight);

		const directionalLight = new THREE.DirectionalLight(0xffffff, 2);
		scene.add(directionalLight);
		directionalLight.position.set(3, 10, 2);

		// add vehicle
		const vehicleGeometry = new THREE.ConeGeometry(0.125, 0.5, 16);
		vehicleGeometry.rotateX(Math.PI * 0.5);
		vehicleGeometry.translate(0, 0.25, 0);
		const vehicleMaterial = new THREE.MeshNormalMaterial();

		const vehicleMesh = new THREE.Mesh(vehicleGeometry, vehicleMaterial);
		vehicleMesh.matrixAutoUpdate = false;
		scene.add(vehicleMesh);

		function sync(entity, renderComponent) {
			renderComponent.matrix.copy(entity.worldMatrix);
		}

		const entityManager = new YUKA.EntityManager();

		const followPathBehavior = new YUKA.FollowPathBehavior();
		followPathBehavior.active = false;
		followPathBehavior.nextWaypointDistance = 0.5;

		// add RaceTrack
		const loader = new GLTFLoader();
		loader.load('MonacoTrackScaled.glb', function (glb) {
			const model = glb.scene;
			scene.add(model);
		});

		// add navMesh
		const navmeshLoader = new YUKA.NavMeshLoader();
		navmeshLoader.load('MonacoNavMeshScaled3.glb').then((navigationMesh) => {
			const navMesh = navigationMesh;

			const graph = navMesh.graph;
			const graphHelper = createGraphHelper(graph, 0.2);
			scene.add(graphHelper);

			const navMeshGroup = createConvexRegionHelper(navMesh);
			scene.add(navMeshGroup);

			const vehicle = new YUKA.Vehicle();
			vehicle.setRenderComponent(vehicleMesh, sync);
			entityManager.add(vehicle);
			vehicle.steering.add(followPathBehavior);

			const mousePosition = new THREE.Vector2();
			const raycaster = new THREE.Raycaster();

			window.addEventListener('click', function (e) {
				mousePosition.x = (e.clientX / this.window.innerWidth) * 2 - 1;
				mousePosition.y = (e.clientY / this.window.innerHeight) * 2 + 1;

				raycaster.setFromCamera(mousePosition, camera);
				const intersects = raycaster.intersectObject(navMeshGroup);
				console.log(intersects.length);
				if (intersects.length > 0) {
					findPathTo(new YUKA.Vector3().copy(intersects[0].point));
				}
			});

			function findPathTo(target) {
				console.log('click');
				const from = vehicle.position;
				const to = target;
				const path = navMesh.findPathTo(from, to);

				const followPathBehavior = vehicle.steering.behaviors[0];
				followPathBehavior.active = true;
				followPathBehavior.path.clear();

				for (let point of path) followPathBehavior.path.add(point);
			}
		});

		const time = new YUKA.Time();
		function animate() {
			const delta = time.update().getDelta();
			entityManager.update(delta);
			renderer.render(scene, camera);
		}

		renderer.setAnimationLoop(animate);

		window.addEventListener('resize', function () {
			camera.aspect = window.innerWidth / window.innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(window.innerWidth, window.innerHeight);
		});
	}
</script>

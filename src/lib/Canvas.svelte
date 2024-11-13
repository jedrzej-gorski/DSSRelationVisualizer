<script>
    import { onMount } from "svelte";
    import * as THREE from "three";

    let canvas;
    const colorDict = {
        0: 0xff0000,
        1: 0x00ffff,
        2: 0xeeeeee,
        3: 0xee00ee,
    };

    const boxWidth = 0.005;
    const boxHeight = 0.005;
    const boxDepth = 0.005;
    const circleRadius = 0.005;
    //const geometry = new THREE.BoxGeometry(boxWidth, boxHeight, boxDepth);
    const geometry = new THREE.CircleGeometry(circleRadius, 6);

    let { pointData = [] } = $props();
    let isReady = $state(false);
    let cubes;
    let scene;
    let renderer;
    let camera;
    let angle = 0;
    const pivot = new THREE.Vector3(0.5, 0.5, 0.5);
    const origin = new THREE.Vector3(0.5, 0.5, 2.5);
    let savedPosition = origin.clone();
    let savedRotation = new THREE.Vector3(0, 0, 0);

    let binaryMode = false;
    let visualizationState = 0;
    let depth = 0;

    $effect(() => {
        cubes = [];
        pointData.forEach((point) => {
            cubes.push(
                makeCubeInstance(
                    scene,
                    geometry,
                    point[0],
                    point[1],
                    point[2],
                    point[3],
                ),
            );
        });
    });

    function makeCubeInstance(scene, geometry, x, y, z, color) {
        const material = new THREE.MeshBasicMaterial({
            color: colorDict[color],
        });

        const cube = new THREE.Mesh(geometry, material);
        cube.lookAt(camera.position);
        scene.add(cube);

        cube.position.x = x;
        cube.position.y = y;
        cube.position.z = z;

        return cube;
    }

    function needsToBeResized(canvas) {
        const needResize =
            canvas.width !== canvas.clientWidth ||
            canvas.height !== canvas.clientHeight;
        return needResize;
    }

    function resize() {
        const canvas = renderer.domElement;
        if (needsToBeResized(canvas)) {
            renderer.setSize(canvas.clientWidth, canvas.clientHeight, false);
            camera.aspect = canvas.clientWidth / canvas.clientHeight;
            camera.updateProjectionMatrix();
        }
    }

    function render(time) {
        time *= 0.001;

        let rotationAxis = new THREE.Vector3(0, 0, 0);
        let angle = 0;
        let right = new THREE.Vector3(1, 0, 0)
            .applyEuler(camera.rotation)
            .normalize();
        let up = new THREE.Vector3(0, 1, 0)
            .applyEuler(camera.rotation)
            .normalize();
        let forward = new THREE.Vector3(0, 0, 1)
            .applyEuler(camera.rotation)
            .multiplyScalar(-0.02);
        resize();

        if (inputMap["Alt"] && inputMap["w"]) {
            camera.position.add(forward);
        } else if (inputMap["Alt"] && inputMap["s"]) {
            camera.position.sub(forward);
        } else if (inputMap["w"]) {
            // W - rotate clockwise along x-axis
            if (binaryMode) {
                depth = Math.min(1, depth + 0.01);
            }
            else {
                rotationAxis = right;
                angle = 0.03;
            }
        } else if (inputMap["s"]) {
            // S - rotate anti-clockwise along x-axis
            if (binaryMode) {
                depth = Math.max(0, depth - 0.01);
            }
            else {
                rotationAxis = right;
                angle = -0.03;
            }
        } else if (inputMap["a"]) {
            // A - rotate clockwise along y-axis
            rotationAxis = up;
            angle = 0.03;
        } else if (inputMap["d"]) {
            // D - rotate anti-clockwise along y-axis
            rotationAxis = up;
            angle = -0.03;
        } else if (inputMap["b"]) {
            if (!visualizationState) {
                visualizationState = 1;
            }
        }

        if (!inputMap["b"] && visualizationState) {
            if (!binaryMode) {
                savedPosition.copy(camera.position);
                savedRotation.copy(camera.rotation);
                camera.position.copy(origin);
                camera.lookAt(0.5, 0.5, 0.5);
            } else {
                camera.position.copy(savedPosition);
                camera.rotation.copy(savedRotation);
                cubes.forEach((cube) => {
                    cube.visible = true;
                });
            }
            binaryMode = !binaryMode;
            visualizationState = 0;
        }

        if (!binaryMode) {
            let quaternion = new THREE.Quaternion();
            quaternion.setFromAxisAngle(rotationAxis, angle);

            camera.position.sub(pivot);
            camera.position.applyQuaternion(quaternion);
            camera.applyQuaternion(quaternion);
            camera.position.add(pivot);
        } else {
            cubes.forEach((cube) => {
                if (Math.abs(cube.position.z - depth) > 0.05) {
                    cube.visible = false;
                }
                else {
                    cube.visible = true;
                }
            });
        }

        cubes.forEach((cube) => {
            cube.lookAt(camera.position);
        });

        console.log(camera.rotation);

        renderer.render(scene, camera);
        requestAnimationFrame(render);
    }

    onMount(() => {
        renderer = new THREE.WebGLRenderer({ antialias: true, canvas });
        scene = new THREE.Scene();
        const fov = 75;
        const aspect = 2;
        const near = 0.1;
        const far = 5;
        camera = new THREE.PerspectiveCamera(fov, aspect, near, far);

        camera.position.copy(origin);
        camera.lookAt(pivot);

        savedRotation = camera.rotation.clone();

        const color = 0xffffff;
        const light = new THREE.AmbientLight(color);
        const axesHelper = new THREE.AxesHelper(5);

        scene.add(axesHelper);

        scene.add(light);

        requestAnimationFrame(render);

        isReady = true;
    });

    let inputMap = {};
    onkeydown = onkeyup = function (e) {
        inputMap[e.key] = e.type == "keydown";
    };
</script>

<div>
    <canvas bind:this={canvas} id="c"> </canvas>
</div>

<svelte:window {onkeydown} {onkeyup} />

<style>
    #c {
        width: 100%;
        height: 100%;
        display: block;
    }
</style>

<script>
    import { onMount } from "svelte";
    import * as THREE from "three";
    import * as BufferGeometryUtils from "three/addons/utils/BufferGeometryUtils.js";
    import ToolPalette from "./ToolPalette.svelte";

    let canvas;
    const colorDict = {
        0: new THREE.Color("rgb(255, 0, 0)"),
        1: new THREE.Color("rgb(0, 255, 255)"),
        2: new THREE.Color("rgb(238, 238, 238)"),
        3: new THREE.Color("rgb(238, 0, 238)"),
    };

    const boxWidth = 0.005;
    const boxHeight = 0.005;
    const boxDepth = 0.005;
    const circleRadius = 0.005;
    const delta = 0.01;
    //const geometry = new THREE.BoxGeometry(boxWidth, boxHeight, boxDepth);
    const geometry = new THREE.OctahedronGeometry(circleRadius, 0);
    let mergedGeometry = new THREE.BufferGeometry();
    let material = new THREE.MeshBasicMaterial();

    let { pointData = [], metadata = [] } = $props();

    let fieldX = $state(null);
    let fieldY = $state(null);
    let fieldZ = $state(null);
    let definedFieldCount = $derived.by(() => {
        let count = 0;
        if (fieldX != null) {
            count++;
        }
        if (fieldY != null) {
            count++;
        }
        if (fieldZ != null) {
            count++;
        }
        return count;
    });
    let validPoints = $derived.by(() => {
        if (definedFieldCount > 1 && pointData.length > 0) {
            return pointData.filter((point) => {
                let valid = true;
                savedDimensions.forEach((dimension, index) => {
                    if (
                        fieldX != index &&
                        fieldY != index &&
                        fieldZ != index &&
                        attributeActivity[index]
                    ) {
                        if (Math.abs(point[index] - dimension) > delta) {
                            valid = false;
                            return;
                        }
                    }
                });
                return valid;
            });
        }
        return [];
    });
    $inspect(validPoints);

    let attributes = $derived.by(() => {
        let newAttributes = [];
        if (pointData.length > 0) {
            for (let i = 0; i < pointData[0].length - 1; i++) {
                newAttributes.push(`Val${i}`);
            }
        }
        return newAttributes;
    });
    let isReady = $state(false);
    let scene;
    let renderer;
    let camera;
    let angle = 0;
    const pivot = new THREE.Vector3(0.5, 0.5, 0.5);
    const origin = new THREE.Vector3(0.5, 0.5, 2.5);
    let savedPosition = origin.clone();
    let savedRotation = new THREE.Vector3(0, 0, 0);

    let savedDimensions = $state(null);
    // TODO: Change name to match bool convention
    let attributeActivity = $state(null);
    let mesh = null;
    let binaryMode = false;
    let showAllMode = false;
    let visualizationState = 0;
    let swap = false;
    let depth = $state(0);

    let hasInitializedDimensions = false;

    $effect(() => {
        if (metadata.length - 1 > 0 && !hasInitializedDimensions) {
            // Set default values to minimum values of each column
            savedDimensions = metadata.map((v) => {
                return v[0];
            });
            attributeActivity = metadata.map((v) => {
                return true;
            });
            hasInitializedDimensions = true;
        }

        if (mesh != null && scene != null) {
            let oldMesh = mesh;
            scene.remove(oldMesh);
        }
        mesh = new THREE.InstancedMesh(geometry, material, validPoints.length);
        setInstances();
        if (isReady) {
            scene.add(mesh);
        }
    });

    function setInstances() {
        validPoints.forEach((point, index) => {
            const transformMatrix = new THREE.Matrix4();
            if (definedFieldCount == 3) {
                transformMatrix.makeTranslation(
                    point[fieldX],
                    point[fieldY],
                    point[fieldZ],
                );
            } else {
                if (fieldX == null) {
                    transformMatrix.makeTranslation(
                        0,
                        point[fieldY],
                        point[fieldZ],
                    );
                } else if (fieldY == null) {
                    transformMatrix.makeTranslation(
                        point[fieldX],
                        0,
                        point[fieldZ],
                    );
                } else if (fieldZ == null) {
                    transformMatrix.makeTranslation(
                        point[fieldX],
                        point[fieldY],
                        0,
                    );
                }
            }
            mesh.setMatrixAt(index, transformMatrix);
            mesh.setColorAt(index, colorDict[point[4]]);
        });
    }

    function makeCubeInstance(scene, geometry, x, y, z, w, color) {
        const material = new THREE.MeshBasicMaterial({
            color: colorDict[color],
        });

        const cube = new THREE.Mesh(geometry, material);
        cube.lookAt(camera.position);
        scene.add(cube);

        cube.position.x = x;
        cube.position.y = y;
        cube.position.z = z;
        cube.position.w = w;

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
            } else {
                rotationAxis = right;
                angle = 0.03;
            }
        } else if (inputMap["s"]) {
            // S - rotate anti-clockwise along x-axis
            if (binaryMode) {
                depth = Math.max(0, depth - 0.01);
            } else {
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
        } else if (inputMap["o"]) {
            if (!binaryMode) {
                depth = Math.min(1, depth + 0.01);
            }
        } else if (inputMap["l"]) {
            if (!binaryMode) {
                depth = Math.max(0, depth - 0.01);
            }
        } else if (inputMap["p"]) {
            swap = true;
        }

        if (!inputMap["p"] && swap) {
            swap = false;
            showAllMode = !showAllMode;
            depthDimension = -1;
        }

        if (!inputMap["b"] && visualizationState) {
            if (!binaryMode) {
                savedPosition.copy(camera.position);
                savedRotation.copy(camera.rotation);
                camera.position.copy(origin);
                camera.lookAt(0.5, 0.5, 0.5);
                depthDimension = 2;
            } else {
                camera.position.copy(savedPosition);
                camera.rotation.copy(savedRotation);
                depthDimension = 3;
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
        }

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

<div id="container">
    <canvas bind:this={canvas} id="c"> </canvas>
    <ToolPalette
        {attributes}
        {metadata}
        {definedFieldCount}
        bind:fieldX
        bind:fieldY
        bind:fieldZ
        bind:savedDimensions
        bind:attributeActivity
    ></ToolPalette>
</div>

<svelte:window {onkeydown} {onkeyup} />

<style>
    #container {
        position: absolute;
        width: 100%;
        height: 100%;
    }
    #c {
        position: relative;
        width: 100%;
        height: 100%;
        display: block;
    }
</style>

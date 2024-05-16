<script setup lang="ts">
    import {
        Engine,
        FreeCamera,
        GroundMesh,
        HemisphericLight,
        Scene,
        StandardMaterial,
        Vector3,
        MeshBuilder,
    } from "@babylonjs/core";
    import {onBeforeUnmount, onMounted, ref, type Ref} from "vue";
    import {GrassProceduralTexture} from "@babylonjs/procedural-textures";

    const renderCanvas: Ref<HTMLCanvasElement | undefined> = ref();

    let engine: Engine;

    const initEngine = (): void => {
        engine = new Engine(renderCanvas.value!, true);
    };

    let scene: Scene;

    const initSceneGravity = () => {
        const fps = 144;
        const gravity = -9.81;

        scene.gravity = new Vector3(0, gravity / fps, 0);
        scene.collisionsEnabled = true;
    };

    const createScene = (): void => {
        scene = new Scene(engine);
        initSceneGravity();
    };

    const createLight = (): void => {
        new HemisphericLight('light', new Vector3(0, 1, 0), scene);
    };

    let camera: FreeCamera;

    const createCamera = (): void => {
        camera = new FreeCamera('camera', new Vector3(0, 4, 0), scene);
    };

    const attachCameraControl = (): void => {
        camera.attachControl();

        camera.checkCollisions = true;
        camera.applyGravity = true;
        camera.ellipsoid = new Vector3(1, 2, 1);
        camera.minZ = 0.01;
        camera.speed = 2;
        camera.inertia = 0.5;
        camera.angularSensibility = 350;

        scene.onPointerDown = (event) => {
            if (event.button === 0) {
                engine.enterPointerlock();
            }
            if (event.button === 1) {
                engine.exitPointerlock();
            }
        };
    };

    const connectWASDControl = (): void => {
        camera.keysUp.push(87); //W
        camera.keysLeft.push(65); //A
        camera.keysDown.push(83); //S
        camera.keysRight.push(68); //D
    };

    let interval: number = 0;
    let totalStamina = 30;

    const stamina: Ref<number> = ref(30);

    const startSprinting = (event: KeyboardEvent): void => {
        if (event.key === 'Shift') {
            clearInterval(interval);

            interval = setInterval(() => {
                if (stamina.value > 0) {
                    camera.speed = 4;
                    stamina.value -= 1;
                } else {
                    camera.speed = 2;
                }
            }, 100);
        }
    };
    const stopSprinting = (event: KeyboardEvent): void => {
        if (event.key === 'Shift') {
            camera.speed = 2;

            clearInterval(interval);

            interval = setInterval(() => {
                if (stamina.value < totalStamina) {
                    stamina.value += 1;
                }
            }, 100);
        }
    };

    const connectSprintControls = () => {
        window.addEventListener('keydown', (event) => startSprinting(event));
        window.addEventListener('keyup', (event) => stopSprinting(event));
    };

    let ground: GroundMesh;

    const createGround = (): void => {
        ground = MeshBuilder.CreateGround('ground', {width: 100, height: 100}, scene);
        ground.checkCollisions = true;

        const grassMaterial: StandardMaterial = new StandardMaterial('grass_material', scene);

        grassMaterial.ambientTexture = new GrassProceduralTexture('grass_texture', 1024, scene);
        ground.material = grassMaterial;
    };

    const startRenderLoop = () => {
        engine.runRenderLoop(() => {
            scene.render();
        });
    };

    onMounted((): void => {
        if (renderCanvas.value) {
            //Init engine, game and lighting
            initEngine();
            createScene();
            createLight();

            //Add camera and controls
            createCamera();
            attachCameraControl();
            connectWASDControl();
            connectSprintControls();

            //Create game meshes
            createGround();

            //Start game render loop
            startRenderLoop();
        }
    });

    onBeforeUnmount((): void => {
        window.removeEventListener('keydown', () => startSprinting);
        window.removeEventListener('keyup', () => stopSprinting);
    });
</script>

<template>
    <div class="relative w-full aspect-video">
        <canvas ref="renderCanvas" class="w-full aspect-video block"/>

        <div class="fixed top-4 left-4 w-96 h-4 bg-black/30 rounded-md overflow-hidden">
            <div class="h-full bg-green-600 transition-all duration-75 rounded-md"
                 :style="{width: `${(stamina/30) * 100}%`}"
            />
        </div>
    </div>
</template>

<style scoped>

</style>
<template>
    <div ref="three" />
</template>
  
<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import TWEEN from 'three-tween'

const bufArrays = [];
let current = 0;

export default {
    mounted() {
        this.initGraph()
    },
    methods: {
        initGraph() {
            // 初始化场景
            this.scene = new THREE.Scene();
            // setting scene background
            this.scene.background = this.sceneTexture();

            // setting camera and set its position 透视相机
            this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 100);
            this.camera.position.set(0, 0, 6);

            // setting render
            this.renderer = new THREE.WebGLRenderer({ antialias: true });
            this.renderer.setPixelRatio(window.devicePixelRatio);
            this.renderer.setSize(window.innerWidth, window.innerHeight);

            // add axes 
            // const axesHelper = new THREE.AxesHelper(5);
            // this.scene.add(axesHelper);

            const manager = new THREE.LoadingManager();
            manager.onStart = () => {
                console.log('onStart')
            }

            manager.onLoad = () => {
                console.log('onLoad')
                this.transition()
            }

            manager.onError = url => {
                console.log(url)
            }
            // import model 
            const gltfLoader = new GLTFLoader(manager);
            gltfLoader.load('/box.glb', gltf => {
                gltf.scene.traverse(child => {
                    if (child.isMesh) {
                        child.geometry.translate(0, 0.5, 0);
                        const { array } = child.geometry.attributes.position;
                        bufArrays.push(array);
                    }
                })
            });
            gltfLoader.load('/box1.glb', gltf => {
                gltf.scene.traverse(child => {
                    if (child.isMesh) {
                        child.geometry.scale(0.5, 0.5, 0.5);
                        const { array } = child.geometry.attributes.position;
                        bufArrays.push(array);
                    }
                })
            });
            gltfLoader.load('/sphere.glb', gltf => {
                gltf.scene.traverse(child => {
                    if (child.isMesh) {
                        child.geometry.translate(1, 0, 0);
                        const { array } = child.geometry.attributes.position;
                        bufArrays.push(array);
                    }
                })
            });

            //create points
            this.geometry = new THREE.BufferGeometry();
            // console.log(this.geometry);
            this.geometry.tween = [];
            const vertices = [];
            for (let i = 0; i < 26016; i++) {
                const position = THREE.MathUtils.randFloat(-4, 4); // random setting points position
                this.geometry.tween.push(new TWEEN.Tween({ position }).easing(TWEEN.Easing.Exponential.In));
                vertices.push(position);
            };

            this.geometry.setAttribute('position', new THREE.BufferAttribute(new Float32Array(vertices), 3))

            // create and setting points then add them in scene
            this.points = new THREE.Points(this.geometry, new THREE.PointsMaterial({
                size: 0.032,
                map: new THREE.TextureLoader().load('/white-dot.png'),
                alphaTest: 0,
                opacity: 0.5,
                transparent: true,
                depthTest: true
            }));
            this.scene.add(this.points);

            // initialize and create controls
            this.controls = new OrbitControls(this.camera, this.renderer.domElement);
            this.controls.update();
            this.$refs.three.appendChild(this.renderer.domElement);
            // window size listening
            window.addEventListener('resize', this.changeWindowSize, false);

            this.render();  // render the scene
        },

        // points movement between particle and model
        transition() {
            const self = this;
            // console.log(this.geometry.tween)
            for (let i = 0, j = 0; i < 26016; i++, j++) {
                // console.log(bufArrays);
                const item = this.geometry.tween[i];
                if (j >= bufArrays[current].length) {
                    j = 0;
                }
                item.to({ position: bufArrays[current][j] },
                    THREE.MathUtils.randFloat(1000, 4000)).onUpdate(function () {
                        self.geometry.attributes.position.array[i] = this.position;
                        self.geometry.attributes.position.needsUpdate = true;
                    }).start()
            }
            // console.log(self.geometry.attributes.position);
            setTimeout(() => { this.transition() }, 5000);
            current = (current + 1) % 3;
        },

        // render the whole fragment, setting the points' movement
        render() {
            this.points.rotation.x += 0.0003;
            this.points.rotation.y += 0.001;
            this.points.rotation.z += 0.002;
            TWEEN.update();
            this.renderer.render(this.scene, this.camera);
            requestAnimationFrame(this.render);
        },

        // listening window size
        changeWindowSize() {
            this.renderer.setSize(window.innerWidth, window.innerHeight);
            this.camera.aspect = window.innerWidth / window.innerHeight;
            this.camera.updateProjectionMatrix();
        },

        // function of setting background of scene
        sceneTexture() {
            // create canvas and setting its size
            this.canvas = document.createElement('canvas');
            this.canvas.width = window.innerWidth;
            this.canvas.height = window.innerHeight;
            // creating canvas environment 
            this.ctx = this.canvas.getContext('2d');
            // setting style of texture of cancas 
            const gradient = this.ctx.createLinearGradient(0, 0, window.innerWidth, 0);
            gradient.addColorStop(0, '#0F2027');
            gradient.addColorStop(1, '#2C5364');
            this.ctx.fillStyle = gradient;
            // this.ctx.fillStyle = '#f4f4f4';
            this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
            // setting canvas texture
            this.canvasTexture = new THREE.CanvasTexture(this.canvas);
            return this.canvasTexture
        }
    }
}
</script>
  
<template>
    <div id="ar_container">
        <button id="enterAR" class="btn sizeEnter" v-on:click="init">START AR</button>

        <div id="webxr">
            <div id="overlay">
                <div id="searchingDiv">
                    <PlaneSVG id="searchingPlaneSVG"></PlaneSVG>
                    <p id="searchingText">SEARCHING FOR A PLANE</p>
                </div>
                <div id="modelControllers">
                    <div id="modelButtons">
                        <button id="modelScale" class="btn" v-on:click="scaleInput">Sca</button>
                        <button id="modelRotation" class="btn" v-on:click="rotationInput">Rot</button>
                        <button id="toggleReticle" class="btn reticle" v-on:click="webxr_toggleReticle">Ret</button>
                    </div>
                    <input id="inputController" type="range" step="0.1" @change="inputValue"/>
                </div>
            </div>
        </div>

        <div v-if="arjs_bool" id="arjs_aframe">
            <a-scene
                id="arjs_scene"
                vr-mode-ui="enabled: false;"
                renderer="logarithmicDepthBuffer: true;"
                arjs="trackingMethod: best; sourceType: webcam; debugUIEnabled: false;">

                <div v-if="arjs_imageTrackingBool" id="arjs_imageTracking">
                    <a-nft
                        type="nft"
                        url="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/trex-image/trex"
                        smooth="true"
                        smoothCount="10"
                        smoothTolerance=".01"
                        smoothThreshold="5">
                        <vuePropModel></vuePropModel>
                        <!-- <a-box color="tomato" depth="2" height="4" width="0.5" position="0 0 0"></a-box> -->
                    </a-nft>
                    <a-entity camera></a-entity>
                </div>

                <div v-if="arjs_LocationTrackingBool" id="arjs_LocationTracking">
                    <a-text
                        id="locationExample"
                        value="This content will always face you."
                        look-at="[gps-camera]"
                        scale="1 1 1"
                     
                    ></a-text>
                    <a-camera gps-camera rotation-reader> </a-camera>
                </div>

                <div v-if="arjs_MarkerTrackingBool" id="arjs_MarkerTracking">
                    <a-marker preset="hiro">
                        <a-entity id="propModel"></a-entity>
                        <!-- <vuePropModel></vuePropModel> -->
                        <!-- <a-entity
                        position="0 -1 0"
                        scale="0.05 0.05 0.05"
                        gltf-model="https://arjs-cors-proxy.herokuapp.com/https://raw.githack.com/AR-js-org/AR.js/master/aframe/examples/image-tracking/nft/trex/scene.gltf"
                        ></a-entity> -->
                    </a-marker>
                    <a-entity camera></a-entity>
                </div>
            </a-scene>
        </div>

        <div v-if="mindar_bool" id="mindAR">
            <a-scene id="mindar_scene" mindar="imageTargetSrc: MindAR_assets/card.mind; showStats: false; autoStart: false" color-space="sRGB" renderer="colorManagement: true, physicallyCorrectLights" vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false">
                <a-assets>
                    <img id="card" src="MindAR_assets/card.png" />
                    <a-asset-item id="avatarModel" src="MindAR_assets/softmind/scene.gltf"></a-asset-item>
                </a-assets>

                <a-camera position="0 0 0" look-controls="enabled: false"></a-camera>

                <a-entity mindar-image-target="targetIndex: 0">
                    <a-plane src="#card" position="0 0 0" height="0.552" width="1" rotation="0 0 0"></a-plane>

                    <a-gltf-model rotation="0 0 0 " position="0 0 0.1" scale="0.005 0.005 0.005" src="#avatarModel"
                    animation="property: position; to: 0 0.1 0.1; dur: 1000; easing: easeInOutQuad; loop: true; dir: alternate"/>
                </a-entity>
            </a-scene>
        </div>
    </div>
</template>

<script>
import Vue from 'vue'
import * as THREE from "three";
import { ARButton } from '../assets/ARButton.js';
import PlaneSVG from '../components/PlaneAnim.vue';
import * as ARnft from '@kalwalt/ar-nft'



Vue.config.ignoredElements = [
  'a-scene',
  'a-entity',
  'a-camera',
  'a-nft',
  'a-marker',
  'a-text',
  'a-box',
  'a-assets',
  'a-asset-item',
  'a-plane',
  'a-gltf-model'
];

export default {
    name: "ArButton",
    components:{
        PlaneSVG
    },
    props:[
        "engine",
        "tracking",
        "model",
        "coords"
    ],
    data(){
        return{
            ar_camera: null,
            ar_scene: null,
            ar_renderer: null,

            // Immersive WebXR
            webxr_controller: null,
            webxr_container: null,
            webxr_reticle: null,
            webxr_reticleBool: true,
            webxr_hitTestSource: null,
            webxr_hitTestSourceRequested: false,
            webxr_session: null,
            webxr_rootOverlay: null,
            // Ar.js
            arjs_bool: false,
            arjs_imageTrackingBool: false,
            arjs_LocationTrackingBool: false,
            arjs_MarkerTrackingBool: false,
            arjs_addModelTimer: null,
            // MindAR
            mindar_bool: false,
            mindar_started: false,

            input_scale: {
                current: 1,
                max: 3,
                min: 0.1
            },
            input_rotation:{
                current: 0,
                max: 6.28319,
                min: -6.28319
            },
            input_current_mod: 'scale'
        }
    },
    methods:{
        init(){
            if(this.engine == 'Arjs'){  
                this.arjs_bool = true;

                let _this = this;

                if(this.arjs_LocationTrackingBool){
                    setTimeout(function(){ 
                        let el = document.querySelector('#locationExample')
                        el.setAttribute('gps-entity-place', 'latitude:' + _this.coords.lat + ", longitude:" + _this.coords.lng + ";") 
                    }, 500);
                }


                if(this.arjs_LocationTrackingBool || this.arjs_MarkerTrackingBool || this.arjs_ImageTrackingBool){
                    setTimeout(function(){ 
                        var groupObject3D = document.querySelector('a-entity');
                        // let el = document.querySelector('#propModel');
                        // el.setAttribute('geometry', )
                        console.log(groupObject3D)
                    }, 3000);
                }

                // this.arjs_addModelTimer = setInterval(function(){ 
                //     // var scene = document.querySelector('#arjs_scene');
                //     // if (scene.hasLoaded) {
                //     //     run();
                //     // } else {
                //     //     scene.addEventListener('loaded', this.addModel);
                //     // }
                //     // AFRAME.registerComponent('vuePropModel', {
                //     //     init: function () {
                //     //         var scene = this.el.sceneEl.object3D;  // THREE.Scene
                //     //         scene.add(_this.model)
                //     //     }
                //     // });
                // }, 500);
               
            }else if(this.engine == "ArNFT"){
                this.initArNFT();
                document.querySelector('#enterAR').style.display = "none";
            }else if(this.engine == "MindAR"){
                document.querySelector("#mindAR").style.display = "block";

                const sceneEl = document.querySelector('#mindar_scene');
                const arSystem = sceneEl.systems['mindar-system'];

                if (!this.mindar_started) {
                    arSystem.start();
                    document.querySelector('#enterAR').style.display = "none";
                    this.mindar_started = true;
                } else {
                    arSystem.stop();
                }
            }
        },
        initWebXR(){	
            this.ar_scene = new THREE.Scene();

            this.ar_camera = new THREE.PerspectiveCamera( 70, window.innerWidth / window.innerHeight, 0.01, 20 );

            var light = new THREE.HemisphereLight( 0xffffff, 0xbbbbff, 1 );
            light.position.set( 0.5, 1, 0.25 );
            this.ar_scene.add( light );

            //

            this.ar_renderer = new THREE.WebGLRenderer( { antialias: true, alpha: true } );
            this.ar_renderer.setPixelRatio( window.devicePixelRatio );
            this.ar_renderer.setSize( window.innerWidth, window.innerHeight );
            this.ar_renderer.xr.enabled = true;
            this.ar_renderer.domElement.style.display = "none";
            this.webxr_container.appendChild(this.ar_renderer.domElement);

            //
            var button = ARButton.createButton( this.ar_renderer, true, this.webxr_rootOverlay, {  optionalFeatures: ["dom-overlay"], domOverlay:{root: this.webxr_rootOverlay }, requiredFeatures: [ 'hit-test' ] } ); 
            this.webxr_container.appendChild( button );

            //
            this.webxr_controller = this.ar_renderer.xr.getController( 0 );
            this.webxr_controller.addEventListener( 'select', this.webxr_onSelect );
            this.ar_scene.add( this.webxr_controller );

            this.webxr_reticle = new THREE.Mesh(
                new THREE.RingBufferGeometry( 0.15, 0.2, 32 ).rotateX( - Math.PI / 2 ),
                new THREE.MeshBasicMaterial()
            );

            this.webxr_reticle.matrixAutoUpdate = false;
            this.webxr_reticle.visible = false;
            this.ar_scene.add( this.webxr_reticle );
        },
        webxr_onSelect() {
            if ( this.webxr_reticle.visible ) {
                if(this.model != null){
                    this.model.position.setFromMatrixPosition( this.webxr_reticle.matrix );
                    this.ar_scene.add( this.model );
                }else{
                    console.error("UNDEFINED MODEL")
                }
            }
        },
        webxr_Animate(){
            this.ar_renderer.setAnimationLoop( this.webxr_Render );
        },
        webxr_Render( timestamp, frame ) {
				if ( frame ) {
					const referenceSpace = this.ar_renderer.xr.getReferenceSpace();
					this.webxr_session = this.ar_renderer.xr.getSession();

					if ( this.webxr_hitTestSourceRequested === false ) {

                        var _this = this;

						this.webxr_session.requestReferenceSpace( 'viewer' ).then( function ( referenceSpace ) {

							_this.webxr_session.requestHitTestSource( { space: referenceSpace } ).then( function ( source ) {

								_this.webxr_hitTestSource = source;

							} );

						} );

						this.webxr_session.addEventListener( 'end', function () {

							this.webxr_hitTestSourceRequested = false;
							this.webxr_hitTestSource = null;

						} );

						this.webxr_hitTestSourceRequested = true;

					}

					if ( this.webxr_hitTestSource ) {

                        const hitTestResults = frame.getHitTestResults( this.webxr_hitTestSource );

						if ( hitTestResults.length ) {

                            document.querySelector("#searchingDiv").style.display = "none";
                            document.querySelector("#modelControllers").style.display = "block";
            
							const hit = hitTestResults[ 0 ];

                            this.webxr_reticle.visible = this.webxr_reticleBool;
                            
                            
							this.webxr_reticle.matrix.fromArray( hit.getPose( referenceSpace ).transform.matrix );

						} else {
                            document.querySelector("#searchingDiv").style.display = "block";
                            document.querySelector("#modelControllers").style.display = "none";

                            this.webxr_reticle.visible = false;
						}
					}
				}

				this.ar_renderer.render( this.ar_scene, this.ar_camera);
		},    
        webxr_toggleReticle(){
            this.webxr_reticleBool = !this.webxr_reticleBool;
        },
        initArNFT(){
            let _this = this;
            ARnft.ARnft.init(640, 480, "ArNFT_assets/DataNFT/pinball", 'ArNFT_assets/config.json', false)
			.then((nft) => {
               _this.model.scale.set(180,180,180);
				nft.add(_this.model);
			}).catch((error) => {
				console.log("asadas",error);
			});
        },    
        onWindowResize(){
            if(this.engine == "WebXR"){
                this.ar_camera.aspect = window.innerWidth / window.innerHeight;
                this.ar_camera.updateProjectionMatrix();
                this.ar_renderer.setSize( window.innerWidth, window.innerHeight );
            }
        },
        scaleInput(){
            let input = document.querySelector("#inputController");
            input.setAttribute('max', this.input_scale.max);
            input.setAttribute('min', this.input_scale.min);
            input.setAttribute('value', this.input_scale.current);
            this.input_current_mod = 'scale';
        },
        rotationInput(){
            let input = document.querySelector("#inputController");
            input.setAttribute('max', this.input_rotation.max);
            input.setAttribute('min', this.input_rotation.min);
            input.setAttribute('value', this.input_rotation.current);
            this.input_current_mod = 'rotation';
        },
        inputValue(e){
            this["input_" + this.input_current_mod].current = e.target.value;
            if(this.input_current_mod == "scale"){
                this.model.scale.setScalar(this["input_" + this.input_current_mod].current);
            }else{
                this.model.rotation.y = this["input_" + this.input_current_mod].current;
            }
        },
        addModel(){

        }
    },
    created(){
        console.log("Engine: ", this.engine, "Tracking:", this.tracking);
        if(this.engine == "MindAR"){
            process.env.AR_ENGINE = 'mindar';
            this.mindar_bool = true;
        }

        if(this.engine == 'Arjs'){  
            if(this.tracking == "Image"){
                process.env.AR_ENGINE = 'arjs_image';
                this.arjs_imageTrackingBool = true;
            }else if(this.tracking == "Marker"){
                process.env.AR_ENGINE = 'arjs_marker';
                this.arjs_MarkerTrackingBool = true;
            }else if(this.tracking == "Location"){
                process.env.AR_ENGINE = 'arjs_location';
                this.arjs_LocationTrackingBool = true;       
            }
        }
    },
    mounted(){
        this.webxr_container = document.querySelector("#ar_container");
        this.webxr_rootOverlay = document.querySelector("#overlay");
        window.addEventListener( 'resize', this.onWindowResize, false );

        if(this.engine == "WebXR"){
            document.querySelector("#enterAR").style.display = "none";
            this.initWebXR();
            this.webxr_Animate();
        }
    },
    beforeDestroy(){
        window.removeEventListener( 'resize', this.onWindowResize, false );
        if( this.webxr_controller != null){
            this.webxr_controller.removeEventListener( 'select', this.webxr_onSelect);
        }
        if( this.webxr_session != null){
            this.webxr_session.removeEventListener( 'end', function () {
                this.webxr_hitTestSourceRequested = false;
                this.webxr_hitTestSource = null;
            });
        }
    }
}
</script>

<style scoped>
    #ar_container{
        width: 100px;
    }
    #overlay{
        display: none;
    }

    #arjs-aframe{
        display: none;
    }

    a-scene{
        width: 100vw;
        height: 100vh;
    }
    /* SEARCHING OVERLAY */
    #searchingDiv{
        width: 100vw; 
        height: 100vh;
    }
    #searchingPlaneSVG{
        position: relative;
        width: 65vw;
        left: calc(30vw - 35px);
        top: calc(45vh - 13px);
    }
    #searchingText{
        position: relative;
        width: 65vw;
        left: calc(30vw - 35px);
        top: calc(45vh - 13px);
        font-size: 20px;
        font-weight: bold;
        color: rgb(97,100,117);
    }

    /* CONTROLS UI */
    #modelControllers{
        width: 100vw;
        height: 100vh;
    }
    #modelButtons{
        position: fixed;
        width: 40px;
        /* height: 135px; */
        margin-right: 20px;
        top: calc(40vh - 60px);
        right: 0;
    }
    #inputController{
        position: absolute;
        width: calc(100vw - 40%);
        height: 20px;
        left: 20%;
        bottom: 50px;
    }
    .btn{
		padding: 12px 6px;
        margin-bottom: 5px;
		border:2px solid rgb(97,100,117);
		border-radius:4px;
		background:rgba(0,0,0,0.2);
		color:rgb(97,100,117);
		font:normal 20px sans-serif;
		text-align:center;
		opacity:0.5;
		outline:none;
		z-index:2147483647;
    }
    .btn :hover{
        opacity:1;
    }
    .sizeEnter{
        width: 150px
    }
    /* INPUT UI */
    input[type="range"]{
        -webkit-appearance: none;
        width: 100%;
        height: 8px;
        outline: none !important;
        appearance:none;
        border:none;
        border-radius:30px;
    }
    input[type="range"]::-moz-focus-outer {
        border: 0;
    }
    input[type="range"]:hover {
        outline:none;
    }
    /* Chrome */
    input[type="range"]::-webkit-slider-thumb {
        -webkit-appearance: none;
        appearance: none;
        width: 30px;
        height: 30px;
        background: #4CAF50;
        cursor: pointer;
        border-radius:30px;
        outline:none;
    }
    /* Moz */
    input[type="range"]::-moz-range-thumb {
        width: 30px;
        height: 30px;
        background: #f1f9f4;
        cursor: pointer;
        border-radius:50%;
        border:none;
        outline:none;
    }
    input[type="range"]::-moz-range-progress {
        background-color: #fff;
        height: 100%;
        border-radius:30px;
        border:none;
    }
    input[type="range"]::-moz-range-track {  
        background-color: #ccc;
        border-radius:30px;
        border:none;
        height: 100%;
    }
    /* IE*/
    input[type="range"]::-ms-fill-lower {
        background-color: #fff;
        height: 100%;
        border-radius:30px;
        border:none;
    }
    input[type="range"]::-ms-fill-upper {  
        background-color: #ccc;
        border-radius:30px;
        border:none;
        height: 100%;
    }
</style>
<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted} from "vue"
import * as dat from "dat.gui"
import { OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import * as THREE from "three"

var controls


let canvasRef = ref();

//Utilizziamo i valori del viewport per impostare la grandezza del canvas
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}

//Aggiungiamo l'event listner per il resize del canvas
window.addEventListener("resize", ()=>{
  //update sizes for the canvas
  sizes.width = window.innerWidth
  sizes.height = window.innerHeight

  //update the camera aspect ratio
  camera.aspect = sizes.width / sizes.height
  camera.updateProjectionMatrix()

  //update renderer
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})


//HANDLE FULL SCREEN (Il codice aggiuntivo è per permettere il funzionamento corretto anche con browser Safari)
window.addEventListener("dblclick",()=>{

  const fullscreenElement = document.fullscreenElement ||  document.webkitFullscreenElement
  if(!fullscreenElement){
    if(canvasRef.value.requestFullscreen){
      canvasRef.value.requestFullscreen()
    }
    else if(canvasRef.value.webkitRequestFullscreen){
      canvasRef.value.webkitRequestFullscreen()
    }
    
  }
  else{
    if(document.exitFullscreen)
    {
      document.exitFullscreen()
    }
    else if(document.webkitExitFullscreen){
      document.webkitExitFullscreen()
    }
  }
})

let scene = new THREE.Scene()

let renderer

//Create a GUI for controlling the property
const gui = new dat.GUI()

//LOADING THINGS
const textureLoader = new THREE.TextureLoader()
const cubeTextureLoader = new THREE.CubeTextureLoader()

const textureScaglia = textureLoader.load("/texture3.png")
const textureGradient = textureLoader.load("/bw-gradient.jpg")

//---------------MATERIALS------------------------------------------------------//


//Mesh Basic Material : The most basic material

let basicMaterial = new THREE.MeshBasicMaterial({
  //map: textureScaglia //for appling a texture to a mesh, you have to use map  (puoi anche applicare la text dopo scrivendo semplicemente =>   material.map = texture)
})

/*

//for the color is different, we can set the color as "red" when we declare the material, but after instanciating, we can t use material.color="red" but  material.color = new THREE.Color("red") or rgb or hexadecimal )

// You can combine the map and the color, the color applies on the texture like a color "mask"
basicMaterial.color= new THREE.Color("red");


//You have access to the wireframe to see the inner structure of the mesh
//basicMaterial.wireframe = true

//You can control the opacity
basicMaterial.opacity = 0.5
//but remember..it need the transparent = true to work!
basicMaterial.transparent = true

//We can use the alphamap: where is black is hidden , what is white is visible
basicMaterial.alphaMap = textureScaglia

//For the plane surface mesh, we see only 1 side by default, the other one is invisible, but we can use this property to see it both side (REMEMBER IT ALTO NEED MATERIAL.TRANSPARENT = TRUE TO WORK)
basicMaterial.side = THREE.DoubleSide
// there are other 2 properties for taking one side at time:
// - THREE.FrontSide  for the front side of the mesh (the default)
// - THREE.BackSide for the backside of the mesh

*/




// Mesh Normal Material

const normalMaterial = new THREE.MeshNormalMaterial()

/*

//The Mesh Normal Material => NORMALS are information that contains the direction of the outside of the face

//Normals can be used for lighning, reflection, refraction ecc.
//Normal also shares common properties with basicMaterial like: wireframe, transparent, opacity, and side,
//but there is also a flatShading property

normalMaterial.flatShading = true

//flatshading will flatten the faces , because the normals won't be interpolated between the vertices

*/





// Mesh Matcap Material 

/* 

// will display a color by using the normals as a reference to pick the right color on a texture that looks like a sphere

const matcapMaterial = new THREE.MeshMatcapMaterial()
matcapMaterial.matcap = textureScaglia

*/




// Mesh Depth Material

//will simply color the geometry in white if it's close to the near and in black if it's close to the far value of the camera

// const depthMaterial = new THREE.MeshDepthMaterial()



//OTHER MATERIALS REQUIRE LIGHTS, SO LET'S DECLARE IT
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)

const pointLight = new THREE.PointLight(0xffffff, 0.5)
pointLight.position.set(2,3,4)

scene.add(ambientLight, pointLight)


// Mesh Lamber Material 

//The first materials that react to light
const lambertMaterial = new THREE.MeshLambertMaterial()

//lamber material is performant but we can see strange pattern on the geometry.
//lambert material have new properties related to the light, but we explore this properties in detail next, when use another material that react to light





// Mesh Phong Material
const phongMaterial = new THREE.MeshPhongMaterial()

/* 

//like the lambert material, but without the strane line pattern on the geometry

//With the phong material you can also see the reflection on the light on the geometry surface
//the only issue is that it's less performant than lamber, but for now that's not a problem

//We Start to see some of the properties of this material that react to light

// We can control the light reflection with shininess
phongMaterial.shininess = 100

//and cthe color of this reflextion with specular
phongMaterial.specular = new THREE.Color(0x1188ff)

*/





// Mesh Toon Material


const toonMaterial = new THREE.MeshToonMaterial()

/* 

//this one is similar to the lamber material but with a "cartoon" effect

//we can "control" the shadow providing gradient 

//toonMaterial.gradientMap = textureGradient

//but we lost the cartonish effect , because the magFilter find that the image is smal and try to fix it stretching, and bluring it with mipmapping
//(It works best if the base has 1 pixel for each color of the gradient, or so)

//but we can prevent this using the Nearest Filter on the texture
textureGradient.minFilter = THREE.NearestFilter //if you are not sure what of the 2 use for your case, you can use both
textureGradient.magFilter = THREE.NearestFilter

//because we use the nearest so we can deactivate the mipmapping
textureGradient.generateMipmaps = false

*/





// Mesh Standard Material

const standardMaterial = new THREE.MeshStandardMaterial()

//This material is the best one (depending on what you want do)

//Standard material uses physically based rendering principles (PBR) 
// It support Light (like lambert and phong) but with a more realistic algoritghm and a better parameters like:
// metalness and roughness

standardMaterial.metalness = 0.7
standardMaterial.roughness = 0.2
standardMaterial.map = textureScaglia

//standard material also have access to aoMap (Ambient Occlusion Map) (the one that add shadow in the curveses and angle of the image on a texture)

gui.add(standardMaterial, "metalness").min(0).max(1).step(0.0001) //tells the ui => what object, the name of the property to tweak, a min, a max, and a step (precisio)
gui.add(standardMaterial, "roughness").min(0).max(1).step(0.0001)






// Mesh Physical Material

/* 
const physicalMaterial = new THREE.MeshPhysicalMaterial()
physicalMaterial.map = textureScaglia
//the same as the standard material, but with a support of a clear coat effect (imita l'effetto di un rivestimento trasparente/lucido su una superficie materiale)
//so use this only if you need this clear coat effect

*/



// THERE ARE ALSO OTHER MATERIALS THAT WE SEE BETTER IN ACTION IN NEXT LESSONS


// Mesh Points Material => you can use points material for creating particles

// Shader Material & Raw Shader Material => used to create your own material



//ENVIRONMENT MAPS 

/*

//Environmen maps are image that sorrund the scene, and can be used for lightning and reflaction
//Currentyly threejs only support CUBE environment maps

//To Load a cube Texture we need to use a CubeTextureLoader (that we have already create at the top of the script section)
//and we need to privide 6 photos for render the cube (one for each face of the cube), in the correct order

const environmentMapTexture = cubeTextureLoader.load(["Positive-x.jpg", "Negative-x.jpg","Positive-y.jpg", "Negative-y.jpg", "Positive-z.jpg","Negative-z.jpg",])

// And then assign it to the envMap property of the material

standardMaterial.envMap = environmentMapTexture;

*/




//SPHERE
const sphere = new THREE.Mesh(
  new THREE.SphereBufferGeometry(0.5,16,16),
  standardMaterial
)
sphere.position.x -= 1.5

//PLANE
const plane = new THREE.Mesh(
  new THREE.PlaneBufferGeometry(1,1),
  standardMaterial
)

//TORUS
const torus = new THREE.Mesh(
  new THREE.TorusBufferGeometry(0.5,0.2,16,32),
  standardMaterial
)
torus.position.x += 1.5

scene.add(sphere, plane, torus)

// CAMERA
let camera = new THREE.PerspectiveCamera(
  75, 
  sizes.width / sizes.height, 
  0.1,
  100  
);

camera.position.set(0,0,3)
scene.add(camera);

const clock = new THREE.Clock()

const animate = () =>{
  const elapsedTime = clock.getElapsedTime()

  //Update (rotate) objects
  sphere.rotation.y = 0.1 * elapsedTime
  plane.rotation.y = 0.1 * elapsedTime
  torus.rotation.y = 0.1 * elapsedTime

  sphere.rotation.x = 0.1 * elapsedTime
  plane.rotation.x = 0.1 * elapsedTime
  torus.rotation.x = 0.1 * elapsedTime


  controls.update()
  
  renderer.render(scene, camera)
}

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });
 
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  //è possivile che veriamo un po di imperfezioni sul render, soprattutto sugli angoli
  //il pixel ratio corrisponde a quanti pixel fisici hai sul tuo schermo, per un unita di pixel del software
  renderer.setPixelRatio(Math.min(window.devicePixelRatio,2)) // in questo modo il max valore di pixel ratio utilizzaro è 2 (altrimenti devi renderizzare troppi pixel, dispiego enorme di potenza gpu)
  renderer.render(scene, camera)
  renderer.setAnimationLoop(animate)

  

  //CONTROLS
  controls = new OrbitControls(camera, canvasRef.value)
  controls.enableDamping = true; // permette uno effetto smooth una volta che rilasciamo l'elemento, rallenta fino a fermarsi
});

onUnmounted(() => {
  renderer.setAnimationLoop(null)
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>
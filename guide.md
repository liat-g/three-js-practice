here is my guide to wrong akram's three cubes tutorial:

steps for this project: npm install create-react-app
npm install react-three-fiber drei three
react-three-fiber is just threejs for react apps
change app.css to app.scss (sass) and remove the contents of App.js
import Canvas into app.js file -- html cannot be written within the Canvas tag, which is what renders the 3d images to the dom
Canvas only takes three js elements
using the mesh tag inside the canvas tag, we define the geometery (what shape it is) and the material (how it looks and how it will interact with the environment)
using boxBufferGeometry we define a box and use the attach and args properties. args property takes the size of what we are working with
like so:
<Canvas>
<mesh>
<boxBufferGeometry attach='geometry' args={[1, 1, 1]} />
<meshStandardMaterial attach='material' />
</mesh>
</Canvas>

or if we want a circle:
<Canvas>
<mesh>
<circleBufferGeometry attach='geometry' args={[1, 200]} />
<meshStandardMaterial attach='material' />
</mesh>
</Canvas>

to make the shape move, we need to use useFrame from react-three-fiber and require useRef from React

then, to get the box to render, we move it to its own component and render the name of that component in the Canvas Tag:

const Box = () => {
const mesh = useRef(null)
useFrame(() => (mesh.current.rotation.x = mesh.current.rotation.y += 0.01))
return(
<mesh ref={mesh}>
<boxBufferGeometry attach='geometry' args={[1, 1, 1]} />
<meshStandardMaterial attach='material' />
</mesh>
)
}

---

    <>
      <Canvas>
      <Box />
      </Canvas>
    </>

to change the light: we can use the ambientLight tag like so:

<ambientLight intensity={0.3}>

you can add the colorManagement property to the Canvas tag to make the colors more like what they actually are like so:

 <Canvas colorManagement>

you can also change the camera angle with the camera properties's fov and position args

      <Canvas colorManagement camera={{position: [-5, 2, 10], fov: 60}}>

to make mulitple boxes, we need to reuse the box component we made. To give them all different positions, we have to destructure the position property and pass it through the mesh tag:

const SpinningMesh = ({position}) => {
const mesh = useRef(null)
useFrame(() => (mesh.current.rotation.x = mesh.current.rotation.y += 0.01))
return(
<mesh ref={mesh} position={position}>
<boxBufferGeometry attach='geometry' args={[1, 1, 1]} />
<meshStandardMaterial attach='material' color='pink'/>
</mesh>
)
}

we can also destructure and pass color, args and position as props through the SpinningMesh component so that we can reuse it with different colors.

const SpinningMesh = ({position, args, color}) => {
const mesh = useRef(null)
useFrame(() => (mesh.current.rotation.x = mesh.current.rotation.y += 0.01))
return(
<mesh ref={mesh} position={position}>
<boxBufferGeometry attach='geometry' args={args} />
<meshStandardMaterial attach='material' color={color}/>
</mesh>
)
}

returned:
<Canvas colorManagement camera={{position: [-5, 2, 10], fov: 60}}>
<ambientLight intensity={.3}/>
<SpinningMesh position={[0, 1, 0]} args={[3, 2, 1]} color='lightblue'/>
<SpinningMesh position={[-2, 1, -5]} color="pink"/>
<SpinningMesh position={[5, 1, -2]} color="pink"/>
</Canvas>

to add point light add the self closing point tag <pointLight /> a pointlight is a light source that comes from the direction specified in position based on the {[x, y, z]} coordinate.

To cast shadows, we use directionalLight, which is a self closing tag as well. <directionalLight />

for directional light, we need to specify where the light is coming from like so:

        <directionalLight
        position={[0, 10, 0]}
        intensity={.3}
        shadow-mapSize-width={1024}
        shadow-mapSize-height={1024}
        shadow-camera-far={50}
        shadow-camera-left={-10}
        shadow-camera-right={10}
        shadow-camera-top={10}
        shadow-camera-bottom={-10}
        />

to cast shadows, we'll need a floor or a plane to do so.
to start off, we will need to make a new group and attach a shadow material that represents a 'floor', something that shadow can hit.

        <group>
          <mesh rotation={[-Math.PI / 2, 0, 0]} position={[0, -3, 0]}>
            <planeBufferGeometry attach='geometry' args={[100, 100]} />
            <shadowMaterial attach='material' />
          </mesh>

        </group>

    here the shadowMaterial tag is the material for which we will be making the shadow.
    We also need to add 'shadows' to our canvas.

we need to pass castShadows to mesh returned from our spinningBox component like so:
<mesh castShadow ref={mesh} position={position}>
we also need to pass it to the directionalLight as a prop
and we need to pass it to our shadow material mesh as receiveShadow
<group>
<mesh receiveShadow rotation={[-Math.PI / 2, 0, 0]} position={[0, -3, 0]}>
<planeBufferGeometry attach='geometry' args={[100, 100]} />
<shadowMaterial attach='material' opacity={0.2}/>
</mesh>
</group>

to use softShadows, we simply import drei like so: import { softShadows } from '@react-three/drei/' (this is the current way to do it)
and initialize it at the top of the document like so:
softShadows();

to expand the objects when clicked we will be using react-spring:
we import useSpring and a (which stands for animation) into from react-spring like so:
import {useSpring, a} from 'react-spring/three'

then, we need to pass a state in order to expand the shapes. we create an expand and setExpand state in our return:

const [expand, setExpand] = useState(false)

and then we also create a props variable to hold our useSpring animation like so:
const props = useSpring({
scale: expand ? [1.4, 1.4, 1.4] : [1, 1, 1]
})

if its expaneded, its scaled by 1.4 else it stays at its current size

then we have to turn our mesh into an animated mesh by adding a. infront of it:

         <a.mesh onClick={() => setExpand(!expand)} scale={props.scale} receiveShadow rotation={[-Math.PI / 2, 0, 0]} position={[0, -3, 0]}>
            <planeBufferGeometry attach='geometry' args={[100, 100]} />
            <shadowMaterial attach='material' opacity={0.2}/>
          </a.mesh>

and we add an onClick handler to our meshes to that they expand when they are clicked and we pass the scale prop to it as well.

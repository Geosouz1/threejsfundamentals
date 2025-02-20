Title: Three.js Primitives
Description: A tour of three.js primitives.
TOC: Primitives

This article is one in a series of articles about three.js.
The first article was [about fundamentals](threejs-fundamentals.html).
If you haven't read that yet you might want to start there.

Three.js has a large number of primitives. Primitives
are generally 3D shapes that are generated at runtime
with a bunch of parameters.

It's common to use primitives for things like a sphere
for a globe or a bunch of boxes to draw a 3D graph. It's
especially common to use primitives to experiment
and get started with 3D. For the majority of 3D apps
it's more common to have an artist make 3D models
in a 3D modeling program. Later in this series we'll
cover making and loading data from several 3D modeling
programs. For now let's go over some of the available
primitives.

<div data-primitive="BoxBufferGeometry">A Box</div>
<div data-primitive="CircleBufferGeometry">A flat circle</div>
<div data-primitive="ConeBufferGeometry">A Cone</div>
<div data-primitive="CylinderBufferGeometry">A Cylinder</div>
<div data-primitive="DodecahedronBufferGeometry">A dodecahedron (12 sides)</div>
<div data-primitive="ExtrudeBufferGeometry">An extruded 2d shape with optional bevelling.
Here we are extruding a heart shape. Note this is the basis
for <code>TextBufferGeometry</code> and <code>TextGeometry</code> respectively.</div>
<div data-primitive="IcosahedronBufferGeometry">An icosahedron (20 sides)</div>
<div data-primitive="LatheBufferGeometry">A shape generated by spinning a line. Examples would lamps, bowling pins, candles, candle holders, wine glasses, drinking glasses, etc... You provide the 2d silhouette as series of points and then tell three.js how many subdivisions to make as it spins the silhouette around an axis.</div>
<div data-primitive="OctahedronBufferGeometry">An Octahedron (8 sides)</div>
<div data-primitive="ParametricBufferGeometry">A surface generated by providing a function that takes a 2D point from a grid and returns the corresponding 3d point.</div>
<div data-primitive="PlaneBufferGeometry">A 2D plane</div>
<div data-primitive="PolyhedronBufferGeometry">Takes a set of triangles centered around a point and projects them onto a sphere</div>
<div data-primitive="RingBufferGeometry">A 2D disc with a hole in the center</div>
<div data-primitive="ShapeBufferGeometry">A 2D outline that gets triangulated</div>
<div data-primitive="SphereBufferGeometry">A sphere</div>
<div data-primitive="TetrahedronBufferGeometry">A tetrahedron (4 sides)</div>
<div data-primitive="TextBufferGeometry">3D text generated from a 3D font and a string</div>
<div data-primitive="TorusBufferGeometry">A torus (donut)</div>
<div data-primitive="TorusKnotBufferGeometry">A torus knot</div>
<div data-primitive="TubeBufferGeometry">A circle traced down a path</div>
<div data-primitive="EdgesGeometry">A helper object that takes another geometry as input and generates edges only if the angle between faces is greater than some threshold. For example if you look at the box at the top it shows a line going through each face showing every triangle that makes the box. Using an <code>EdgesGeometry</code> instead the middle lines are removed.</div>
<div data-primitive="WireframeGeometry">Generates geometry that contains one line segment (2 points) per edge in the given geometry. Without this you'd often be missing edges or get extra edges since WebGL generally requires 2 points per line segment. For example if all you had was a single triangle there would only be 3 points. If you tried to draw it using a material with <code>wireframe: true</code> you would only get a single line. Passing that triangle geometry to a <code>WireframeGeometry</code> will generate a new geometry that has 3 lines segments using 6 points..</div>

You might notice of most of them come in pairs of `Geometry`
or `BufferGeometry`. The difference between the 2 types is effectively flexibility
vs performance.

`BufferGeometry` based primitives are the performance oriented
types. The vertices for the geometry are generated directly
into an efficient typed array format ready to be uploaded to the GPU
for rendering. This means they are faster to start up
and take less memory but if you want to modify their
data they take what is often considered more complex
programming to manipulate.

`Geometry` based primitives are the more flexible, easier to manipulate
type. They are built from JavaScript based classes like `Vector3` for
3D points, `Face3` for triangles.
They take quite a bit of memory and before they can be rendered three.js will need to
convert them to something similar to the corresponding `BufferGeometry` representation.

If you know you are not going to manipulate a primitive or
if you're comfortable doing the math to manipulate their
internals then it's best to go with the `BufferGeometry`
based primitives. If on the other hand you want to change
a few things before rendering you might find the `Geometry`
based primitives easier to deal with.

As an simple example a `BufferGeometry`
can not have new vertices easily added. The number of vertices used is
decided at creation time, storage is created, and then data for vertices
are filled in. Whereas for `Geometry` you can add vertices as you go.

We'll go over creating custom geometry in another article. For now
let's make an example creating each type of primitive. We'll start
with the [examples from the previous article](threejs-responsive.html).

Near the top let's set a background color

```js
const scene = new THREE.Scene();
+scene.background = new THREE.Color(0xAAAAAA);
```

This tells three.js to clear to lightish gray.

The camera needs to change position so that we can see all the
objects.

```js
-const fov = 75;
+const fov = 40;
const aspect = 2;  // the canvas default
const near = 0.1;
-const far = 5;
+const far = 1000;
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
-camera.position.z = 2;
+camera.position.z = 120;
```

Let's add a function, `addObject`, that takes an x, y position and an `Object3D` and adds
the object to the scene.

```js
const objects = [];
const spread = 15;

function addObject(x, y, obj) {
  obj.position.x = x * spread;
  obj.position.y = y * spread;

  scene.add(obj);
  objects.push(obj);
}
```

Let's also make a function to create a random colored material.
We'll use a feature of `Color` that lets you set a color
based on hue, saturation, and luminance.

`hue` goes from 0 to 1 around the color wheel with
red at 0, green at .33 and blue at .66. `saturation`
goes from 0 to 1 with 0 having no color and 1 being
most saturated. `luminance` goes from 0 to 1
with 0 being black, 1 being white and 0.5 being
the maximum amount of color. In other words
as `luminance` goes from 0.0 to 0.5 the color
will go from black to `hue`. From 0.5 to 1.0
the color will go from `hue` to white.

```js
function createMaterial() {
  const material = new THREE.MeshPhongMaterial({
    side: THREE.DoubleSide,
  });

  const hue = Math.random();
  const saturation = 1;
  const luminance = .5;
  material.color.setHSL(hue, saturation, luminance);

  return material;
}
```

We also passed `side: THREE.DoubleSide` to the material.
This tells three to draw both sides of the triangles
that make up a shape. For a solid shape like a sphere
or a cube there's usually no reason to draw the
back sides of triangles as they all face inside the
shape. In our case though we are drawing a few things
like the `PlaneBufferGeometry` and the `ShapeBufferGeometry`
which are 2 dimensional and so have no inside. Without
setting `side: THREE.DoubleSide` they would disappear
when looking at their back sides.

I should note that it's faster to draw when **not** setting
`side: THREE.DoubleSide` so ideally we'd set it only on
the materials that really need it but in this case we
are not drawing too much so there isn't much reason to
worry about it.

Let's make a function, `addSolidGeometry`, that
we pass a geometry and it creates a random colored
material via `createMaterial` and adds it to the scene
via `addObject`.

```js
function addSolidGeometry(x, y, geometry) {
  const mesh = new THREE.Mesh(geometry, createMaterial());
  addObject(x, y, mesh);
}
```

Now we can use this for the majority of the primitives we create.
For example creating a box

```js
{
  const width = 8;
  const height = 8;
  const depth = 8;
  addSolidGeometry(-2, -2, new THREE.BoxBufferGeometry(width, height, depth));
}
```

If you look in the code below you'll see a similar section for each type of geometry.

Here's the result:

{{{example url="../threejs-primitives.html" }}}

There are a couple of notable exceptions to the pattern above.
The biggest is probably the `TextBufferGeometry`. It needs to load
3D font data before it can generate a mesh for the text.
That data loads asynchronously so we need to wait for it
to load before trying to create the geometry. By promisifiying 
font loading we can make it mush easier.
We create a `FontLoader` and then a function `loadFont` that returns
a promise that on resolve will give us the font. We then create
an `async` function called `doit` and load the font using `await`.
And finally create the geometry and call `addObject` to add it the scene.

```js
{
  const loader = new THREE.FontLoader();
  // promisify font loading
  function loadFont(url) {
    return new Promise((resolve, reject) => {
      loader.load(url, resolve, undefined, reject);
    });
  }

  async function doit() {
    const font = await loadFont('resources/threejs/fonts/helvetiker_regular.typeface.json');  /* threejsfundamentals: url */
    const geometry = new THREE.TextBufferGeometry('three.js', {
      font: font,
      size: 3.0,
      height: .2,
      curveSegments: 12,
      bevelEnabled: true,
      bevelThickness: 0.15,
      bevelSize: .3,
      bevelSegments: 5,
    });
    const mesh = new THREE.Mesh(geometry, createMaterial());
    geometry.computeBoundingBox();
    geometry.boundingBox.getCenter(mesh.position).multiplyScalar(-1);

    const parent = new THREE.Object3D();
    parent.add(mesh);

    addObject(-1, -1, parent);
  }
  doit();
}
```

There's one other difference. We want to spin the text around its
center but by default three.js creates the text such that its center of rotation
is on the left edge. To work around this we can ask three.js to compute the bounding
box of the geometry. We can then call the `getCenter` method
of the bounding box and pass it our mesh's position object.
`getCenter` copies the center of the box into the position.
It also returns the position object so we can call `multiplyScalar(-1)`
to position the entire object such that its center of rotation
is at the center of the object.

If we then just called `addSolidGeometry` like with previous
examples it would set the position again which is
no good. So, in this case we create an `Object3D` which
is the standard node for the three.js scene graph. `Mesh`
is inherited from `Object3D` as well. We'll cover [how the scene graph
works in another article](threejs-scenegraph.html).
For now it's enough to know that
like DOM nodes, children are drawn relative to their parent.
By making an `Object3D` and making our mesh a child of that
we can position the `Object3D` wherever we want and still
keep the center offset we set earlier.

If we didn't do this the text would spin off center.

{{{example url="../threejs-primitives-text.html" }}}

Notice the one on the left is not spinning around its center
whereas the one on the right is.

The other exceptions are the 2 line based examples for `EdgesGeometry`
and `WireframeGeometry`. Instead of calling `addSolidGeometry` they call
`addLineGeomtry` which looks like this

```js
function addLineGeometry(x, y, geometry) {
  const material = new THREE.LineBasicMaterial({color: 0x000000});
  const mesh = new THREE.LineSegments(geometry, material);
  addObject(x, y, mesh);
}
```

It creates a black `LineBasicMaterial` and then creates a `LineSegments`
object which is a wrapper for `Mesh` that helps three know you're rendering
line segments (2 points per segment).

Each of the primitives has several parameters you can pass on creation
and it's best to [look in the documentation](https://threejs.org/docs/) for all of them rather than
repeat them here. You can also click the links above next to each shape
to take you directly to the docs for that shape.

One other thing that's important to cover is that almost all shapes
have various settings for how much to subdivide them. A good example
might be the sphere geometries. Spheres take parameters for
how many divisions to make around and how many top to bottom. For example

<div class="spread">
<div data-diagram="SphereBufferGeometryLow"></div>
<div data-diagram="SphereBufferGeometryMedium"></div>
<div data-diagram="SphereBufferGeometryHigh"></div>
</div>

The first sphere has 5 segments around and 3 high which is 15 segments
or 30 triangles. The second sphere has 24 segments by 10. That's 240 segments
or 480 triangles. The last one has 50 by 50 which is 2500 segments or 5000 triangles.

It's up to you to decide how many subdivisions you need. It might
look like you need a high number of segments but remove the lines
and the flat shading and we get this

<div class="spread">
<div data-diagram="SphereBufferGeometryLowSmooth"></div>
<div data-diagram="SphereBufferGeometryMediumSmooth"></div>
<div data-diagram="SphereBufferGeometryHighSmooth"></div>
</div>

It's now not so clear that the one on the right with 5000 triangles
is entirely better than the one in the middle with only 480.
If you're only drawing a few spheres, like say a single globe for
a map of the earth, then a single 10000 triangle sphere is not a bad
choice. If on the otherhand you're trying to draw 1000 spheres
then 1000 spheres times 10000 triangles each is 10 million triangles.
To animate smoothly you need the browser to draw at 60 frames per
second so you'd be asking the browser to draw 600 million triangles
per second. That's a lot of computing.

Sometimes it's easy to choose. For example you can also choose
to subdivide a plane.

<div class="spread">
<div data-diagram="PlaneBufferGeometryLow"></div>
<div data-diagram="PlaneBufferGeometryHigh"></div>
</div>

The plane on the left is 2 triangles. The plane on the right
is 200 triangles. Unlike the sphere there is really no trade off in quality for most
use cases of a plane. You'd most likely only subdivide a plane
if you expected to want to modify or warp it in some way. A box
is similar.

So, choose whatever is appropriate for your situation. The less
subdivisions you choose the more likely things will run smoothly and the less
memory they'll take. You'll have to decide for yourself what the correct
tradeoff is for your particular siutation.

Next up let's go over [how three's scene graph works and how
to use it](threejs-scenegraph.html).

<canvas id="c"></canvas>
<link rel="stylesheet" href="resources/threejs-primitives.css">
<script type="module" src="resources/threejs-primitives.js"></script>


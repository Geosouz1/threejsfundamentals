Title: Three.js WebVR
Description: How to use Virtual Reality in Three.js.
TOC: WebVR - Basics

Making WebVR apps in three.js is pretty simple. You basically just have to tell
three.js you want to use WebVR. If you think about it a few things about WebVR
should be clear. Which way the camera is pointing is supplied by the VR system
itself since the user turns their head to choose a direction to look. Similarly
the field of view and aspect will be supplied by the VR system since each system
has a different field of view and display aspect.

Let's take an example from the article on [making a responsive
webpage](threejs-responsive.html) and make it support VR.

Before we get started you're going to need a VR capable device like an Android
smartphone, Google Daydream, Oculus Go, Oculus Rift, Vive, Samsung Gear VR, an
iPhone with a [WebVR browser](https://itunes.apple.com/us/app/webvr-browser/id1286543066?mt=8).

Next, if you are running locally you need to run a simple web server like is
covered in [the article on setting up](threejs-setup.html). 

You then need to look up the *local* IP address of your computer. On Mac you can
Option-Click the WiFi controls (assuming you're on Wifi). 

<div class="threejs_center"><img src="resources/images/ipaddress-macos.png" style="width: 428px;"></div>

On Windows you can
[click the Wifi icon in the taskbar and then click "Properties" for a particular
network connection](https://support.microsoft.com/en-us/help/4026518/windows-10-find-your-ip-address).

If you see more than one IP address, local IP addresses most commonly start with
`10.` or `192.` or `172.`. If you see an address that starts with `127.` that
address is for your computer only. It's the address the computer can use to talk
to itself so ignore anything that starts with `127.`.

On your VR device, first make sure it's connected to the same WiFi as your
computer. Then, open your browser and type in `http://<ipaddress:port>`. Which
port depends on what server you're running. If you're using
[Servez](https://greggman.github.io/servez) or the http-server from node.js then
the port will likely be 8080.

On my computer as I write this article its IP address is `192.168.100.78` and my
web server is using port 8080 so on my VR device I'd enter
`http://192.168.100.78:8080` to get the main page of my webserver to appear. If
you've downloaded these articles and are serving them you should see the front
page of this website appear. If you're making a new page then you might need to
add the path to your page for example when entering the URL on your VR device
something like `http://192.168.100.78:8080/path/to/page.html`

Note that anytime you switch networks your local ip address will change.
Even on the same network when you re-connect to it your local ip address
might change so you will need to look it up again and type a different
address into your VR device.

Also note that this will likely work on your home network or your work network but
may likely **not work** at a cafe. The WiFi at many cafe's, especially at large
chains like Starbucks or McDonalds are configured so that machines on the local
network can not talk to each other.

If you're really going to do WebVR development another thing you should learn about is
[remote debugging](https://developers.google.com/web/tools/chrome-devtools/remote-debugging/)
so that you can see console warnings, errors, and of course actually 
[debug your code](threejs-debugging-javascript.html).

If you just want to see the code work below you can just run the code from
this site.

The first thing we need to do is include the VR support after
including three.js

```js
import * as THREE from './resources/three/r108/build/three.module.js';
+import {WEBVR} from './resources/threejs/r108/examples/jsm/vr/WebVR.js';
```

Then we need to enable three.js's WebVR support and add its
WebVR button to our page

```js
function main() {
  const canvas = document.querySelector('#c');
  const renderer = new THREE.WebGLRenderer({canvas});
+  renderer.vr.enabled = true;
+  document.body.appendChild(WEBVR.createButton(renderer));
```

We need to not try to resize when in VR mode as the VR device
will decide the size

```js
-if (resizeRendererToDisplaySize(renderer)) {
+if (!renderer.vr.isPresenting() && resizeRendererToDisplaySize(renderer)) {
```

The last major thing we need to do is let three.js run our
render loop. Until now we have used a `requestAnimationFrame`
loop but to support VR we need to let three.js handle our
render loop for us. We can do that by calling `WebGLRenderer.setAnimationLoop`
and passing a function to call for the loop.

```js
function render(time) {
  time *= 0.001;

  if (!renderer.vr.isPresenting() && resizeRendererToDisplaySize(renderer)) {
    const canvas = renderer.domElement;
    camera.aspect = canvas.clientWidth / canvas.clientHeight;
    camera.updateProjectionMatrix();
  }

  cubes.forEach((cube, ndx) => {
    const speed = 1 + ndx * .1;
    const rot = time * speed;
    cube.rotation.x = rot;
    cube.rotation.y = rot;
  });

  renderer.render(scene, camera);

-  requestAnimationFrame(render);
}

-requestAnimationFrame(render);
+renderer.setAnimationLoop(render);
```

You'd think that would be it but there is one more detail.
In VR just like we don't control the field of view and direction
the camera is looking we also don't control where the camera
starts. At least not as of three.js r105. The camera is hardcoded
to default to `x = 0`, `y = 1.6`, `z = 0`

Before we had the cubes along the X axis at Y and Z = 0.
Let's move them to be in front of the default VR camera.

```js
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

cube.position.x = x;
+cube.position.y = 1.6;
+cube.position.z = -2;
```

We set them to `z = -2` since the camera will now be at `z = 0` and
cameras default to looking down the -z axis.

Let's also remove setting our camera's position since three.js
will override it anyway.

```js
const fov = 75;
const aspect = 2;  // the canvas default
const near = 0.1;
const far = 5;
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
-camera.position.z = 2;
```

This brings up an extremely important point. **Units in VR are in meters**.
In other words **One Unit = One Meter**. This means the camera is 1.6 meters above 0.
The cube's centers are 2 meters in front of the camera. Each cube
is 1x1x1 meter large. This is important because VR needs to adjust things to the
user *in the real world*. That means we need the units used in three.js to match
the user's on movements.

And with that we should get 3 spinning cubes in front
of the camera with a button to enter WebVR.

{{{example url="../threejs-webvr-basic.html" }}}

I find that VR works better if we have something surrounding the camera like
room for reference so let's add a simple grid cubemap like we covered in
[the article on backgrounds](threejs-backgrounds.html). We'll just use the same grid
texture for each side of the cube which will give as a grid room.

```js
const scene = new THREE.Scene();
+{
+  const loader = new THREE.CubeTextureLoader();
+  const texture = loader.load([
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+    'resources/images/grid-1024.png',
+  ]);
+  scene.background = texture;
+}
```

That's better.

{{{example url="../threejs-webvr-basic-w-background.html" }}}

Note: To actually see WebVR you will need a WebVR compatible device.
I believe most Android Phones can support WebVR using Chrome or Firefox.
For iOS you might be able to use this [WebVR App](https://itunes.apple.com/us/app/webvr-browser/id1286543066?mt=8)
though in general WebVR support on iOS is unsupported as of May 2019.

To use WebVR on Android or iPhone you'll need a *VR Headset*
for phones. You can get them for anywhere from $5 for one made of cardboard
to $100. Unfortunately I don't know which ones to recommend. I've purchased
6 of them over the years and they are all of varying quality. I've
never paid more than about $25.

Just to mention some of the issues

1. Do they fit your phone

   Phones come in a variety of sizes and so the VR headsets need to match.
   Many headsets claim to match a large variety of sizes. My experience
   is the more sizes they match the worse they actually are since instead
   of being designed for a specific size they have to make compromises
   to match more sizes. Unfortunately multi-size headsets are the most common type.

2. Can they focus for your face

   Some devices have more adjustments than others. Generally there
   are at most 2 adjustments. How far the lenses are from your eyes
   and how far apart the lenses are.

3. Are they too reflective

   Many headsets of a cone of plastic from your eye to the phone.
   If that plastic is shinny or reflective then it will act like
   a mirror reflecting the screen and be very distracting.

   Few if any of the reviews seem to cover this issue.

4. Are the comfortable on your face.

   Most of the devices rest on your nose like a pair of glasses.
   That can hurt after a few minutes. Some have straps that go around
   your head. Others have a 3rd strap that goes over your head. These
   may or may not help keep the device at the right place.

   It turns out for most (all?) devices, you eyes need to be centered
   with the lenses. If the lenses are slightly above or below your
   eyes the image gets out of focus. This can be very frustrating
   as things might start in focus but 45-60 seconds later the device
   has shifted up or down 1 millimeter and you suddenly realize you've
   been struggling to focus on a blurry image.

5. Can they support your glasses.

   If you wear eye glasses then you'll need to read the reviews to see
   if a particular headset works well with eye glasses.

I really can't make any recommendations unfortunately. [Google has some
cheap recommendations made from cardboard](https://vr.google.com/cardboard/get-cardboard/)
some of them as low as $5 so maybe start there and if you enjoy it
then consider upgrading. $5 is like the price of 1 coffee so seriously, give it try!

There are also 3 basic types of devices.

1. 3 degrees of freedom (3dof), no input device

   This is generally the phone style although sometimes you can
   buy a 3rd party input device. The 3 degrees of freedom
   mean you can look up/down (1), left/right(2) and you can tilt
   your head left and right (3).

2. 3 degrees of freedom (3dof) with 1 input device (3dof)

   This is basically Google Daydream and Oculus GO

   These also allow 3 degrees of freedom and include a small
   controller that acts like a laser pointer inside VR.
   The laser pointer also only has 3 degrees of freedom. The
   system can tell which way the input device is pointing but
   it can not tell where the device is.

3. 6 degrees of freedom (6dof) with input devices (6dof)

   These are *the real deal* haha. 6 degrees of freedom
   means not only do these device know which way you are looking
   but they also know where your head actually is. That means
   if you move from left to right or forward and back or stand up / sit down
   the devices can register this and everything in VR moves accordingly.
   It's spookily and amazingly real feeling. With a good demo
   you'll be blown away or at least I was and still am.

   Further these devices usually include 2 controllers, one
   for each hand and the system can tell exactly where your
   hands are and which way they are oriented and so you can
   manipulate things in VR by just reaching out, touching,
   pushing, twisting, etc...

   6 degree of freedom devices include the Vive and Vive Pro,
   the Oculus Rift and Quest, and I believe all of the Windows MR devices.

With all that covered I don't for sure know which devices will work with WebVR.
I'm 99% sure that most Android phones will work when running Chrome. You may
need to turn on VR support in [`about:flags`](about:flags). I also know Google
Daydream will also work and similarly you need to enable VR support in
[`about:flags`](about:flags). Oculus Rift, Vive, and Vive Pro will work via
Chrome or Firefox. I'm less sure about Oculus Go and Oculus Quest as both of
them use custom OSes but according to the internet they both appear to work.

Okay, after that long detour about VR Devices and WebVR
there's some things to cover

* Supporting both VR and Non-VR

  AFAICT, at least as of r105, there is no easy way to support
both VR and non-VR modes with three.js. With VR enabled but not in VR mode
the camera's position is hard coded to `0, 1.6, 0`. Ideally
if not in VR mode you'd be able to control the camera using
whatever means you want, for example the `OrbitControls`,
and you'd get some kind of event when switching into and
out of VR mode so that you could turn the controls on/off.

  If three.js adds some support to do both I'll try to update
this article. Until then you might need 2 versions of your 
site OR pass in a flag in the URL, something like

```
https://mysite.com/mycooldemo?allowvr=true
```

Then we could add some links in to switch modes

```html
<body>
  <canvas id="c"></canvas>
+  <div class="mode">
+    <a href="?allowvr=true" id="vr">Allow VR</a>
+    <a href="?" id="nonvr">Use Non-VR Mode</a>
+  </div>
</body>
```

and some CSS to position them

```css
body {
    margin: 0;
}
#c {
    width: 100vw;
    height: 100vh;
    display: block;
}
+.mode {
+  position: absolute;
+  right: 1em;
+  top: 1em;
+}
```

in your code you could use that parameter like this

```js
function main() {
  const canvas = document.querySelector('#c');
  const renderer = new THREE.WebGLRenderer({canvas});
-  renderer.vr.enabled = true;
-  document.body.appendChild(WEBVR.createButton(renderer));

  const fov = 75;
  const aspect = 2;  // the canvas default
  const near = 0.1;
  const far = 5;
  const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);

+  const params = (new URL(document.location)).searchParams;
+  const allowvr = params.get('allowvr') === 'true';
+  if (allowvr) {
+    renderer.vr.enabled = true;
+    document.body.appendChild(WEBVR.createButton(renderer));
+    document.querySelector('#vr').style.display = 'none';
+  } else {
+    // no VR, add some controls
+    camera.position.y = 1.6;
+    const controls = new OrbitControls(camera, canvas);
+    controls.target.set(0, 1.6, -2);
+    controls.update();
+    document.querySelector('#nonvr').style.display = 'none';
+  }
```

Whether that's good or bad I don't know. I have a feeling the differences
between what's needed for VR and what's needed for non-VR are often
very different so for all but the most simple things maybe 2 separate pages
are better? You'll have to decide.

Note for various reasons this will not work in the live editor
on this site so if you want to check it out 
<a href="../threejs-webvr-basic-vr-optional.html" target="_blank">click here</a>.
It should start in non-VR mode and you can use the mouse or fingers to move
the camera. Clicking "Allow VR" should switch to allow VR mode and you should
be able to click "Enter VR" if you're on a VR device.

* Deciding on the level of VR support

  Above we covered 3 types of VR devices. 

  * 3DOF no input
  * 3DOF + 3DOF input
  * 6DOF + 6DOF input

  You need to decide how much effort you're willing to put in
  to support each type of device.

  For example the simplest device has no input. The best you can
  generally do is make it so there are some buttons or objects in the user's view
  and if the user aligns some marker in the center of the display
  on those objects for 1/2 a second or so then that button is clicked.
  A common UX is to display a small timer will appear over the object indicating
  if you keep the marker there for a moment the object/button will be selected.

  Since there is no other input that's about the best you can do

  The next level up you have one 3DOF input device. Generally it
  can point at things and the user has at least 2 buttons. The Daydream
  also has a touchpad which provides normal touch inputs.

  In any case if a user has this type of device it's far more
  comfortable for the user to by able to point at things with
  their controller than it is to make them do it with their
  head by looking at things.

  A similar level to that might be 3DOF or 6DOF device with a
  game console controller. You'll have to decide what to do here.
  I suspect the most common thing is the user still has to look
  to point and the controller is just used for buttons.

  The last level is a user with a 6DOF headset and 2 6DOF controllers.
  Those users will find an experience that is only 3DOF to often
  be frustrating. Similarly they usually expect to be able to 
  virtually manipulate things with their hands in VR so you'll
  have to decide if you want to support that or not.

As you can see getting started in VR is pretty easy but
actually making something shippable in VR will require
lots of decision making and design.

This was a pretty brief intro into WebVR with three.js. We'll
cover some of the input methods in future articles.


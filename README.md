# aframe-gltf-morph-component
An a-frame component that allows you to target and control a `gltf` model's morphTargets created in [Blender](https://www.blender.org/).

![Gltf-morph in action](https://github.com/elbobo/aframe-gltf-morph-component/blob/master/puffer.gif?raw=true)

morphTargets are exported as part of `gltf` models when shape keys have been created in Blender. Here is a summary... 
> Shape keys are modifications of the original mesh that you can animate and mix with each other. Previously called Relative Vertex Keys (RVK), one of their major uses is to create facial expressions. The idea behind Shape keys is simple. The basic, undeformed mesh you start with is the "Basis" shape. You can add a variety of different versions of this shape and store each one as a Shape Key for use in an animation (other software sometimes refers to these as "morph targets").
(taken from [here](https://en.wikibooks.org/wiki/Blender_3D:_Noob_to_Pro/Advanced_Tutorials/Advanced_Animation/Guided_tour/Mesh/Shape))

As well as using them for animations though, a combination of of shape keys used in different ways is a good way to be able to use the same model but with many possible variations and degrees of those variations resulting in a set of fundamentally similar but discrete objects/characters.

# Basic usage

To simply target and effect a morph target, add the component as you would any other. Specific the name of the morph and the value that you want to set it to (0-1)

```html
  <head>
    <script src="https://aframe.io/releases/0.8.2/aframe.min.js"></script>
    <script src="Please copy the script from inside the dist folder or from the bottom of this page, name it and put that name here"></script>
  </head>
  <body>
    <a-scene>
       <a-assets>
          <a-asset-item 
              id="puffer_model" 
              src="path/to/yourmodel.glb">
          </a-asset-item>
       </a-assets>
       <a-entity 
        gltf-model="#puffer_model"
        gltf-morph="morphtarget:Puffed up;value:1">
      </a-entity>
    </a-scene>
  </body>
```
If there are multiple morphTargets in the model, they can be distinguished with double underscore (as with other components)

```html
   <a-entity 
    gltf-model="#puffer_model"
    gltf-morph__puff="morphtarget:Puffed up;value:1"
    gltf-morph__spike="morphtarget:Spikes out;value:1">
  </a-entity>
```
morphTargets can also be animated using the [animation component](https://github.com/supermedium/superframe/tree/master/components/animation) although in most cases it might make more sense to export the animations from Blender and trigger them using the [animation-mixer component](https://github.com/donmccurdy/aframe-extras/tree/master/src/loaders). (You need to add the morph target component you want to animate on to the entity before you can animate it).

```html
   <a-entity 
    gltf-model="#puffer_model"
    gltf-morph__puff="morphtarget:Puffed up;value:0"
    animation="property:gltf-morph__puff.value;from:0;to:0.75">
  </a-entity>
```
And of course you can target and/or set the `gltf-morph` with js as part of another component

```javascript
AFRAME.registerComponent('addamorph',{
  
  init: function(){
    
    this.el.setAttribute('gltf-morph', 'morphtarget:Puffed up;value:0.5')
    
  }
  
})
```

And using such an approach (or simply in HTML) you can see how you could for example, use the same model, but tweaking different morphTargets produce a variety of objects/characters. Example of a family of weird puffer-like fish generated from the same model below.

![alt text](https://github.com/elbobo/aframe-gltf-morph-component/blob/master/variations.png?raw=true "Using one model with many morphTargets we can create a range of similar but distinct entities")

# API

Property | Description | Default
--- | --- | ---
morphtarget | Name of morphTarget, can be found as part of the `GLTF` model | ''
value | Value that you want to set that morphTarget to (0 - 1) | 0


# Notes/Caveats

If you have a model and are not sure if there are morphTargets available, or don't know what they're called. I recommend using [Don McCurdy's GLTF viewer](https://gltf-viewer.donmccurdy.com/) where you can see the available morphTargets and adjust them.

I only know how to use Blender where morphTargets are produced by exporting a model with `shape keys`. I'm afraid I don't know if exports are the same from other packages or whether you could target the morphTargets from other model file types. 

There are many tutorials on both Blender and creating `shape keys` online but [here is a good place to start](https://www.youtube.com/watch?v=n0VspDUOErE) for all basic types of animation in Blender, specific discussion of Key shapes [starts here](https://youtu.be/n0VspDUOErE?t=737).

Feedback welcome as ever.

# The code

Without rawgit I don't know how to provide a direct link to the code. For now, please find below to code to copy below (Its not that much). Alternatively you can take it from the `dist` folder.

```javascript
AFRAME.registerComponent('gltf-morph', {

  multiple: true,

  schema:{
    morphtarget: {type: 'string', default: ''},
    value: {type: 'number', default: 0},
  },//end schema

  init: function(){

    //make sure object3D is set at first
    this.el.addEventListener('object3dset', () =>{ 
      
      //run morpher function
      this.morpher()

    });

  },//end init
  
  update: function(){
    
    //run morpher function
    this.morpher()
    
  },//end update

  morpher: function(){
    
    //get the mesh
    var mesh = this.el.object3D
    
    //traverse it looking for morphtargets and targetnames
    mesh.traverse((o) => {

      //if morphtargets exist & targetNames exist
      if ( o.morphTargetInfluences && o.userData.targetNames ){

        //store the array position of the passed targetname
        var pos = o.userData.targetNames.indexOf(this.data.morphtarget);

        //target that morphtarget and update value
        o.morphTargetInfluences[pos] = this.data.value

      }//end if

    });//end traverse
    
  },//end morpher

})//end morph
```









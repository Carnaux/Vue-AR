# Vue-AR

This is a button component for enabling AR experiences for e-commerce websites created with Vue.js.

The project is a work in progress and it is open source and customizable, so if you think something can be better, you can change it yourself. Pull Requests are all welcome!

# Work in progress

The usage of Ar.js and MindAR are still being worked on, they don't work right now, updates will be released soon.

# Repository

The main branch has everything you need to add to make this component work.

And the dev branch has the Vue app that was used to develop, it may be usefull as a template for you.

# Featured libraries:

### Immersive WebXR 
    https://immersive-web.github.io/
### Ar.js
    https://ar-js-org.github.io/AR.js-Docs/
### ArNFT
    https://github.com/webarkit/ARnft
### MindAr
    https://github.com/hiukim/mind-ar-js

If you find any issue related to tracking/funcionalities, please, refer to the libraries github pages to solve it.

# Structure

- public
    - ArNFT_assets
        - ALL NECESSARY CONTENTS FOR ArNFT TO WORK
    - MindAR_assets
        - ALL NECESSARY CONTENTSFOR MindAR TO WORK
    - index.html (VUE TEMPLATE)
- src
    - assets
        - ARButton.js (FOR WebXR)
    - components
        - ARButton.vue (THE MAIN COMPONENT)
        - PlaneAnim.vue (FOR WebXR)


# Installation
1. Copy the contents of the 'publicFolder' to your Vue's public folder
2. Copy the contents of the 'src' to your Vue's src folder

If you will use ArNFT library:

3. Install the module using:  ```npm i @kalwalt/ar-nft```

# Usage

```<ArButton :engine="''"></ArButton>```

## Attributes

 - engine (STRING - REQUIRED)
    - 'WebXR'
    - 'Arjs'
    - 'ArNFT'
    - 'MindAR'
 - tracking (STRING - ONLY REQUIRED FOR Arjs)
    - 'Image'
        - NFT tracking
    - 'Marker'
        - Hiro patter
    - 'Location'
        - Geolocalization 
 - model (THREE.JS MESH - REQUIRED)
 - coords (OBJECT - REQUIRED FOR LOCATION TRACKING)

### Soon the 'marker' attribute will be added, so you can dinamically set the image or pattern markers.


## Snippets

### WebXR

```<ArButton :engine="'WebXR'" :model="'model'"></ArButton>```

### Ar.js

#### Image

```<ArButton :engine="'Arjs'" :tracking="'Image'" :model="'model'"></ArButton>```

#### Marker
```<ArButton :engine="'Arjs'" :tracking="'Marker'" :model="'model'"></ArButton>```

#### Location
```<ArButton :engine="'Arjs'" :tracking="'Location'" :model="'model'" :coords="'{lat: 0, lng: 0}'"></ArButton>```

### ArNFT

```<ArButton :engine="'ArNFT'" :model="'model'"></ArButton>```

### MindAR

```<ArButton :engine="'Mindar'" :model="'model'"></ArButton>```

# More points that you should know

There are some specific modifications to make each library work the way you want, the (README)[] on the dev branch explains more about it.
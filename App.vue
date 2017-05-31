<template>
  <div class="app">
  <div class="header">
    <img alt="Halftone.me" src="logo.svg" class="logo"/>
    <div class="search">
      <input type="text" v-model="url" placeholder="Paste an image url"/>
      <button class="btn bttn bttn-fill bttn-royal" v-on:click="getRandom">ImgurRr...Random!</button>
    </div>
  </div>
  <div v-if="url" class="result">
    <halftone :url="url" :web="web"></halftone>
    <div class="cmds">
      <button class="bttn bttn-fill" v-on:click="download()">Download</button>
    </div>
  </div>
  <div v-else="url" class="columns tile is-multiline is-ancestor">
    <div v-for="item in svgs" :class="getRandomTileClass()">
      <div class="tile is-child card">
        <div class="card-content">
          <figure class="image">
            <img :src="item" />
          </figure>
        </div>
      </div>
    </div>
  </div>
  </div>
</template>

<style scoped>
 .tile.is-ancestor{
   text-align: center;
 }
 .tile.is-parent{
   margin: 0 auto !important;
 }

</style>

<script>
 import Halftone from './Halftone.vue'
 import * as firebase from 'firebase';

 var config = {
   apiKey: "AIzaSyDg9DCsszf1x4H-1hJ4K9s2LGtyyatwhwI",
   authDomain: "halftone-856ed.firebaseapp.com",
   databaseURL: "https://halftone-856ed.firebaseio.com",
   projectId: "halftone-856ed",
   storageBucket: "halftone-856ed.appspot.com",
   messagingSenderId: "230534234164"
 }

 firebase.initializeApp(config)

 var storage = firebase.storage().ref()
 var db = firebase.database()
 var ref = db.ref('/').child("svg").limitToLast(20)

 function rand(x, y) {
   return Math.floor(Math.random() * (x - y + 1) + y)
 }

 /* var _url = window.location.href.indexOf('#') > -1 ?
  *            window.location.href.split('#')[1] :
  *            "http://i.imgur.com/" + getRandomString() + "m.jpg";*/
 export default {
   data(){
     return {
       logo: './logo.svg',
       url: null,
       web: false,
       svgs: []
     }
   },
   components: {
     halftone: Halftone
   },
   mounted(){
     this.start = 5

     ref.on("value", (snapshot) => {
       var svgs = []
       snapshot.forEach((svg) => {
         svgs.push(svg.val());
       });
       this.svgs = svgs.reverse();
     }, (errorObject) => {
       console.log("The read failed: " + errorObject.code);
     });

     window.addEventListener('scroll', (event) => {
       var element = document.body;
       if (element.scrollHeight - element.scrollTop < window.innerHeight + 200){
          this.start += 5
          db.ref('svg').limitToLast(this.start).on("child_added", (dataSnapshot) => {
            this.svgs.push(dataSnapshot.val())
          });
       }
     });
   },
   watch: {
     url(){
       if(this.timeout) clearTimeout(this.timeout);
       if(this.url){
         this.timeout = setTimeout(()=>{
           var item = db.ref('svg').push().key;
           storage.child(item+'.svg')
                  .putString(this.getSvgBinary(), 'data_url')
                  .then(function(snapshot){
                    db.ref('svg').child(item).set(snapshot.downloadURL);
                  });
         }, 2000);
       }
     }
   },
   methods: {
     getRandomString(){
       var char, out = '';
       for (var i=0; i < 5; i++) {
         switch (rand(0, 4)) {
           case 1:
             char = rand(47, 58);
             break;
           case 2:
             char = rand(64, 91);
             break;
           case 3:
             char = rand(96, 123);
             break;
         }
         out += String.fromCharCode(char);
       }
       return out
     },
     getRandom(){
       this.url = "http://i.imgur.com/" + this.getRandomString() + "m.jpg";
     },
     getRandomTileClass(){
       return `column tile is-parent is-${rand(8,12)}`;
     },
     getSvgBinary(){
       var $svg = document.querySelector('svg')
       var svg = document.createElement('svg')
       var sizes = $svg.getAttribute('viewBox').split(' ');
       svg.innerHTML = $svg.innerHTML
       svg.setAttribute('xmlns', 'http://www.w3.org/2000/svg')
       svg.setAttribute('version', '1.1')
       svg.setAttribute('width', sizes[2])
       svg.setAttribute('height', sizes[3])
       var b64 = btoa(unescape(encodeURIComponent(svg.outerHTML)))
       return 'data:image/svg+xml;base64,'+b64
     },
     download(){
       var evt = new MouseEvent('click', {
         view: window,
         bubbles: false,
         cancelable: true
       });
       var a = document.createElement('a');
       a.setAttribute('download', 'Halftone-'+new Date()+'.svg');
       a.setAttribute('href', this.getSvgBinary());
       a.setAttribute('target', '_blank');
       a.dispatchEvent(evt);
     }
   }
 }
</script>

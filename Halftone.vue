<template>
  <div class="v-halftone">
    <svg v-if="img" :viewBox="getViewBox()" preserveAspectRatio="xMinYMid slice">
      <g v-if="web" stroke="#000" fill="none">
        <g v-for="item, lum in items">
          <path :stroke-width="lum" stroke-linecap="round" :d="getPathData(item)" />
        </g>
      </g>
      <g v-else stroke="none" fill="#000">
        <g v-for="item, lum in items">
          <circle v-for="circle in item" :cx="circle.x * 3" :cy="circle.y * 3" :r="lum * 1.2" />
        </g>
      </g>
    </svg>
  </div>
</template>
<script>
 export default {

   props: { url: {default: 'http://i.imgur.com/jx3ggSRb.jpg'}, web: {default: true} },

   data(){
     return {
       width: 0,
       height: 0,
       items: [],
       img: null,
       w: {
         width: window.innerWidth,
         height: window.innerHeight
       }
     }
   },

   mounted(){
     if(!this.url) return false
     this.fetchImage()
   },

   watch: {
     url(){
       this.fetchImage()
     }
   },

   methods: {

     fetchImage(){
       let img = document.createElement('img')
       img.crossOrigin = "Anonymous";
       img.onload = ()=>{
         var canvas = document.createElement('canvas');
         var context = canvas.getContext('2d');
         canvas.width = 100;
         canvas.height = 100 * img.height / img.width;
         this.width = canvas.width
         this.height = canvas.height
         context.drawImage(img, 0, 0, this.width, this.height );
         var imgData = context.getImageData(0, 0, this.width, this.height);
         this.img = imgData
         this.items = this.getBrightnessData( imgData )
       }
       img.src = this.url
     },

     getViewBox(){
       let width = this.img.width
       let height = this.img.height
       if(this.web === false){
         return `-0.5 -0.5 ${width*3} ${height*3}`
       }
       return `0 0 ${width} ${height}`
     },

     getPathData(item){
       let path = [];

       for(let k in item){
         path.push(`M${item[k].x} ${item[k].y}z`)
       }
       return path.join('')
     },

     getRgba(data, i){
       return { r: data[i], g: data[i + 1], b: data[i + 2], a: data[i + 3] }
     },

     getLuminosity(rgba){
       return (( rgba.a - (rgba.r + rgba.g + rgba.b) / 3) / 200).toFixed(2)
     },

     isInvisible(rgba){ // Or white
       return (!rgba.a > 0 || (rgba.r === 255 && rgba.g === 255 && rgba.b === 255))
     },

     getBrightnessData(img) {
       let lums = {};
       let data = img.data

       for(let i = 0; i<data.length; i+=4){

         let rgba = this.getRgba(data, i)

         if ( this.isInvisible(rgba) ) continue

         let lum = this.getLuminosity(rgba);
         let x = (i / 4) % img.width
         let y = Math.floor((i / 4) / img.width)

         lums[lum] = lums[lum] || []
         lums[lum].push({ x, y })

       }

       return lums
     }
   }
 }
</script>

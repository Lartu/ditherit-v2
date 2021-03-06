<template>
  <div class="flex flex-col items-center max-w-full pb-8 mx-4">
    <Logo />
    <div v-show="!imageUploaded" class="flex flex-col items-center mt-8">
      <p class="mt-8 text-2xl">An image dithering tool 🏁</p>
      <img class="mt-8" src="~/assets/earth-dither.gif" />
    </div>
    <div
      class="p-y12 px-4 flex flex-col md:flex-row mt-8 w-full justify-center"
    >
      <!-- Begin Main Toolbar -->
      <div
        v-show="imageUploaded"
        class="w-full md:w-1/4 order-last md:order-first"
      >
        <div class="flex-1">
          <ColorPicker
            :initial-palette="rgbQuantOptions.palette"
            @update-palette="onUpdatePalette"
          />

          <div class="mt-4 text-center xl:w-64">
            <button class="btn-red text-lg w-full" @click="ditherImage()">
              🏁 Dither
            </button>
          </div>

          <div class="shadow-lg rounded py-2 px-4 mt-4">
            <h4 class="text-lg font-bold mt-2 mb-2">Options</h4>
            <!-- Image Size Selector -->
            <div class="flex flex-row items-center justify-between">
              <label for="imageSize" class="font-bold">Image Size</label>
              <span
                class="rounded-full h-4 w-4 bg-red-700 text-white flex items-center justify-center float-right text-sm cursor-pointer"
                @click="showOptionsModalSize = !showOptionsModalSize"
              >
                <span v-if="!showOptionsModalSize">
                  ?
                </span>
                <span v-else>
                  X
                </span>
              </span>
            </div>
            <div v-if="!showOptionsModalSize" class="w-full relative">
              <select
                id="imageSize"
                v-model="canvasWidth"
                class="block appearance-none w-full bg-white border border-gray-300 hover:border-gray-500 px-4 py-2 pr-8 rounded leading-tight focus:outline-none focus:shadow-outline"
              >
                <option
                  id="originalSize"
                  name="originalSize"
                  :value="'original'"
                  >Original</option
                >
                <template v-for="(v, i) in imageWidths">
                  <option :id="v" name="imageWidth" :value="v">{{ v }}</option>
                </template>
              </select>
              <div
                class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700"
              >
                <svg
                  class="fill-current h-4 w-4"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 20 20"
                >
                  <path
                    d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"
                  />
                </svg>
              </div>
            </div>
            <div v-else>
              <div class="mt-2 bg-red-100 p-2 rounded">
                This determines the size of the final file, and can also affect how the dither looks.
              </div>
            </div>
            <!-- End Image Size Selector -->
            <!-- Algorithm Selector -->
            <div class="flex flex-row items-center justify-between mt-4">
              <label for="ditherAlgo" class="font-bold">Algorithm</label>
              <span
                class="rounded-full h-4 w-4 bg-red-700 text-white flex items-center justify-center float-right text-sm cursor-pointer"
                @click="showOptionsModalAlgo = !showOptionsModalAlgo"
              >
                <span v-if="!showOptionsModalAlgo">
                  ?
                </span>
                <span v-else>
                  X
                </span>
              </span>
            </div>
            <div
              v-if="!showOptionsModalAlgo"
              class="inline-block relative w-full"
            >
              <select
                id="ditherAlgo"
                v-model="rgbQuantOptions.dithKern"
                class="block appearance-none w-full bg-white border border-gray-300 hover:border-gray-500 px-4 py-2 pr-8 rounded leading-tight focus:outline-none focus:shadow-outline"
              >
                <template v-for="(v, i) in algorithmOptions">
                  <option :id="v" name="imageWidth" :value="v">{{ v }}</option>
                </template>
              </select>
              <div
                class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700"
              >
                <svg
                  class="fill-current h-4 w-4"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 20 20"
                >
                  <path
                    d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"
                  />
                </svg>
              </div>
            </div>
            <div v-else>
              <div class="mt-2 bg-red-100 p-2 rounded">
                These are different ways of spreading the quantization errors
                around. Certain ones might work better than others depending on
                the image.
              </div>
            </div>
            <!-- End Algorithm Selector -->
            <!-- Serpentine Dither -->
            <div class="flex flex-col items-center justify-between mt-4">
              <div class="inline-block relative w-full">
                <input
                  id="ditherSerp"
                  v-model="rgbQuantOptions.dithSerp"
                  type="checkbox"
                  class="form-checkbox"
                />
                <span class="ml-2"><strong>Serpentine Dither</strong></span>
                <span
                  class="rounded-full h-4 w-4 bg-red-700 text-white flex items-center justify-center float-right text-sm cursor-pointer"
                  @click="showOptionsModalSerp = !showOptionsModalSerp"
                >
                  <span v-if="!showOptionsModalSerp">
                    ?
                  </span>
                  <span v-else>
                    X
                  </span>
                </span>
              </div>
              <div
                v-if="showOptionsModalSerp"
                class="inline-block relative w-full"
              >
                <div class="mt-2 bg-red-100 p-2 rounded">
                  This determines if the dithering just goes left to right, top
                  to bottom, or does a snake wiggle.
                </div>
              </div>
            </div>

            <!-- End Serpentine Dither -->
          </div>
        </div>
        <ImageUpload @number-images="getNumberOfImages" @image-upload="onImageUpload" />
      </div>
      <!-- End Toolbar -->
      <div
        class="w-full md:w-3/4 flex flex-col xl:flex-row order-first md:order-last"
      >

        <!-- Begin Main Display -->
        <div class="px-4 flex flex-col flex-1 items-center">
          <!-- Dithered Canvas Display -->
          <div class="max-w-full" v-show="showDitheredImage">
            <div
              v-show="dithering"
              class="flex flex-col justify-center items-center"
            >
              <div class="loader h-20 w-20 mb-4"></div>
              <div class="">Dithering...</div>
            </div>
            <div
              class="flex flex-col justify-center items-center w-full h-full "
              v-show="!dithering && showDitheredImage && !viewOriginal"
            >
              <div
              class="max-w-full "
                v-for="n in this.numberOfImages"
                v-show="selectedImage.id == 'originalImage' + n"
              >
                <canvas :id="'dithered_originalImage' + n" class="max-w-full selectedImage"></canvas>
              </div>
            </div>
          </div>
          <!-- End Dithered Canvas Display -->
          <!-- Original Image Display -->
          
          <div class="w-full flex flex-col ">
            <div v-if="numberOfImages > 0">
              <img class="mx-auto max-w-full" :src="selectedImage.src" v-if="!showDitheredImage || viewOriginal"/>
            </div>
            <div class="flex flex-row justify-between items-center py-2 px-4 mt-8 mx-4 shadow-lg rounded" v-if="showDitheredImage && !dithering"> 
              <Toggler
              @view-original="viewOriginal = !viewOriginal"
              >View Original</Toggler>            
              <a
                class="btn-red-outline inline-block self-center"
                target="_blank"
                :href="downloadUrl"
                :download="'dither_it_' + selectedFile.name"
                @click="downloadImage"
                
                >💾 Download</a
              >  
            </div>

            <div class="flex flex-row flex-wrap mt-4 justify-center">
              <div v-for="n in this.numberOfImages" v-show="numberOfImages > 1">
              
                <img
                  :id="'originalImage' + n"
                  class="max-w-full m-4 shadow cursor-pointer"
                  @click="analyzeImagePalette($event.target)"
                  width="75"
                />
              </div>            
            </div>
          </div>
          <!-- End Original Image Display -->
          <!-- Toolbar Stuff -->
          <div
            class="flex flex-row justify-center"
            :class="{ 'w-full': imageUploaded, 'text-sm': imageUploaded }"
          >
            <div
              v-show="showDitheredImage && !dithering"
              class="text-center mt-4 mr-2 justify-center "
            >

            </div>
            <ImageUpload @number-images="getNumberOfImages" @image-upload="onImageUpload" v-if="!imageUploaded" />
          </div>
          <!-- End Toolbar Stuff -->
        </div>
        <!-- Begin Output -->
        <div
          v-if="imageUploaded"
          class="w-full xl:w-1/4 flex flex-col md:flex-row xl:flex-col mt-8 xl:mt-0 px-8 xl:px-0"
        >
          <div v-if="this.selectedImage"
            class="shadow-lg rounded p-3 m-2 w-full h-0 md:h-auto invisible md:visible"
          >
            <h4 class="text-lg font-bold mt-0" >Selected File</h4>
            <div
              v-if="selectingImage"
              class="flex flex-col justify-center items-center"
            >
              <div class="loader h-8 w-8 my-4 "></div>
            </div>
            <div  v-else class="flex xl:flex-col">
              <div class="w-1/2 lg:w-full">
                <ul class="mt-1">
                  <li>
                    <strong>Size: </strong>{{ selectedImage.naturalWidth }}px x
                    {{ selectedImage.naturalHeight }}px
                  </li>
                  <li>
                    <strong>Filesize: </strong>{{ (Math.round(((selectedImage.src.length) * 3) / 4) / 1000).toFixed(2) }}kb
                  </li>
                  <li>
                    <strong>Filetype: </strong>{{selectedFile.type}}
                  </li>
                </ul>
              </div>
              <!-- <div class="w-1/2 lg:w-full" v-show="showDitheredImage">
                      <h5 class="mt-2 text-md font-bold">Dithered</h5>
                      <ul>
                      <li>Filename: dither_it_{{fileName}} </li>
                      <li>Width: {{canvasWidth}}px</li>
                      <li>Height: {{canvasHeight}}px</li>
                      </ul>                
                  </div> -->
            </div>
          </div>
          <div
            v-if="showDitheredImage"
            class="shadow-lg rounded p-3 m-2 w-full h-0 md:h-auto invisible md:visible"
          >
            <h4 class="text-lg font-bold mt-0">Dithered File</h4>
            <div
              v-if="selectingImage"
              class="flex flex-col justify-center items-center"
            >
              <div class="loader h-8 w-8 my-4"></div>
            </div>
            <div v-else class="flex xl:flex-col">
              <div class="w-1/2 lg:w-full">
                <ul class="mt-1">
                  <li>
                    <strong>Size: </strong>{{ ditheredWidth }}px x
                    {{ ditheredHeight }}px
                  </li>
                  <li>
                    <strong>Filesize: </strong>
                    {{ downloadFileSize.toFixed(2) }}kb
                  </li>

                  <!-- {{originalFileSize}}<br />
                    {{downloadFileSize}} -->
                </ul>
              </div>
            </div>
          </div>
          <div
            v-if="showDitheredImage"
            class="shadow-lg rounded p-3 m-2 w-full h-0 md:h-auto invisible md:visible"
            :class="[ratioGood ? 'bg-green-100' : '', 'bg-red-100']"
          >
            <h4 class="text-lg font-bold mt-0">Filesize Ratio</h4>
            <div
              v-if="selectingImage"
              class="flex flex-col justify-center items-center"
            >
              <div class="loader h-8 w-8 my-4"></div>
            </div>   
            <div v-else>         
              <div v-if="ratioGood">
                <p class="mt-1">
                  The filesize is
                  <strong
                    >{{
                      100 -
                        ((downloadFileSize / (Math.round(((selectedImage.src.length) * 3) / 4) / 1000)) * 100).toFixed(2)
                    }}%</strong
                  >
                  smaller than the original 👍
                </p>
              </div>
              <div v-else>
                <p class="mt-1">
                  The filesize is
                  <strong
                    >{{
                      (( downloadFileSize / (Math.round(((selectedImage.src.length) * 3) / 4) / 1000)) * 100).toFixed(2)
                    }}%</strong
                  >
                  larger than the original 🎭
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <BottomContent />
    <!-- Fathom - simple website analytics - https://usefathom.com -->
    <script>
      ;(function(f, a, t, h, o, m) {
        a[h] =
          a[h] ||
          function() {
            ;(a[h].q = a[h].q || []).push(arguments)
          }
        ;(o = f.createElement('script')),
          (m = f.getElementsByTagName('script')[0])
        o.async = 1
        o.src = t
        o.id = 'fathom-script'
        m.parentNode.insertBefore(o, m)
      })(document, window, '//cdn.usefathom.com/tracker.js', 'fathom')
      fathom('set', 'siteId', 'AHDLJXNJ')
      fathom('trackPageview')
    </script>
    <!-- / Fathom -->
  </div>
</template>

<script>
import RgbQuant from 'rgbquant'
import Logo from '~/components/Logo.vue'
import ImageUpload from '~/components/ImageUpload.vue'
import ColorPicker from '~/components/ColorPicker.vue'
import InputBlock from '~/components/InputBlock.vue'
import BottomContent from '~/components/BottomContent.vue'
import Toggler from '~/components/Toggler.vue'

export default {
  components: {
    ImageUpload,
    ColorPicker,
    Logo,
    InputBlock,
    BottomContent,
    Toggler
  },
  data() {
    return {
      canvasWidth: 'original', // this is what the dithered canvas renders off of, height comes from a computed property
      fileName: '', // the name of the loaded file
      loading: false, // for the loading spinner
      dithering: false, // for the dithering spinner
      switchingImages: false,
      specsCalculated: false, // for the specs
      imageUploaded: false,
      showDitheredImage: false, // control if the dithered image is visible
      imageWidths: [320, 640, 1080, 1280],
      algorithmOptions: [
        'FloydSteinberg',
        'FalseFloydSteinberg',
        'Stucki',
        'Atkinson',
        'Jarvis',
        'Burkes',
        'Sierra',
        'TwoSierra',
        'SierraLite'
      ],
      rgbQuantOptions: {
        // ---- Options ------
        colors: 8, // desired palette size
        method: 2, // histogram method, 2: min-population threshold within subregions; 1: global top-population
        boxSize: [8, 8], // subregion dims (if method = 2)
        boxPxls: 2, // min-population threshold (if method = 2)
        initColors: 4096, // # of top-occurring colors to start with (if method = 1)
        minHueCols: 2000, // # of colors per hue group to evaluate regardless of counts, to retain low-count hues
        dithKern: 'FloydSteinberg', // dithering kernel name, see available kernels in docs below
        dithDelta: 0, // dithering threshhold (0-1) e.g: 0.05 will not dither colors with <= 5% difference
        dithSerp: false, // enable serpentine pattern dithering
        palette: [], // a predefined palette to start with in r,g,b tuple format: [[r,g,b],[r,g,b]...]
        reIndex: false, // affects predefined palettes only. if true, allows compacting of sparsed palette once target palette size is reached. also enables palette sorting.
        useCache: true, // enables caching for perf usually, but can reduce perf in some cases, like pre-def palettes
        cacheFreq: 10, // min color occurance count needed to qualify for caching
        colorDist: 'euclidean' // method used to determine color distance, can also be "manhattan"
      },
      showOptionsModalSize: false,
      showOptionsModalAlgo: false,
      showOptionsModalSerp: false,
      downloadUrl: '', // the url src thing to download the image
      downloadFileSize: '50', // the filesize of the download
      originalFileSize: '', // filesize of the original
      imageType: '',
      numberOfImages: '',
      images: [],
      selectedImage: '',
      ditheredWidth: '',
      ditheredHeight: '',
      viewOriginal: false,
      selectingImage: false
    }
  },
  computed: {
    ratioGood() {
      if (this.downloadFileSize / (Math.round(((this.selectedImage.src.length) * 3) / 4) / 1000) < 1) {
        return true
      } else {
        return false
      }
    },
    selectedFile() {
      return this.images.find(this.getSelected)
    }
  },
  methods: {
    // ---------------------------
    // Set the image id to the id of the currently selected image
    // ----------------------------    
    getSelected(image) {
      console.log('Get selected image.')
      return image.id === this.selectedImage.id;
    },
    // ---------------------------
    // Get the number of images
    // ----------------------------        
    getNumberOfImages(number) {
      console.log('Get the number of images.')
      this.numberOfImages = number
    },
    // ---------------------------
    // Create the downloadable image
    // And get the new file specs
    // ----------------------------            
    downloadImage() {
      console.log('Function downloadImage called')

      this.ditheredWidth = document.getElementById('dithered_' + this.selectedImage.id).width
      this.ditheredHeight = document.getElementById('dithered_' + this.selectedImage.id).height

      const ditheredImageCanvas = document.getElementById('dithered_' + this.selectedImage.id) // the canvas that holds the dithered image
      const downloadUrl = ditheredImageCanvas.toDataURL(this.imageType, 0.72)
      this.downloadFileSize = Math.round((downloadUrl.length * 3) / 4) / 1000
      this.downloadUrl = downloadUrl
    },
    // ---------------------------
    // This receives a palette from ColorPicker in the form of an array of hex values
    // ----------------------------        
    onUpdatePalette(palette) {
      console.log('Color palette updating:')
      console.log(palette.length)
      this.rgbQuantOptions.colors = palette.length
      this.rgbQuantOptions.palette = []
      palette.forEach((v, i) => {
        this.rgbQuantOptions.palette.push(v)
      })
    },
    // ---------------------------
    // When images get uploaded
    // ----------------------------       
    onImageUpload(images) {
      console.log('Process the uploaded images.')
      this.images = images // load the image array into data

      fathom('trackGoal', 'HORTCOPW', 0)

      this.selectedImage = document.getElementById('originalImage1') // set the selected image

      // Dithered image should not be showing yet
      this.showDitheredImage = false

      // Clear the paletter (it might be set from old uploads)
      this.rgbQuantOptions.palette = []

      // Set imageUploaded to true
      this.imageUploaded = true

      // Tell the palette selector to be set to original
      this.$children[1]._data.presetPaletteSelection = 'original'

      // Wait a second before analyzing the image palette (presumably to let the image load?)
      setTimeout(() => {
        this.analyzeImagePalette(document.getElementById('originalImage1'))
      }, 100)
    },
    // ---------------------------
    // Dither the images
    // ----------------------------
    ditherImage() {
      console.log('Dither the images')
      fathom('trackGoal', 'SFMGAORY', 0)

      // When dithering starts, go back to the top
      window.scrollTo(0, 0) 
      
      // Dithering has started
      this.dithering = true 

      // Hide the original images
      this.viewOriginal = false

      // Show the dithered image
      this.showDitheredImage = true

      // Dont show stats while dithering is happening
      this.selectingImage = true

      setTimeout(() => {

        // Go through each of the images and...
        for (let i = 1; i < this.numberOfImages + 1; i++) {
          // Get the original image
          const originalImage = document.getElementById('originalImage' + i)
          // Get the canvas where the dithered image will go
          const ditheredImageCanvas = document.getElementById('dithered_originalImage' + i)
          // Get the canvas context
          const ctx = ditheredImageCanvas.getContext('2d')
          // Set an arbitrary initial width
          const width = 1
          // If the canvas width param is set to 'Original'
          if (this.canvasWidth === 'original') {
            // Set the canvas width to the original image width
            // eslint-disable-next-line no-const-assign
            width = originalImage.naturalWidth
          } else {
            // Otherwise, set it to whatever is selected
            // eslint-disable-next-line no-const-assign
            width = this.canvasWidth
          }
          // Set the height based on the original ratio and the determined width
          const height = (originalImage.naturalHeight / originalImage.naturalWidth) * width
          // Tell the canvas which width and height to be
          ctx.canvas.width = width
          ctx.canvas.height = height
          // Put the image on the canvas
          ctx.drawImage(originalImage, 0, 0, width, height)
          // Create new RgbQuant instance
          const q = new RgbQuant(this.rgbQuantOptions)
          // Analyze histograms to get colors
          q.sample(originalImage) 
          // Dither what is on the canvas
          const ditherResult = q.reduce(ditheredImageCanvas) 
          // Get the newly dithered image data
          const imgData = ctx.getImageData(0, 0, width, height)
          // Set the value of imageData to the dithered image data
          imgData.data.set(ditherResult) 
          // Put the new image data on the canvas
          ctx.putImageData(imgData, 0, 0)
          // Set the data for the image download
          this.downloadImage()
        }
        // Turn off dithering indicator
        this.dithering = false
        // Show stats
        this.selectingImage = false
      }, 100)
      // Set the data for the image download (again?)
      this.downloadImage()
    },
    // ---------------------------
    // Analyze the image palette
    // ----------------------------    
    analyzeImagePalette(e) {
      console.log('Analyze the image palette.')
      console.log('New image selected.')
       this.selectingImage = true
       setTimeout(() => {
        this.selectedImage = e
      
        

        this.rgbQuantOptions.palette = []

        // create new RgbQuant instance
        const q = new RgbQuant(this.rgbQuantOptions)
        // analyze histograms
        q.sample(e)

        this.rgbQuantOptions.palette = q.palette(true)
        this.specsCalculated = true

        if(this.showDitheredImage) {
          this.ditheredWidth = document.getElementById('dithered_' + this.selectedImage.id).width
          this.ditheredHeight = document.getElementById('dithered_' + this.selectedImage.id).height

          const ditheredImageCanvas = document.getElementById('dithered_' + this.selectedImage.id) // the canvas that holds the dithered image
          const downloadUrl = ditheredImageCanvas.toDataURL(this.imageType, 0.72)
          this.downloadFileSize = Math.round((downloadUrl.length * 3) / 4) / 1000
          this.downloadUrl = downloadUrl        
        }

        this.$children[1]._data.presetPaletteSelection = 'original'
        console.log('New image selected.')
        this.selectingImage = false
      }, 50)
      
    }
  }
}
</script>

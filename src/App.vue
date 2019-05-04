<template>
  <div id="app">
    <div class="content gradient">
      <div class="field">
        <div id="iron-man">
          <img src="./assets/Iron_man_mark_85.png" alt="iron man" @click="ease">
          <transition name="slide-fade">
            <div v-if="text" class="bubby iron-man-bubby">
              <p>{{ text }}</p>
            </div>
          </transition>
        </div>
        <div class="thanos-aura">
          <img id="thanos" src="./assets/thanos.png" alt="Thanos">
        </div>
      </div>
    </div>
    <div class="instruction">
      <transition name="fade">
        <p v-if="guide" class="animated">Hit the Iron Man to destroy Thanos</p>
      </transition>
    </div>
  </div>
</template>

<script>
  import html2canvas from 'html2canvas';
  import Chance from 'chance';

  const chance = new Chance();
  const text = ['Thanos! We are in the end game now', 'SNAP!', "I'm Iron Man!"];

  export default {
    name: 'app',

    data() {
      return {
        imageDataArray: [],
        canvasCount: 35,
        text: '',
        guide: false,
      };
    },

    mounted() {
      setTimeout(() => { this.text = text[0] }, 1500);
      setTimeout(() => { this.guide = true }, 3000);
    },

    methods: {
      ease() {
          this.guide = false;
          this.text = text[1];
          html2canvas(window.$(".thanos-aura")[0], {
            backgroundColor: null
          }).then(canvas => {
            //capture all div data as image
            const ctx = canvas.getContext("2d");
            var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
            var pixelArr = imageData.data;
            this.createBlankImageData(imageData);
            //put pixel info to imageDataArray (Weighted Distributed)
            for (let i = 0; i < pixelArr.length; i += 4) {
              //find the highest probability canvas the pixel should be in
              let p = Math.floor((i / pixelArr.length) * this.canvasCount);
              let a = this.imageDataArray[this.weightedRandomDistrib(p)];
              a[i] = pixelArr[i];
              a[i + 1] = pixelArr[i + 1];
              a[i + 2] = pixelArr[i + 2];
              a[i + 3] = pixelArr[i + 3];
            }
            //create canvas for each imageData and append to target element
            for (let i = 0; i < this.canvasCount; i++) {
              let c = this.newCanvasFromImageData(this.imageDataArray[i], canvas.width, canvas.height);
              c.classList.add("dust");
              window.$(".thanos-aura").append(c);
              window.$("#thanos").addClass('hidden');
            }
            this.text = '';
            // clear all children except the canvas
            window.$(".thanos-aura").children().not(".dust").fadeOut(10000);
            //apply animation
            const self = this;
            window.$(".dust").each(function(index) {
              self.animateBlur(window.$(this), 0.8, 800);
              setTimeout(() => {
                self.animateTransform(window.$(this), 100, -100, chance.integer({
                  min: -15,
                  max: 15
                }), 800 + (110 * index));
              }, 70 * index);
              setTimeout(() => { self.text = text[2]; }, 3000);
              setTimeout(() => { self.text = ''; }, 7000);
              //remove the canvas from DOM tree when faded
              window.$(this).delay(70 * index).fadeOut((110 * index) + 800, "easeInQuint", () => {
                window.$(this).remove();
              });
            });
          });
        },

        weightedRandomDistrib(peak) {
          var prob = [],
            seq = [];
          for (let i = 0; i < this.canvasCount; i++) {
            prob.push(Math.pow(this.canvasCount - Math.abs(peak - i), 3));
            seq.push(i);
          }
          return chance.weighted(seq, prob);
        },

        animateBlur(elem, radius, duration) {
          window.$({
            rad: 0
          }).animate({
            rad: radius
          }, {
            duration: duration,
            easing: "easeOutQuad",
            step: function(now) {
              elem.css({
                filter: 'blur(' + now + 'px)'
              });
            }
          });
        },

        animateTransform(elem, sx, sy, angle, duration) {
          var td = 0,
            tx = 0,
            ty = 0;
          window.$({
            x: 0,
            y: 0,
            deg: 0
          }).animate({
            x: sx,
            y: sy,
            deg: angle
          }, {
            duration: duration,
            easing: "easeInQuad",
            step: function(now, fx) {
              if (fx.prop == "x")
                tx = now;
              else if (fx.prop == "y")
                ty = now;
              else if (fx.prop == "deg")
                td = now;
              elem.css({
                transform: 'rotate(' + td + 'deg)' + 'translate(' + tx + 'px,' + ty + 'px)'
              });
            }
          });
        },

        createBlankImageData(imageData) {
          for (let i = 0; i < this.canvasCount; i++) {
            let arr = new Uint8ClampedArray(imageData.data);
            for (let j = 0; j < arr.length; j++) {
              arr[j] = 0;
            }
            this.imageDataArray.push(arr);
          }
        },

        newCanvasFromImageData(imageDataArray, w, h) {
          var canvas = document.createElement('canvas');
          canvas.width = w;
          canvas.height = h;
          const tempCtx = canvas.getContext("2d");
          tempCtx.putImageData(new ImageData(imageDataArray, w, h), 0, 0);

          return canvas;
        }
    }
  }
</script>

<style>
  body {
    margin: 0;
    padding: 0;
    overflow: hidden;
    font-family: 'Viga', sans-serif;
    color: #353535;
  }

  .content {
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .gradient {
    background-image: radial-gradient(#201E33, #000000);
  }

  .field {
    width: 60%;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    min-height: calc(50vh + 110px);
  }

  #iron-man {
    height: 50vh;
    cursor: pointer;
    position: relative;
  }

  #iron-man img {
    max-height: 100%;
  }

  #thanos {
    height: calc(50vh + 100px);
  }

  #thanos.hidden {
    visibility: hidden;
  }

  .bubby {
    background: white;
    border: 2px solid #353535;
    font-size: 1em;
    text-transform: uppercase;
    padding: 0px 40px;
    border-radius: 50%;
    text-align: center;
    max-width: 232px;
  }

  .bubby.iron-man-bubby {
    position: absolute;
    bottom: 100%;
    left: -50%;
    transform: translate(-50px, -20px);
  }

  .bubby.iron-man-bubby:before,
  .bubby.iron-man-bubby:after {
    content: "";
    width: 24px;
    height: 12px;
    background: white;
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translate(20px, 20px);
    border-radius: 50%;
    border: 2px solid #353535;
  }

  .bubby.iron-man-bubby:after {
    width: 12px;
    height: 6px;
    transform: translate(48px, 30px);
  }

  .thanos-aura {
    position: relative;
  }

  .dust {
    position: absolute;
    top: 0;
    left: 0;
  }

  .slide-fade-enter-active {
    transition: all .3s ease;
  }
  .slide-fade-leave-active {
    transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }
  .slide-fade-enter,
  .slide-fade-leave-to {
    transform: translateX(10px);
    opacity: 0;
  }

  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }

  .fade-enter,
  .fade-leave-to {
    opacity: 0;
  }

  @-webkit-keyframes animated {
    from {
        background-position: -300px 0;
    }
    to {
        background-position: 300px 0;
    }
  }

  .animated {
    -webkit-animation: animated 3s linear infinite;
    -webkit-background-clip: text;
    background-color: rgba(153, 153, 153, .15);
    background-image: -webkit-linear-gradient(left, rgba(255, 255, 255, 0), rgba(255, 255, 255, .6), rgba(255, 255, 255, 0));
    background-repeat: no-repeat;
    color: transparent;
    font-size: 1em;
    line-height: 20px;
    position: relative;
    top: 1px;
  }

  .instruction {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
  }
</style>

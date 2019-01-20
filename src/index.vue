<template>
  <div>
    <div class="pswp" tabindex="-1" role="dialog" aria-hidden="true">
      <div class="pswp__bg"></div>
      <div class="pswp__scroll-wrap">
        <div class="pswp__container">
          <div class="pswp__item"></div>
          <div class="pswp__item"></div>
          <div class="pswp__item"></div>
        </div>
        <div class="pswp__ui pswp__ui--hidden">
          <div class="pswp__top-bar">
            <div class="pswp__counter"></div>
            <button class="pswp__button pswp__button--close" title="Close (Esc)"></button>
            <button class="pswp__button pswp__button--share" title="Share"></button>
            <button class="pswp__button pswp__button--fs" title="Toggle fullscreen"></button>
            <button class="pswp__button pswp__button--zoom" title="Zoom in/out"></button>
            <div class="pswp__preloader">
              <div class="pswp__preloader__icn">
                <div class="pswp__preloader__cut">
                  <div class="pswp__preloader__donut"></div>
                </div>
              </div>
            </div>
          </div>
          <div class="pswp__share-modal pswp__share-modal--hidden pswp__single-tap">
            <div class="pswp__share-tooltip"></div>
          </div>
          <button class="pswp__button pswp__button--arrow--left" title="Previous (arrow left)"></button>
          <button class="pswp__button pswp__button--arrow--right" title="Next (arrow right)"></button>
          <div class="pswp__caption">
            <div class="pswp__caption__center"></div>
          </div>
        </div>
      </div>
    </div>
    <div class="collection" :style="{ width: `${containerW}px`, height: `${containerH}px` }">
      <figure v-for="(photo, key) in collection" :key="key">
        <a :href="photo.srcHd" :data-dimension="photo.dimension">
          <img
            :src="photo.src"
            :srcset="photo.srcset"
            :alt="photo.name"
            :style="photo.inline"
            sizes="1px"
            class="js-photo"
          >
        </a>
      </figure>
    </div>
  </div>
</template>

<script>
import fixedPartition from './layouts/fixed-partition.js';
import { photoswipe } from './layouts/photoswipe.js';

export default {
  name: 'MoulCollection',
  data() {
    return {
      photos: [],
      currentWidth: window.innerWidth,
      containerW: 0,
      containerH: 0
    };
  },
  computed: {
    collection() {
      let idealHeight = 320;

      if (this.currentWidth < 600) {
        idealHeight = 200;
      }
      const photo = fixedPartition(this.photos, {
        containerWidth: this.currentWidth - 16,
        idealElementHeight: idealHeight,
        spacing: 8
      });

      const elements = [],
        positions = photo.positions;

      positions.forEach((position, index) => {
        elements.push({
          name: this.photos[index].name,
          srcHd: this.photos[index].src_hd,
          dimension: `${this.photos[index].width_hd}x${
            this.photos[index].height_hd
          }`,
          src: this.photos[index].src,
          srcset: this.photos[index].srcset,
          inline: {
            width: `${positions[index].width}px`,
            height: `${positions[index].height}px`,
            top: `${positions[index].y}px`,
            left: `${positions[index].x}px`
          }
        });
      });

      // eslint-disable-next-line
      this.containerW = photo.width;
      // eslint-disable-next-line
      this.containerH = photo.height;

      return elements;
    }
  },
  methods: {
    handleResize() {
      this.currentWidth = window.innerWidth;
    }
  },
  beforeMount() {
    let photoCollection = document
      .querySelector('#photo-collection')
      .getAttribute('value');
    this.photos = JSON.parse(photoCollection);
  },
  mounted() {
    (() => {
      const throttle = (type, name, obj) => {
        obj = obj || window;
        let running = false;
        const func = () => {
          if (running) {
            return;
          }
          running = true;
          requestAnimationFrame(() => {
            obj.dispatchEvent(new CustomEvent(name));
            running = false;
          });
        };
        obj.addEventListener(type, func);
      };

      throttle('resize', 'optimizedResize');
    })();
    window.addEventListener('optimizedResize', this.handleResize);
    photoswipe('.collection');

    setTimeout(() => {
      let allPhoto = document.querySelectorAll('.js-photo');
      allPhoto.forEach(photo => {
        const sizes = Math.ceil(
          (photo.getBoundingClientRect().width / window.innerWidth) * 100
        );
        photo.setAttribute('sizes', sizes + 'vw');
      });
    }, 100);
  }
};
</script>

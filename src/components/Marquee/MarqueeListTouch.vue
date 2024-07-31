<template>
  <div class="marquee-box" ref="marqueeBox" :class="[inview ? 'inview' : 'no-inview', paused ? 'animation-paused' : '']"
    @touchstart="handleTouchStart" @touchmove="handleTouchMove" @touchend="handleTouchEnd">
    <div class="marquee-content" ref="marqueeContent" :style="contentStyle">
      <div class="marquee-item" v-for="item in repeatData" ref="marqueeItems">
        <slot name="marquee-children"></slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    repeat: {
      type: Number,
      default: 4,
    },
    speed: {
      type: Number,
      default: 0.5, // pixels per frame
    },
  },
  data() {
    return {
      inview: false,
      paused: false,
      offset: 0,
      gap: 6,
      touchStartX: 0,
      isDragging: false,
      itemWidth: 0,
      contentWidth: 0,
      direction: 1, // 1 for right, -1 for left
      isRebounding: false,
    };
  },
  computed: {
    repeatData() {
      return Array(this.repeat).fill().map((_, index) => index);
    },
    contentStyle() {
      return {
        transform: `translate(${this.offset}px, 0)`,
        transition: 'all 0ms ease-in 0s',
        width: `${this.contentWidth}px`,
      };
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.initSizes();
      this.startAnimation();
    });
    window.addEventListener('resize', this.initSizes);
    if ('IntersectionObserver' in window) {
      this.createIntersectionObserver();
    }
    // document.addEventListener('visibilitychange', this.handlePageVisibilityChange);
    // this.startAnimation();
  },
  beforeDestroy() {
    this.stopAnimation();
    window.removeEventListener('resize', this.initSizes);
    if (this.intersectionObserver) {
      this.intersectionObserver.disconnect();
    }
    document.removeEventListener('visibilitychange', this.handlePageVisibilityChange);
  },
  methods: {
    initSizes() {
      if (this.$refs.marqueeItems && this.$refs.marqueeItems.length > 0) {
        this.itemWidth = this.$refs.marqueeItems[0].offsetWidth + this.gap;
        this.contentWidth = this.itemWidth * this.repeat;
      }
    },
    handlePageVisibilityChange() {
      this.paused = document.visibilityState !== 'visible';
      if (this.paused) {
        this.stopAnimation();
      } else {
        this.startAnimation();
      }
    },
    createIntersectionObserver() {
      this.intersectionObserver = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          this.inview = entry.isIntersecting;
          if (this.inview) {
            this.$nextTick(() => {
              this.initSizes();
              this.startAnimation();
            });
          } else {
            this.stopAnimation();
          }
        });
      });
      this.intersectionObserver.observe(this.$refs.marqueeBox);
    },
    startAnimation() {
      if (this.animationFrame) {
        cancelAnimationFrame(this.animationFrame);
      }
      const animate = () => {
        if (!this.paused && this.inview && !this.isDragging && !this.isRebounding) {
          this.offset -= this.speed * this.direction;

          if (this.direction > 0 && this.offset <= -this.itemWidth) {
            this.offset = 0;
          } else if (this.direction < 0 && this.offset >= 0) {
            this.offset = -this.itemWidth + this.speed;
          }
        }
        this.animationFrame = requestAnimationFrame(animate);
      };
      this.animationFrame = requestAnimationFrame(animate);
    },
    stopAnimation() {
      if (this.animationFrame) {
        cancelAnimationFrame(this.animationFrame);
      }
    },
    handleTouchStart(e) {
      this.touchStartX = e.touches[0].clientX;
      this.isDragging = true;
    },
    handleTouchMove(e) {
      if (!this.isDragging) return;
      const touchCurrentX = e.touches[0].clientX;
      const deltaX = touchCurrentX - this.touchStartX;

      this.direction = deltaX > 0 ? 1 : -1;
      this.offset += deltaX;

      // Constrain the offset
      if (this.offset > 0) {
        this.offset = Math.min(this.offset, this.itemWidth); // Allow some overscroll
      } else if (this.offset < -this.itemWidth) {
        this.offset = Math.max(this.offset, -this.contentWidth); // Allow some overscroll
      }

      this.touchStartX = touchCurrentX;
    },
    handleTouchEnd() {
      this.isDragging = false;
      if (this.offset > 0) {
        this.reboundLeft();
      } else if (this.offset < -this.itemWidth) {
        this.offset += this.itemWidth;
      }
      this.$nextTick(() => {
        this.direction = 1;
      });
    },
    reboundLeft() {
      this.isRebounding = true;
      const startOffset = this.offset;
      const startTime = performance.now();
      const duration = 300; // ms

      const animate = (currentTime) => {
        const elapsed = currentTime - startTime;
        if (elapsed < duration) {
          this.offset = startOffset * (1 - elapsed / duration);
          requestAnimationFrame(animate);
        } else {
          this.offset = 0;
          this.isRebounding = false;
        }
      };

      requestAnimationFrame(animate);
    },
  },
};
</script>

<style lang="less" scoped>
.marquee-box {
  overflow: hidden;
  position: relative;
  touch-action: pan-y;

  &::before,
  &::after {
    width: 38px;
    height: 100%;
    opacity: 0.4;
    position: absolute;
    display: block;
    bottom: 0;
    top: 0;
    z-index: 1;
    content: "";
  }

  &::before {
    left: 0;
    background: linear-gradient(90deg, rgba(170, 170, 170, 0.54) 0%, rgba(255, 255, 255, 0) 100%);
  }

  &::after {
    background: linear-gradient(90deg, rgba(255, 255, 255, 0) 0%, rgba(170, 170, 170, 0.54) 100%);
    right: 0;
  }

  .marquee-content {
    display: flex;
    flex-direction: row;
    will-change: transform;
  }

  .marquee-item {
    flex-shrink: 0;
    display: flex;
    margin-right: var(--2xgap);
    flex-direction: row;
    justify-content: space-around;
  }
}
</style>
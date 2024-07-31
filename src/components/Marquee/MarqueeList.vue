<template>
  <div class="marquee-box" ref="observedElement" :class="inview ? 'inview' : 'no-inview'">
    <div class="marquee-item" v-for="item in repeatData" :class="paused ? 'animation-paused' : ''">
      <slot name="marquee-children" />
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
  },
  data() {
    return {
      inview: false,
      paused: false,
      timeRef: null,
    };
  },
  computed: {
    repeatData() {
      return Array(this.repeat).map((_, index) => index);
    },
  },
  mounted() {
    if ('IntersectionObserver' in window) {
      this.createIntersectionObserver();
    } else {
      console.warn('IntersectionObserver is not supported in this browser.');
      // Handle the case where IntersectionObserver is not supported
    }
    document.addEventListener('visibilitychange', this.handlePageVisibilityChange);
  },
  beforeDestroy() {
    if (this.intersectionObserver) {
      this.intersectionObserver.disconnect();
    }
    document.removeEventListener('visibilitychange', this.handlePageVisibilityChange);
  },
  destroyed() { },
  methods: {
    handlePageVisibilityChange() {
      if (document.visibilityState === 'visible') {
        this.paused = false;
      } else {
        this.paused = true;
      }
    },
    createIntersectionObserver() {
      this.intersectionObserver = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          this.inview = entry.isIntersecting;
        });
      });

      this.intersectionObserver.observe(this.$refs.observedElement);
    },
  },
};
</script>
<style lang="less" scoped>
.marquee-box {
  --gap: 6px;
  --2xgap: 6px;

  display: flex;
  overflow: hidden;
  flex-direction: row;
  position: relative;

  &.animation-paused {
    overflow-y: hidden;
    overflow-x: scroll;
  }

  &::-webkit-scrollbar {
    display: none;
  }

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

  @keyframes marquee {
    0% {
      transform: translate3d(0, 0, 0);
    }

    to {
      transform: translate3d(calc(-100% - var(--2xgap)), 0, 0);
    }
  }

  @keyframes marquee-vertical {
    0% {
      transform: translateY(0);
    }

    to {
      transform: translateY(calc(-100% - var(--2xgap)));
    }
  }

  .marquee-item {
    flex-shrink: 0;
    display: flex;
    flex-direction: row;
    overflow: hidden;
    margin-right: var(--gap);
    justify-content: space-around;
    will-change: transform;
    transform-style: preserve-3d;
    backface-visibility: hidden;
    perspective: 1000px;
    transform: translate3d(0, 0, 0);

    &.animation-paused {
      animation-play-state: paused;
    }
  }

  &.inview {
    .marquee-item {
      animation: marquee var(--duration) infinite linear;
    }
  }
}
</style>
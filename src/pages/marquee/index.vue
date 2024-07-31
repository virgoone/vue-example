<script setup lang="ts">
definePage({
  name: 'Marquee Example',
  meta: {
    level: 2,
    title: '🎨 Marquee Example',
    i18n: 'home.marquee',
  },
})
const items = ref(Array.from({ length: 20 }))
const duration = computed(() => items.value.length * 1.5)
const isIOS = ref(/(iPhone|iPad|iPod|iOS)/i.test(navigator.userAgent))
</script>

<template>
  <div class="marquee-page">
    <div class="header-img">
      <div class="title"></div>
    </div>
    <div class="content">
      <div class="module-content">
        <div class="book-view-scroll">
          <div class="wrap">
            <div :class="['scroll-wrap', isIOS ? 'ios' : 'not-ios']" :style="`--duration: ${duration}s`">
              <div class="wrap flex row overflow-hidden">
                <MarqueeList :repeat="2">
                  <template #marquee-children>
                    <div class="module-item" v-for="(_, index) in items" :key="index">
                      <div class="module-item-content">
                      </div>
                    </div>
                  </template>
                </MarqueeList>
              </div>
            </div>
          </div>
        </div>

        <div class="book-view-scroll">
          <div class="wrap">
            <div :class="['scroll-wrap', isIOS ? 'ios' : 'not-ios']" :style="`--duration: ${duration}s`">
              <div class="wrap flex row overflow-hidden">
                <MarqueeListTouch :repeat="2">
                  <template #marquee-children>
                    <div class="module-item" v-for="(_, index) in items" :key="index">
                      <div class="module-item-content">
                      </div>
                    </div>
                  </template>
                </MarqueeListTouch>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style lang="less">
.marquee-page {
  width: 100%;
  min-height: 100vh;
  background: #55c1fa url('https://meme-static.douni.one/header-iamge.jpg') no-repeat top/100%;

  .header-img {
    height: 205px;

    .title {
      position: absolute;
      z-index: 5;
      top: 60px;
      right: 18px;
      width: 223px;
      height: 160px;
      background: url(https://imgservices-1252317822.image.myqcloud.com/coco/s07242024/299b202d.6jy83o.png) no-repeat center/contain;
    }
  }

  .content {
    width: 100%;
    height: 603px;
    background: url('https://meme-static.douni.one/content-bg.png') no-repeat top/100%;

    .module-content {
      padding: 40px 28px 0;

      .book-view-scroll {
        width: 320px;
        height: 108px;
        margin: 0 auto 12px;
        background: url(https://meme-static.douni.one/content-scroll-bg.png) no-repeat center/contain;

        .wrap {
          padding: 7px 8px 0 6px;
        }

        .scroll-wrap {
          overflow: hidden;

          &.ios {
            * {
              will-change: transform;
              transform-style: preserve-3d;
              backface-visibility: hidden;
              perspective: 1000px;
              transform: translate3d(0, 0, 0);
            }
          }
        }

        .module-item {
          position: relative;
          width: 56px;
          margin-right: 6px;
          flex-shrink: 0;

          &-content {
            width: 56px;
            height: 80px;
            border-radius: 16px;
            background: linear-gradient(180deg, #25d9ff 0%, #6ef4ff 100%);
          }
        }
      }
    }
  }
}
</style>
<template>
  <div id="app">
    <gesture-wrapper @right="onSwipe('right')" @left="onSwipe('left')" :min-distance="200">
      <template v-slot="{ wrapper, isMove, isMoveRight, isMoveLeft, cssNumber }">
        <div>
          <div
            class="transition-curve transition-curve-left"
            :style="[
              isMoveRight && { opacity: cssNumber }, 
              isMoveRight && { transform: `translate3d(0, 0, 0) scale(${cssNumber}, 1)`}, 
              (!isMoveRight || !isMove) && { transition: 'transform 400ms, opacity 400ms' }
            ]"
          >
            <svg id="curve-left" viewBox="0 0 347.6 1416.9">
              <path d="M0,0c0,354.2,347.6,354.2,347.6,708.5C347.6,1062.7,0,1062.7,0,1416.9V0z"></path>
            </svg>
          </div>

          <div
            class="transition-curve transition-curve-right"
            :style="[
              isMoveLeft && { opacity: cssNumber }, 
              isMoveLeft && { transform: `translate3d(0, 0, 0) scale(${cssNumber}, 1)`}, 
              (!isMoveLeft || !isMove) && { transition: 'transform 400ms, opacity 400ms' }
            ]"
          >
            <svg id="curve-right" viewBox="0 0 347.6 1416.9">
              <path
                d="M347.6,1416.9C347.6,1062.7,0,1062.7,0,708.4C0,354.2,347.6,354.2,347.6,0V1416.9z"
              ></path>
            </svg>
          </div>
        </div>
      </template>
    </gesture-wrapper>
  </div>
</template>

<script>
import GestureWrapper from './components/GestureWrapper.vue';

export default {
    components: {
        GestureWrapper,
    },

    methods: {
        onSwipe(direction) {
            switch (direction) {
                case 'left':
                    console.log('next route');
                    break;
                case 'right':
                    console.log('prev route');
                    break;
                default:
                    break;
            }
        },
    },
};
</script>

<style lang="scss">
.transition-curve {
    position: fixed;
    top: 0;
    width: 25vh;
    height: 100vh;
    pointer-events: none;
    z-index: 900;
    will-change: transform;
    transform: translate3d(0, 0, 0) scale(0, 1);

    svg {
        display: block;
        height: 100%;
        fill: #c3dafe;
    }
}
.transition-curve-left {
    transform-origin: center left;
    left: -2px;
}
.transition-curve-right {
    transform-origin: center right;
    right: -2px;
}
</style>

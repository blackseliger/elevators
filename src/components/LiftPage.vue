
<template>
  <div class="lift">
    <div class="lift__floors">
      <div
        v-for="level in levels"
        :key="`${level}-lift`"
        :class="`lift__shaft_${level} lift__shaft`"
      >
        {{ level }}
      </div>

      <div
        class="lift__shaft_cabin lift__shaft"
        :class="{ lift__shaft_cabin_blinker: config.blink }"
        :style="config.classMove"
        ref="cabin"
      >
        <ui-icon
          style="
            position: absolute;
            transform: rotate(270deg);
            top: -17px;
            background-color: #2711c1b3;
            width: 40px;
            height: 50px;
            opacity: 0.6;
          "
          :icon="`arrow`"
          v-show="config.showDirect === 'up' ? true : false"
        />
        cabin
        <ui-icon
          style="
            position: absolute;
            transform: rotate(90deg);
            top: 68%;
            background-color: #2711c1b3;
            width: 40px;
            height: 50px;
            opacity: 0.6;
          "
          :icon="`arrow`"
          v-show="config.showDirect === 'down' ? true : false"
        />
      </div>
    </div>
  </div>
</template>

<script>
import UiIcon from "@/components/UiIcon.vue";

export default {
  name: "LiftPage",

  components: {
    UiIcon,
  },

  data() {
    return {
      floor: null,
      config: {
        classMove: {
          top: `${100 - Math.floor((1 * 100) / this.levels.length)}%`,
        },
        floors: [0],
        finish: false,
        lastfloor: null,
        queue: [],
        pause: false,
        showDirect: null,
        move: true,
        blink: false,
        start: false,
      },
    };
  },

  props: {
    levels: {
      type: Array,
      required: true,
    },
    selectedFloor: {
      type: Number || null,
      required: true,
    },
    configParent: {
      type: Object,
      required: true,
    },
  },

  emits: ["deleteFloorQueue", "changeConfigPar", "changeQueueFloors", "deleteSelected"],

  mounted() {
    this.config.queue =
      JSON.parse(localStorage.getItem(this.configParent.index)) || [];

    const defaultMove = {
      top: `${100 - Math.floor((1 * 100) / this.levels.length)}%`,
    };
    this.config.classMove =
      JSON.parse(
        localStorage.getItem(`${this.configParent.index}/classMove`)
      ) || defaultMove;

    this.config.lastfloor =
      JSON.parse(
        localStorage.getItem(`${this.configParent.index}/lastFloor`)
      ) || 1;
    // this.config.showDirect = JSON.parse(localStorage.getItem( `${this.configParent.index.index}/showDirect`)) || null;
    if (this.config.queue.length) {
      this.startWork();
    }
  },

  methods: {
    pause() {
      let _this = this.config;
      let _thisPar = this.configParent;
      setTimeout(() => {
        if (this.configParent.pause) return null;

        if (_this.finish === true && !this.configParent.move) {
          this.$emit("changeConfigPar", this.configParent.index, "pause", true);
          _this.blink = true;
          setTimeout(() => {
            if (_thisPar.floors.length === 1) {
              _this.lastfloor = _thisPar.floors[0];
              localStorage.setItem(
                `${this.configParent.index}/lastFloor`,
                JSON.stringify(_this.lastfloor)
              );
            }
            _this.finish = false;

            this.$emit("changeConfigPar", _thisPar.index, "pause", false);
             this.$emit("deleteFloorQueue");
            _this.blink = false;
            _this.queue.shift();
            localStorage.setItem(_thisPar.index, JSON.stringify(_this.queue));
            localStorage.setItem(
              `${_thisPar.index}/classMove`,
              JSON.stringify(_this.classMove)
            );
            this.$emit("changeConfigPar", _thisPar.index, "move", true);
            if (_this.queue.length) {
              this.pushLift();
            }
          }, 3000);
        }
      }, 0);
    },

    pushLift() {
      let _this = this.config;
      // let _thisPar = this.configParent;

      setTimeout(() => {
        _this.classMove = {
          top: `${_this.queue[0]?.coords}%`,
          transition: `top ${_this.queue[0]?.speed}s ease-in-out`,
        };
      }, 0);

      setTimeout(() => {
        if (_this.queue.length && !this.configParent.pause) {
          _this.showDirect = _this.queue[0]?.direction;
        }

        if (this.configParent.move && _this.queue[0]) {
          this.$emit("changeConfigPar", this.configParent.index, "move", false);

          setTimeout(() => {
            _this.finish = true;
            this.$emit("changeQueueFloors", this.configParent.index, "shift");
            this.$emit("deleteFloorQueue");
            _this.showDirect = null;
            this.pause();
            this.$emit('deleteSelected');
          }, _this.queue[0].speed * 1000);
        }
      });
    },

    startWork() {
      this.pushLift();
    },
  },

  computed: {
    cssFloors() {
      return this.levels.length;
    },
  },

  watch: {
    selectedFloor() {
      if (this.selectedFloor !== null) {
        this.floor = this.selectedFloor;
      }
    },

    floor() {
      let _this = this.config;
      let _thisPar = this.configParent;
      this.$emit(
        "changeQueueFloors",
        _thisPar.index,
        "push",
        this.selectedFloor
      );

      let actualFloor = _thisPar.floors[_thisPar.floors.length - 1];
      let prevFloor = _thisPar.floors[_thisPar.floors.length - 2] || _this.lastfloor;
      if (actualFloor === prevFloor) return null;

      if (_thisPar.floors.length === 1 && prevFloor === undefined) {
        if (actualFloor === _this.lastfloor) {
          this.$emit("changeQueueFloors", this.configParent.index, "shift");
          return null;
        }

        _this.queue.push({
          actualFloor: actualFloor,
          speed: Math.abs(actualFloor - _this.lastfloor),
          direction: actualFloor > _this.lastfloor ? "up" : "down",
          coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
        });
        // this.workLocalStorage("queue");
        localStorage.setItem(_thisPar.index, JSON.stringify(_this.queue));
      } else {
        _this.queue.push({
          actualFloor: actualFloor,
          speed: Math.abs(actualFloor - prevFloor),
          direction: actualFloor > prevFloor ? "up" : "down",
          coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
        });
        localStorage.setItem(_thisPar.index, JSON.stringify(_this.queue));

        // this.workLocalStorage("queue");
      }
      this.startWork();
    },
  },
};
</script>

<style lang="scss" scoped>
.lift {
  position: relative;

  /*  grid  */
  display: grid;
  grid-template-columns: 100px 100px;

  &__floors {
    display: grid;
    grid-template-rows: repeat(v-bind(cssFloors), 100px);
    position: relative;
    gap: 5px;
    align-items: center;
    justify-items: center;
  }

  &__controls {
    display: grid;
    grid-template-rows: repeat(v-bind(cssFloors), 100px);
    gap: 5px;
    align-items: center;
    justify-items: center;
  }

  &__shaft {
    // grid-area: shaft;
    background-color: rgb(230, 118, 118);
    border: 1px solid gray;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  &__shaft_cabin {
    position: absolute;
    background-color: rgb(211, 78, 78);
    // top: 80%;
    width: 100px;
    height: 100px;
    border: 1px solid #d34e4e;
  }

  &__shaft_cabin_blinker {
    animation: blinker 1s step-start infinite;
  }

  &__button {
    &_queue {
      background-color: rgba(54, 106, 136, 0.753);
    }
    &_actual {
      background-color: #4bc9e9;
    }
  }

  @keyframes blinker {
    50% {
      opacity: 0.5;
    }
  }
}
</style>


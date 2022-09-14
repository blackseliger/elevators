
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
    <!-- <div class="lift__controls">
      <ui-button
        v-for="level in levels"
        :key="`${level}-button`"
        :value="level"
        v-model:selected="selected"
        :class="[
          parseInt(level) === queue[0]?.actualFloor
            ? 'lift__button_actual'
            : '',
          floors?.includes(parseInt(level)) ? 'lift__button_queue' : '',
          `lift__button_${level}`,
        ]"
        >{{ level }} этаж
      </ui-button>
    </div> -->
  </div>
</template>

<script>
// import UiButton from "@/components/UiButton.vue";
import UiIcon from "@/components/UiIcon.vue";

export default {
  name: "LiftPage",

  components: {
    // UiButton,
    UiIcon,
  },

  props: {
    levels: {
      type: Array,
      required: true,
    },
    config: {
      type: Object,
      required: true,
    }
  },

  // data() {
  //   return {
  //     floors: [],
  //     lastfloor: "",
  //     queue: [],
  //     pause: { rest: false },
  //     selected: 0,
  //     classMove: null,
  //     showDirect: null,
  //     indexStart: false,
  //     move: false,
  //     blink: false,
  //   };
  // },

  // mounted() {
  //   this.floors = JSON.parse(localStorage.getItem("floors")) || [];
  //   this.queue = JSON.parse(localStorage.getItem("queue")) || [];
  //   this.lastfloor = JSON.parse(localStorage.getItem("lastfloor")) || 1;

  //   this.classMove = {
  //     top: `${100 - Math.floor((this.lastfloor * 100) / this.levels.length)}%`,
  //   };

  //   if (this.floors.length || this.queue.length) {
  //     this.classMove = { top: `${this.queue[0]?.coords}%` };
  //     this.startWork();
  //   }
  // },

  // methods: {
  //   pushLift() {
  //     const cabin = this.$refs.cabin;
  //     let amountValuesH =
  //       this.levels.length * 100 + 6 + (this.levels.length - 1) * 5;
  //     let coords = Math.ceil(
  //       Math.ceil(parseFloat(getComputedStyle(cabin).top)) /
  //         (amountValuesH * 0.01)
  //     );

  //     this.classMove = {
  //       top: `${this.queue[0]?.coords}%`,
  //       transition: `top ${this.queue[0]?.speed}s ease-in-out`,
  //     };

  //     this.move = true;
  //     this.showDirect = this.queue[0]?.direction;

  //     if (this.pause.rest) return null;
  //     if (coords === this.queue[0]?.coords && this.move) {
  //       // не могу понять в чем баг если выбрать больше 2 этажей, то последний будет бликать 6 секунд. this.queue.shift(); первый раз срабатывает корректно
  //       // но сам this.queue для setTimeout не обновляется

  //       this.move = false;
  //       this.pause.rest = true;
  //       this.blink = true;

  //       setTimeout(() => {
  //         if (this.floors.length === 1) {
  //           this.lastfloor = this.floors[0];
  //           this.workLocalStorage("lastfloor");
  //         }
  //         this.floors.shift();
  //         this.pause.rest = false;
  //         this.blink = false;
  //         this.queue.shift();
  //         this.workLocalStorage("floors");
  //         this.workLocalStorage("queue");
  //       }, 3000);
  //     }
  //   },

  //   startWork() {
  //     if (this.indexStart === false) {
  //       this.indexStart = setInterval(() => {
  //         this.pushLift();
  //       }, 1000);
  //     }
  //   },

  //   workLocalStorage(name) {
  //     localStorage.setItem(name, JSON.stringify(this[name]));
  //   },
  // },

  computed: {
    cssFloors() {
      return this.levels.length;
    },
  },

  // watch: {
  //   selected() {
  //     this.floors.push(this.selected);
  //     this.workLocalStorage("floors");
  //   },

  //   floors: {
  //     deep: true,
  //     handler() {
  //       let actualFloor = this.floors[this.floors.length - 1];
  //       let prevFloor = this.floors[this.floors.length - 2];

  //       if (actualFloor === prevFloor) return null;

  //       if (this.floors.length === 1 && prevFloor === undefined) {
  //         if (actualFloor === this.lastfloor) {
  //           this.floors.shift();
  //           this.workLocalStorage("floors");
  //           return null;
  //         }

  //         this.queue.push({
  //           actualFloor: actualFloor,
  //           speed: Math.abs(actualFloor - this.lastfloor),
  //           direction: actualFloor > this.lastfloor ? "up" : "down",
  //           coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
  //         });
  //         this.workLocalStorage("queue");
  //       } else {
  //         this.queue.push({
  //           actualFloor: actualFloor,
  //           speed: Math.abs(actualFloor - prevFloor),
  //           direction: actualFloor > prevFloor ? "up" : "down",
  //           coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
  //         });
  //         this.workLocalStorage("queue");
  //       }

  //       this.startWork();
  //     },
  //   },
  // },
};
</script>

<style lang="scss" scoped>
.lift {
  // margin: auto;
  // width: 205px;
  // max-width: 205px;
  // border: 1px solid black;
  // padding: 2px;
  // gap: 5px;
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


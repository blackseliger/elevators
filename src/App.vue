<template>
  <div class="lifts">
    <div class="lifts__wrapper-lift">
      <LiftPage
        v-for="lift in lifts"
        :key="lift.id"
        :config="lift"
        :levels="levels"
        :classMove="classMove"
        :blink="blink"
        :showDirect="showDirect"
      />
    </div>
    <div class="lifts__controls">
      <ui-button
        v-for="level in levels"
        :key="`${level}-button`"
        :value="level"
        v-model:selected="selected"
        :class="[
          parseInt(level) === actualLift?.queue[0]?.actualFloor
            ? 'controls__button_actual'
            : '',
          floors?.includes(parseInt(level)) ? 'controls__button_queue' : '',
          `controls__button_${level}`,
        ]"
        >{{ level }} этаж
      </ui-button>
    </div>
  </div>
</template>

<script>
import LiftPage from "@/components/LiftPage.vue";
import UiButton from "@/components/UiButton.vue";

// подумать над динамическим изображением кнопок

export default {
  name: "App",
  components: {
    LiftPage,
    UiButton,
  },

  data() {
    return {
      levels: [5, 4, 3, 2, 1],
      liftsAmount: 4,
      lifts: [],
      floors: [],
      // lastfloor: "",
      // queue: [],
      // pause: { rest: false },
      selected: 0,
      // classMove: null,
      // showDirect: null,
      indexStart: false,
      // move: false,
      // blink: false,
      actualLift: null,
      intervalRest: false,
      // cabinCoords:
    };
  },

  created() {
    for (let i = 0; i < this.liftsAmount; i++) {
      this.lifts.push({
        classMove: {
          top: `${100 - Math.floor((1 * 100) / this.levels.length)}%`,
        },
        floors: [0],
        finish: false,
        lastfloor: 1,
        queue: [],
        pause: { rest: false },
        showDirect: null,
        move: false,
        blink: false,
        start: false,
        id: `${i}/lift`,
        // watchLift: false,
      });
    }
  },
  // `${100 - Math.floor((1 * 100) / this.levels.length)}px`
  computed: {
    widthLifts() {
      return `${this.liftsAmount * 150}px`;
    },

    cssFloors() {
      return this.levels.length;
    },
  },

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

  methods: {
    pause() {
      let _this = this.actualLift;
      _this.showDirect = false;

      if (_this.pause.rest) return null;
      if (_this.finish === true && _this.move) {
        // не могу понять в чем баг если выбрать больше 2 этажей, то последний будет бликать 6 секунд. this.queue.shift(); первый раз срабатывает корректно
        // но сам this.queue для setTimeout не обновляется
        _this.finish = false;
        _this.move = false;
        _this.pause.rest = true;
        _this.blink = true;
        setTimeout(() => {
          if (_this.floors.length === 1) {
            _this.lastfloor = _this.floors[0];
            // this.workLocalStorage("lastfloor");
          }
          _this.showDirect = false;
          _this.floors.shift();
          _this.pause.rest = false;
          _this.blink = false;
          _this.queue.shift();
          this.pushLift();
          // this.workLocalStorage("floors");
          // this.workLocalStorage("queue");
        }, 3000);
      }
    },

    pushLift() {
      let _this = this.actualLift;
      _this.classMove = {
        top: `${_this.queue[0]?.coords}%`,
        transition: `top ${_this.queue[0]?.speed}s ease-in-out`,
      };

      if (!this.move && _this.queue[0]) {
        setTimeout(() => {
          _this.finish = true;
          this.intervalRest = false;
          this.pause();
          this.floors.shift();
        }, _this.queue[0].speed * 1000);
      }

      _this.move = true;
      _this.showDirect = _this.queue[0]?.direction;

      // if (_this.pause.rest) return null;
      // if (_this.finish === true && _this.move) {
      // if (this.intervalRest) return null;
      //   this.intervalRest = true;
      //   // не могу понять в чем баг если выбрать больше 2 этажей, то последний будет бликать 6 секунд. this.queue.shift(); первый раз срабатывает корректно
      //   // но сам this.queue для setTimeout не обновляется
      //   _this.finish = false;
      //   _this.move = false;
      //   _this.pause.rest = true;
      //   _this.blink = true;
      //   console.log('heh')
      //   setTimeout(() => {
      //     if (_this.floors.length === 1) {
      //       _this.lastfloor = _this.floors[0];
      //       // this.workLocalStorage("lastfloor");
      //     }
      //     _this.floors.shift();
      //     _this.pause.rest = false;
      //     _this.blink = false;
      //     _this.queue.shift();
      //     // this.pushLift();
      //     // this.intervalRest = false;
      //     // this.workLocalStorage("floors");
      //     // this.workLocalStorage("queue");
      //   }, 3000);

      // }
    },

    startWork() {
      this.pushLift();
      // if (this.indexStart === false) {
      //   this.indexStart = setInterval(() => {
      //     this.pushLift.call(this);
      //   }, 1000);
      // }
    },

    workLocalStorage(name) {
      localStorage.setItem(name, JSON.stringify(this[name]));
    },
  },

  watch: {
    selected() {
      const freeLifts = this.lifts.filter(
        (el) => el.move === false && el.pause.rest === false
      );
      console.log(freeLifts);
      if (freeLifts.length >= 2) {
        const lastFloors = freeLifts.map(
          (el) => {
            let value = el.floors[el.floors.length - 1];
            return value;
            } 
        );
        const closest = lastFloors.sort(
          (a, b) => Math.abs(this.selected - a) - Math.abs(this.selected - b)
        )[0];
        const actualLiftIndex = this.lifts.findIndex(
          (el) => el.floors[el.floors.length - 1] === closest
        );
        console.log(actualLiftIndex);
        this.actualLift = this.lifts[actualLiftIndex];
        this.actualLift.floors.push(this.selected);
      } else {
        // если не подходит условию просто берем всегда первый лифт
        this.actualLift = this.lifts[0];
        this.actualLift.floors.push(this.selected);
      }

      this.floors.push(this.selected);
      // const actualNum = Math.min(...queueAmount);
      // if (this.actualLift) this.actualLift === null;
      // this.watchLift = true;
      // this.watchLift = false;
      // this.floors.push(this.selected); старое
      // нужно закидывать активные лифт в список локал стораджа
      // this.workLocalStorage("floors");
    },

    // floors: { старое
    "actualLift.floors": {
      deep: true,
      handler() {
        let _this = this.actualLift;
        let actualFloor = _this.floors[_this.floors.length - 1];
        let prevFloor = _this.floors[_this.floors.length - 2];

        if (actualFloor === prevFloor) return null;

        if (_this.floors.length === 1 && prevFloor === undefined) {
          if (actualFloor === this.lastfloor) {
            _this.floors.shift();
            // this.workLocalStorage("floors");
            return null;
          }

          _this.queue.push({
            actualFloor: actualFloor,
            speed: Math.abs(actualFloor - _this.lastfloor),
            direction: actualFloor > _this.lastfloor ? "up" : "down",
            coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
          });
          // this.workLocalStorage("queue");
        } else {
          _this.queue.push({
            actualFloor: actualFloor,
            speed: Math.abs(actualFloor - prevFloor),
            direction: actualFloor > prevFloor ? "up" : "down",
            coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
          });
          // this.workLocalStorage("queue");
        }
        // this.watchLift = false;
        this.startWork();
      },
    },
  },
};
</script>

<style lang="scss" scoped>
.lifts {
  width: calc(152px + v-bind(widthLifts));
  display: grid;
  grid-template-columns: v-bind(widthLifts) 100px;
  gap: 10px;
  margin: auto;
  padding: 10px;
  border: 1px solid black;

  &__controls {
    display: grid;
    grid-template-rows: repeat(v-bind(cssFloors), 100px);
    gap: 5px;
    align-items: center;
    justify-items: center;
  }

  &__wrapper-lift {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    gap: 5px;
  }
}

.controls {
  &__button {
    &_queue {
      background-color: rgba(54, 106, 136, 0.753);
    }
    &_actual {
      background-color: #4bc9e9;
    }
  }
}
</style>

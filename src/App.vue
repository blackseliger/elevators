<template>
  <div class="lifts">
    <div class="lifts__wrapper-lift">
      <LiftPage
        v-for="lift in lifts"
        :key="lift.id"
        :levels="levels"
        :configParent="{
          index: lift.id,
          move: lift.move,
          pause: lift.pause,
          floors: lift.floors,
        }"
        :selectedFloor="lift.id === actualIndex ? selected : null"
        :deleteFloorQueue="deleteFloorQueue()"
        :changeConfigPar="changeConfigPar()"
        :changeQueueFloors="changeQueueFloors()"
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

export default {
  name: "App",
  components: {
    LiftPage,
    UiButton,
  },

  data() {
    return {
      levels: [5, 4, 3, 2, 1],
      liftsAmount: 2,
      queueLifts: [],
      lifts: [],
      floors: [],
      selected: 0,
      indexStart: false,
      actualLift: null,
      actualIndex: null,
    };
  },

  created() {
    for (let i = 0; i < this.liftsAmount; i++) {
      this.lifts.push({
        // classMove: {
        //   top: `${100 - Math.floor((1 * 100) / this.levels.length)}%`,
        // },
        floors: [0],
        // finish: false,
        // lastfloor: 1,
        // queue: [],
        pause: false,
        // showDirect: null,
        move: true,
        // blink: false,
        // start: false,
        id: i,
      });
    }
  },

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
    deleteFloorQueue() {
      this.floors.shift();
    },

    changeConfigPar(index, type, bool) {
      this.lifts[index][type] = bool;
    },

    changeQueueFloors(index, method, value = null) {
      if (method === 'push') {
        this.lifts[index].floors.push(value)
      } else if (method === 'shift') {
        this.lifts[index].floors.shift();
      }
    },
    // pause() {
    //   // let _this = this.actualLift;
    //   let _this = this.queueLifts[this.actualIndex];

    //   setTimeout(() => {
    //     // console.log("pause");
    //     if (_this.pause.rest) return null;
    //     if (_this.finish === true && !_this.move) {
    //       _this.pause.rest = true;
    //       _this.blink = true;
    //       setTimeout(() => {
    //         if (_this.floors.length === 1) {
    //           _this.lastfloor = _this.floors[0];
    //           // this.workLocalStorage("lastfloor");
    //         }
    //         _this.finish = false;
    //         _this.floors.shift();
    //         _this.pause.rest = false;
    //         _this.blink = false;
    //         _this.queue.shift();
    //         _this.move = true;
    //         // console.log("до повтора", _this.queue.length);
    //         if (_this.queue.length) {
    //           // console.log("повтор");
    //           this.pushLift();
    //         }
    //         // this.workLocalStorage("floors");
    //         // this.workLocalStorage("queue");
    //       }, 3000);
    //     }
    //   }, 0);
    // },

    // pushLift() {
    //   // let _this = this.actualLift;
    //   let _this = this.queueLifts[this.actualIndex];

    //   setTimeout(() => {
    //     _this.classMove = {
    //       top: `${_this.queue[0]?.coords}%`,
    //       transition: `top ${_this.queue[0]?.speed}s ease-in-out`,
    //     };
    //   }, 0);

    //   setTimeout(() => {
    //     if (_this.queue.length && !_this.pause.rest) {
    //       // console.log("работает второй таймаут в pushlift с проверкой 1");
    //       _this.showDirect = _this.queue[0]?.direction;
    //     }
    //     if (_this.move && _this.queue[0]) {
    //       console.log("работает второй таймаут в pushlift с проверкой 2");

    //       _this.move = false;
    //       setTimeout(() => {
    //         _this.finish = true;
    //         this.floors.shift();
    //         _this.showDirect = null;
    //         this.pause();
    //       }, _this.queue[0].speed * 1000);
    //     }
    //   });
    //   // тут что то не так
    //   // setTimeout(() => {
    //   //   // _this.move = true;
    //   //   // _this.showDirect = _this.queue[0]?.direction;
    //   // });
    // },

    // startWork() {
    //   this.pushLift();
    // },

    workLocalStorage(name) {
      localStorage.setItem(name, JSON.stringify(this[name]));
    },
  },

  watch: {
    selected() {
      const freeLifts = this.lifts.filter(
        (el) => el.move === true && el.pause === false
      );
      console.log(freeLifts.length);
      if (freeLifts.length > 0) {
        const lastFloors = freeLifts.map((el) => {
          let value = el.floors[el.floors.length - 1];
          return value;
        });
        const closest = lastFloors.sort(
          (a, b) => Math.abs(this.selected - a) - Math.abs(this.selected - b)
        )[0];
        const actualLiftIndex = this.lifts.findIndex(
          (el) => el.floors[el.floors.length - 1] === closest
        );

        this.actualIndex = actualLiftIndex;
        console.log("ghjghjg");
        // let isNew = this.queueLifts.indexOf(
        //   (el) => el.id === this.lifts[actualLiftIndex].id
        // );
        // if (isNew !== -1) {
        //   this.actualIndex = isNew;
        // } else {
        //   this.queueLifts = [
        //     ...this.queueLifts,
        //     ...[this.lifts[actualLiftIndex]],
        //   ];
        //   this.actualIndex = this.queueLifts.length - 1;
        // }
      } else {
        // let isNew = this.queueLifts.indexOf((el) => el.id === this.lifts[0].id);
        // if (isNew !== -1) {
        //   this.actualIndex = isNew;
        // } else {
        //   this.queueLifts = [...this.queueLifts, ...[this.lifts[0]]];
        //   this.actualIndex = this.queueLifts.length - 1;
        // }
        this.actualIndex = 0;
      }

      // this.queueLifts[this.actualIndex].floors = [
      //   ...this.queueLifts[this.actualIndex].floors,
      //   ...[this.selected],
      // ];
      this.floors.push(this.selected);
      // this.workLocalStorage("floors");
    },

    // queueLifts() {
    //   // let _this = this.actualLift;
    //   let _this = this.queueLifts[this.actualIndex];
    //   let actualFloor = _this.floors[_this.floors.length - 1];
    //   let prevFloor = _this.floors[_this.floors.length - 2];

    //   if (actualFloor === prevFloor) return null;

    //   if (_this.floors.length === 1 && prevFloor === undefined) {
    //     if (actualFloor === _this.lastfloor) {
    //       _this.floors.shift();
    //       // this.workLocalStorage("floors");
    //       return null;
    //     }

    //     _this.queue.push({
    //       actualFloor: actualFloor,
    //       speed: Math.abs(actualFloor - _this.lastfloor),
    //       direction: actualFloor > _this.lastfloor ? "up" : "down",
    //       coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
    //     });
    //     // this.workLocalStorage("queue");
    //   } else {
    //     _this.queue.push({
    //       actualFloor: actualFloor,
    //       speed: Math.abs(actualFloor - prevFloor),
    //       direction: actualFloor > prevFloor ? "up" : "down",
    //       coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
    //     });
    //     // this.workLocalStorage("queue");
    //   }
    //   // this.watchLift = false;
    //   // console.log("queue увеличился");
    //   this.startWork();
    // },
  },
  // },
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

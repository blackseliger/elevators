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
        @deleteFloorQueue="deleteFloorQueue"
        @changeConfigPar="changeConfigPar"
        @changeQueueFloors="changeQueueFloors"
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
      liftsAmount: 3,
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
        floors: [0],
        pause: false,
        move: true,
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
      if (method === "push") {
        this.lifts[index].floors.push(value);
      } else if (method === "shift") {
        this.lifts[index].floors.shift();
      }
    },

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
      } else {
        this.actualIndex = 0;
      }

      this.floors.push(this.selected);
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

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
        @deleteSelected="deleteSelected"
      />
    </div>
    <div class="lifts__controls">
      <ui-button
        v-for="level in levels"
        :key="`${level}-button`"
        :value="level"
        v-model:selected="selected"
        :class="[
          parseInt(level) === actualButton ? 'controls__button_actual' : '',
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
      actualButton: 0,
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

    selectedButton() {
      return this.selected !== this.oldSelected ? this.selected : null;
    },
  },

  mounted() {
    // this.selected = JSON.parse(localStorage.getItem("selected")) || 0;
    // this.floors = JSON.parse(localStorage.getItem("floors")) || [];
  },

  methods: {
    deleteFloorQueue() {
      this.floors.shift();
      // localStorage.setItem("floors", JSON.stringify(this.floors));
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

    deleteSelected() {
      this.selected = null;
      // localStorage.setItem("selected", JSON.stringify(this.selected));
    },
  },

  watch: {
    selected(newvalue, oldValue) {

      if (newvalue === oldValue) {
        this.actualButton = null;
        return;
      }
      this.actualButton = newvalue;
      // localStorage.setItem("selected", JSON.stringify(this.selected));

      const freeLifts = this.lifts.filter(
        (el) => el.move === true && el.pause === false
      );
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
      // localStorage.setItem("floors", JSON.stringify(this.floors));
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

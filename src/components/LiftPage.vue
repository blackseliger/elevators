<!-- eslint-disable no-debugger -->
<template>
  <div class="lift">
    <div
      v-for="level in levels"
      :key="`${level}-lift`"
      :class="`lift__shaft_${level} lift__shaft`"
    >
      {{ level }}
    </div>

    <div
      class="lift__shaft_cabin lift__shaft"
      :class="{ lift__shaft_cabin_blinker: blink }"
      :style="classMove"
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
        v-show="showDirect === 'up' ? true : false"
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
        v-show="showDirect === 'down' ? true : false"
      />
    </div>

    <ui-button
      v-for="level in levels"
      :key="`${level}-button`"
      :value="level"
      v-model:selected="selected"
      :class="[
        parseInt(level) === queue[0]?.actualFloor ? 'lift__button_actual' : '',
        floors?.includes(parseInt(level)) ? 'lift__button_queue' : '',
        `lift__button_${level}`,
      ]"
      >{{ level }} этаж</ui-button
    >
  </div>
</template>

<script>
import UiButton from "@/components/UiButton.vue";
import UiIcon from "@/components/UiIcon.vue";

export default {
  name: "LiftPage",

  components: {
    UiButton,
    UiIcon,
  },

  props: {
    levels: {
      type: Array,
      required: true,
    },
  },

  data() {
    return {
      // стек этажей, первый зашел - первый вышел
      floors: [],
      // очередь
      queue: [],
      // на паузе по умолчанию до первого старта
      pause: { rest: false },
      // выбранный этаж
      selected: 0,
      // класс с опциями, для движения кабины
      classMove: null,
      // ожидаемые актуальные координаты прибытия кабины
      showDirect: null,
      // индекс сет интервала когда запускается работа лифта.
      indexStart: false,
      // находится ли в движении
      move: false,
      blink: false,
    };
  },

  mounted() {
    this.floors = JSON.parse(localStorage.getItem("floors")) || [];
    this.queue = JSON.parse(localStorage.getItem("queue")) || [];
    if (this.floors.length || this.queue.length) {
      this.startWork();
    }
  },

  methods: {
    // метод, который запускает движение лифта
    pushLift() {
      const cabin = this.$refs.cabin;
      let amountValuesH =
        this.levels.length * 100 + 6 + (this.levels.length - 1) * 5;
      let coords = Math.ceil(
        Math.ceil(parseFloat(getComputedStyle(cabin).top)) /
          (amountValuesH * 0.01)
      );

      this.classMove = {
        top: `${this.queue[0]?.coords}%`,
        transition: `top ${this.queue[0]?.speed}s ease-in-out`,
      };

      this.move = true;
      this.showDirect = this.queue[0]?.direction;

      if (this.pause.rest) return null;
      if (coords === this.queue[0]?.coords && this.move) {
        // не могу понять в чем баг если выбрать больше 2 этажей, то последний будет бликать 6 секунд. this.queue.shift(); первый раз срабатывает корректно
        // но сам this.queue для setTimeout не обновляется

        this.move = false;
        this.pause.rest = true;
        this.blink = true;
        // для стилизации кнопок которые в очереди, к ним еще лифт не идет
        setTimeout(() => {
          this.floors.shift();
          this.pause.rest = false;
          this.blink = false;
          this.queue.shift();
          this.workLocalStorage("floors");
          this.workLocalStorage("queue");
        }, 3000);
      }
    },

    startWork() {
      // каждую секунду вызывает метод для проверка
      // проверка на первый запуск, чтоб не запускался по новой регулярно
      if (this.indexStart === false) {
        this.indexStart = setInterval(() => {
          this.pushLift();
        }, 1000);
      }
    },

    workLocalStorage(name) {
      localStorage.setItem(name, JSON.stringify(this[name]));
    },
  },

  watch: {
    // выбирается актуальный вызов лифта и добавляется в стек "вызовов" этажей
    selected() {
      this.floors.push(this.selected);
      this.workLocalStorage("floors");
    },

    // здесь добавляется в очередь количество будущих поездок на этажи
    floors: {
      deep: true,
      handler() {
        let actualFloor = this.floors[this.floors.length - 1];
        let prevFloor = this.floors[this.floors.length - 2];
        // если этажи равны, то ничего не делаем;
        if (actualFloor === prevFloor) return null;
        // первый запуск, предыдущего этажа тут нет
        if (this.floors.length === 1 && prevFloor === undefined) {
          // проверка на первый случай, чтоб находясь на 1 этаже нельзя было выбрать 1 этаж
          if (actualFloor === 1) {
            this.floors.shift();
            this.workLocalStorage("floors");
            return null;
          }

          this.queue.push({
            actualFloor: actualFloor,
            speed: actualFloor === 0 ? 1 : actualFloor,
            // по умолчанию первый раз навверх лифт идет
            direction: "up",
            // ожидаемые координаты прибытия в процентах
            coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
          });
          this.workLocalStorage("queue");
        } else {
          this.queue.push({
            actualFloor: actualFloor,
            // расчет скорости для анимации, должен всегда 1 этаж в секунду
            // speed: Math.abs(actualFloor - (prevFloor + 1)),
            speed: Math.abs(actualFloor - prevFloor),

            // установка направления поездки
            direction: actualFloor > prevFloor ? "up" : "down",
            coords: 100 - Math.floor((actualFloor * 100) / this.levels.length),
          });
          this.workLocalStorage("queue");
        }

        // запускает работу
        this.startWork();
      },
    },
  },
};
</script>

<style lang="scss" scoped>
.lift {
  margin: auto;
  width: 205px;
  max-width: 205px;
  border: 1px solid black;
  padding: 2px;
  gap: 5px;
  position: relative;

  /*  grid  */
  display: grid;
  grid-template-columns: 100px 100px;
  grid-template-rows: repeat(5, 100px);
  justify-items: center;
  align-items: center;
  grid-template-areas:
    "shaft5 control5"
    "shaft4 control4"
    "shaft3 control3"
    "shaft2 control2"
    "shaft1 control1";

  &__shaft {
    grid-area: shaft;
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
    top: 80%;
    left: 1px;
    width: 100px;
    height: 100px;
    border: 1px solid #d34e4e;
  }

  &__shaft_cabin_blinker {
    animation: blinker 1s step-start infinite;
  }

  &__shaft_5 {
    grid-area: shaft5;
  }
  &__shaft_4 {
    grid-area: shaft4;
  }
  &__shaft_3 {
    grid-area: shaft3;
  }
  &__shaft_2 {
    grid-area: shaft2;
  }
  &__shaft_1 {
    grid-area: shaft1;
  }

  &__button {
    &_queue {
      background-color: rgba(54, 106, 136, 0.753);
    }
    &_actual {
      background-color: #4bc9e9;
    }
  }
  &__button_5 {
    grid-area: control5;
  }
  &__button_4 {
    grid-area: control4;
  }
  &__button_3 {
    grid-area: control3;
  }
  &__button_2 {
    grid-area: control2;
  }
  &__button_1 {
    grid-area: control1;
  }

  @keyframes blinker {
    50% {
      opacity: 0.5;
    }
    // придумать что то получше
  }
}
</style>


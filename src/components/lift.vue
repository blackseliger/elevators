<template>
  <div class="lift">
    <div class="lift__shaft_5 lift__shaft">5</div>
    <div class="lift__shaft_4 lift__shaft">4</div>
    <div class="lift__shaft_3 lift__shaft">3</div>
    <div class="lift__shaft_2 lift__shaft">2</div>
    <div class="lift__shaft_1 lift__shaft">1</div>
    <div class="lift__shaft_cabin lift__shaft" :style="classMove" ref="cabin">
      cabin
    </div>

    <ui-button :value="5" v-model:selected="selected" class="lift__button_5"
      >5 этаж</ui-button
    >
    <ui-button :value="4" v-model:selected="selected" class="lift__button_4"
      >4 этаж</ui-button
    >
    <ui-button :value="3" v-model:selected="selected" class="lift__button_3"
      >3 этаж</ui-button
    >
    <ui-button :value="2" v-model:selected="selected" class="lift__button_2"
      >2 этаж</ui-button
    >
    <ui-button :value="1" v-model:selected="selected" class="lift__button_1"
      >1 этаж</ui-button
    >
  </div>
</template>

<script>
import UiButton from "@/components/UiButton.vue";

// фактический параметр задания. В нашем случае работаем с 5 этажами.
const amountFloor = 5;
const amountValuesH = amountFloor * 100 + 6 + (amountFloor - 1) * 5;

export default {
  name: "LiftCommon",

  components: {
    UiButton,
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
      coords: null,
      // индекс сет интервала когда запускается работа лифта.
      indexStart: false,
      // находится ли в движении
      move: false,
    };
  },

  methods: {
    // метод, который запускает движение лифта
    pushLift() {
      const cabin = this.$refs.cabin;
      let coords = Math.ceil(
        Math.ceil(parseFloat(getComputedStyle(cabin).top)) /
          (amountValuesH * 0.01)
      );

      console.log(this.move, this.pause.rest, coords, this.queue[0]?.coords);

      this.classMove = {
        top: `${this.queue[0]?.coords}%`,
        transition: `top ${this.queue[0]?.speed}s ease-in-out`,
      };
      this.move = true;

      if (this.pause.rest) return null;

      if (coords === this.queue[0]?.coords && this.move) {
        this.move = false;
        this.pause.rest = true;

        setTimeout(() => {
          this.move = false;
          this.pause.rest = false;
          this.queue.shift();
        }, 3000);
      }
    },
  },

  watch: {
    // выбирается актуальный вызов лифта и добавляется в стек "вызовов" этажей
    selected() {
      this.floors.push(this.selected);
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
          this.queue.push({
            actualFloor: actualFloor,
            speed: actualFloor === 0 ? 1 : actualFloor,
            // по умолчанию первый раз навверх лифт идет
            direction: "up",
            // ожидаемые координаты прибытия в процентах
            coords: 100 - Math.floor((actualFloor * 100) / amountFloor),
          });
        } else {
          console.log("запустился второй случай");
          this.queue.push({
            actualFloor: actualFloor,
            // расчет скорости для анимации, должен всегда 1 этаж в секунду
            speed: Math.abs(actualFloor - (prevFloor + 1)),
            // установка направления поездки
            direction: actualFloor > prevFloor ? "up" : "down",
            coords: 100 - Math.floor((actualFloor * 100) / amountFloor),
          });
        }

        // запускает работу
        // проверка на первый запуск, чтоб не запускался по новой регулярно

        // вариант если дальше работать через метод
        // каждую секунду вызывает метод для проверка
        if (this.indexStart === false) {
          console.log("запустился метод");
          this.indexStart = setInterval(() => {
            this.pushLift();
            console.log("метод запускается каждую секунду");
          }, 1000);
        }
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
    // проверить динамически ли завит от количества этажей...
    /* const left */
    left: 1px;
    width: 100px;
    height: 100px;
    // transition: top 5s ease-in-out;
    border: 1px solid #d34e4e;
  }

  &__shaft_cabin_move {
    // top: calc(105px + 1px);
    // top: 60%;
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
    width: 50px;
    height: 50px;
    border: none;
    outline: none;
    background-color: rgb(223, 99, 240);
    cursor: pointer;
    transition: background 200ms ease-in-out;

    &:hover {
      background-color: rgb(140, 68, 223);
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
}
</style>


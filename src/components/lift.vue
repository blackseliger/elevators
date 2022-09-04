<template>
  <div class="lift">
    <div class="lift__shaft_5 lift__shaft">5</div>
    <div class="lift__shaft_4 lift__shaft">4</div>
    <div class="lift__shaft_3 lift__shaft">3</div>
    <div class="lift__shaft_2 lift__shaft">2</div>
    <div class="lift__shaft_1 lift__shaft">1</div>
    <div class="lift__shaft_cabin lift__shaft" ref="cabin">cabin</div>

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
// const amountValuesH = 5 * 100;
const amountFloor = 5;

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
      // проверка для computed длины queue
      queueLen: 0,
      // на паузе по умолчанию до первого старта
      pause: { timer: 3 },
      // выбранный этаж
      selected: 0,
      // класс с опциями, для движения кабины
      classMove: null,
      // ожидаемые актуальные координаты прибытия кабины
      coords: null,
      // индекс сет интервала когда запускается работа лифта.
      indexStart: null,
      // находится ли в движении
      move: false,
    };
  },

  // methods: {
  //   // метод, который запускает движение лифта
  //   pushLift() {
  //     if (!this.pause.timer >= 3 || !this.move ) return null;
  //     // проверка на отдых лифта
  //     const styleMove = {
  //       top: `${this.queue[0].coords}`,
  //     }
  //     // this.pause.timer = 0;
  //   },
  // },

  computed: {
    pushLift() {
      console.log('ghdkgjhd');
      // с проверками бедааааааа 
      // чтоб постоянно не перезапускался при удалении из очереди
      if (this.queue.length === this.queueLen) return;
      if (!this.pause.timer >= 3 || !this.move) return;

      const styleMove = {
        top: `${this.queue[0].coords}`,
      };
      // необходимо узнать когда кабина находится на этаже. Див блок над див блоком.
      // попробовать сравнить позиции через top 
      // или через сет таймаут попробовать на крайний случай, но он иногда подводит
      // this.$refs.cabin.styles.top
      console.log('pushLift');

      // if (this.queue[0].coords === )

      return styleMove;
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
      // immediate: true,
      handler() {
        let actualFloor = this.floors[this.floors.length - 1];
        let prevFloor = this.floors[this.floors.length - 2];
        console.log(actualFloor, prevFloor);
        // если этажи равны, то ничего не делаем;
        if (actualFloor === prevFloor) return null;
        // первый запуск, предыдущего этажа тут нет
        if (this.floors.length === 1 && prevFloor === undefined) {
          return this.queue.push({
            actualFloor: actualFloor,
            speed: actualFloor,
            arrive: false,
            // по умолчанию первый раз навверх лифт идет
            direction: "up",
            // ожидаемые координаты прибытия в процентах
            coords: Math.floor((actualFloor * 100) / amountFloor),
          });
        }

        this.queue.push({
          actualFloor: actualFloor,
          // расчет скорости для анимации, должен всегда 1 этаж в секунду
          speed: Math.abs(actualFloor - (prevFloor + 1)),
          arrive: false,
          // установка направления поездки
          direction: actualFloor > prevFloor ? "up" : "down",
          coords: Math.floor((actualFloor * 100) / amountFloor),
        });

        // this.queueLen = this.queue.length;

        // запускает работу
        // проверка на первый запуск, чтоб не запускался по новой регулярно

        // вариант если дальше работать через метод
        // if (!this.indexStart) {
        //   this.indexStart = setInterval(() => this.pushLift(), 1000);
        // }
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
    transition: top 5s ease-in-out;
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


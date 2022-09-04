<template>
  <div class="lift">
    <div class="lift__shaft_5 lift__shaft">5</div>
    <div class="lift__shaft_4 lift__shaft">4</div>
    <div class="lift__shaft_3 lift__shaft">3</div>
    <div class="lift__shaft_2 lift__shaft">4</div>
    <div class="lift__shaft_1 lift__shaft">5</div>
    <div class="lift__shaft_cabin lift__shaft lift__shaft_cabin_move">cabin</div>

    <button class="lift__button_5 lift__button">5#</button>
    <button class="lift__button_4 lift__button">4#</button>
    <button class="lift__button_3 lift__button">3#</button>
    <button class="lift__button_2 lift__button">2#</button>
    <button class="lift__button_1 lift__button">1#</button>
  </div>
</template>

<script>
export default {
  name: "LiftCommon",
  props: {
    msg: String,
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
    top: 1px;
    /* const left */
    left: 1px;
    width: 100px;
    height: 100px;
    transition: top 5s ease-in-out;
    border: 1px solid #d34e4e;
  }

  &__shaft_cabin_move {
    top: calc(105px + 1px);
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


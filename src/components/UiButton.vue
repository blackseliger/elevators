<template>
  <button class="button_default" @click="select()">
    <slot/>
  </button>
</template>

<script>
export default {
  name: "UiButton",

  props: {
    
    value: {
      type: String,
      required: true,
    },
  },

  emits: {
    'update:selected': null, 
  },

  methods: {
    select() {
        this.$emit('update:selected', this.value)
    }
  }
};
</script>

<style lang="scss" scoped>
.button_default {
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
</style>
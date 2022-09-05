<template>
  <div class="wrapper__icon">
  <img class="icon" :alt="icon" :src="iconSrc" />
  </div>
</template>

<script>
const icons = {
  arrow: require("@/assets/icons/arrow.svg"),
};

export default {
  name: "UiIcon",

  props: {
    icon: {
      type: String,
      required: true,
      validator: (name) => Object.keys(icons).includes(name),
    },
  },

  computed: {
    iconSrc() {
      return icons[this.icon];
    },
  },
};
</script>

<style scoped>
.wrapper__icon {
  display: inline-flex;
  width: 24px;
  height: 24px;

}


.icon {
    width: 100%;
    height: 100%;
    /* object-fit: contain; */
}
</style>

<template>
  <svg xmlns="http://www.w3.org/2000/svg" :width="width" :height="height" :fill="fill" viewBox="0 0 24 24"><path d="M16.59 8.59L12 13.17 7.41 8.59 6 10l6 6 6-6z"/><path d="M0 0h24v24H0z" fill="none"/></svg>
</template>

<script>
export default {
  name: 'IconExpandMore',
  props: {
    width: {
      type: Number,
      default: 24,
    },
    height: {
      type: Number,
      default: 24,
    },
    fill: {
      type: String,
      default: 'var(--dark)',
    },
  },
};
</script>

<template>
  <div class="loading-image" :style="`background-color: ${skeletonColor}`" />
</template>
<script>
export default {
  name: 'img-skeleton-loader',
  props: {
    skeletonColor: {
      type: String,
      default: '#cccccc',
      required: false,
    },
  },
  data() {
    return {}
  },
  methods: {},
  created() {},
}
</script>
<style scoped>
.loading-image {
  background-color: #cccccc;
  background: linear-gradient(
      100deg,
      rgba(255, 255, 255, 0) 40%,
      rgba(255, 255, 255, 0.5) 50%,
      rgba(255, 255, 255, 0) 60%
    )
    #cccccc;
  background-size: 200% 100%;
  background-position-x: 180%;
  animation: 1s loading ease-in-out infinite;
  border-radius: 10px;
  min-height: 100px;
  max-height: 400px;
  height: 20vh;
}

@media (min-width: 992px) {
  .loading-image {
    height: 400px;
  }
}

@keyframes loading {
  to {
    background-position-x: -20%;
  }
}
</style>

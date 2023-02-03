<script>
export default {
  name: "custom-button"
}
</script>

<template>
  <button class="btn">
    <slot></slot>
  </button>
</template>

<style scoped>
.btn{
  margin-top: 20px;
  width: 120px;
  height: 30px;
  border: none;
  background-color: transparent;
  color: rgb(90, 90, 90);
  font-weight: 700;
  font-size: 16px;
  text-align: center;
}
</style>
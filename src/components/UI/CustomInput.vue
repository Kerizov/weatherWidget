<script>
export default {
  name: "custom-input",
  props: {
    modelValue: String,
    placeholder: String,
  },
  methods: {
    updateInput(event){
      this.$emit('update:modelValue', event.target.value)
    }
  }
}
</script>

<template>
  <input
      type="text"
      class="input"
      :placeholder="$props.placeholder"
      :value="modelValue"
      @input="updateInput"
  >
</template>

<style scoped>
label{
  font-size: 15px;
  font-weight: bolder;
  margin-bottom: 5px;
}
.input{
  width: 180px;
  height: 30px;
  font-size: 16px;
  font-weight: bolder;
  padding: 10px;
  margin: 5px 0;
  border: none;
  box-shadow: 1px 1px 7px 1px rgba(83, 83, 83, 0.5);
}
</style>
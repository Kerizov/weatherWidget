<script>
import draggable from "vuedraggable";
export default {
  components: {
    draggable
  },
  props: {
    list: {
      type: Array
    }
  },
  methods: {
    getNewWeatherData(index){
      this.$emit('updateLocation', this.$props.list[index].location)
    },
    removeAt(idx) {
      this.list.splice(idx, 1);
      let Storage = JSON.parse(localStorage.getItem('locations'))
      Storage.splice(idx, 1);
      localStorage.setItem('locations', JSON.stringify(Storage))
    },
  }
};
</script>

<template>
  <div class="row">
    <div class="col-7">
      <draggable
          tag="ul"
          :list="$props.list"
          :animation="200"
          class="list-group"
          handle=".handle"
          item-key="name"
          ghost-class="moving-card"
      >
        <template #item="{ element, index }">
          <li class="list-group-item">
            <i class="handle">
              <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="17px" height="16px">
                <path fill-rule="evenodd"  fill="rgb(90, 90, 90)" d="M-0.000,-0.000 L17.000,-0.000 L17.000,2.000 L-0.000,2.000 L-0.000,-0.000 Z"/>
                <path fill-rule="evenodd"  fill="rgb(90, 90, 90)" d="M-0.000,7.000 L17.000,7.000 L17.000,9.000 L-0.000,9.000 L-0.000,7.000 Z"/>
                <path fill-rule="evenodd"  fill="rgb(90, 90, 90)" d="M-0.000,14.000 L17.000,14.000 L17.000,16.000 L-0.000,16.000 L-0.000,14.000 Z"/>
              </svg>
            </i>
            <button class="btn" @click="getNewWeatherData(index)">
              {{element.name}}
            </button>
            <i class="fa fa-times close" @click="removeAt(index)"><img style="width: 24px" src="@/assets/delete.svg" alt="delete"></i>
          </li>
        </template>
      </draggable>
    </div>
  </div>
</template>

<style scoped>
.btn{
  width: 120px;
  height: 25px;
  border: none;
  background-color: transparent;
  color: rgb(90, 90, 90);
  font-weight: 700;
  font-size: 16px;
  text-align: left;
}
.moving-card {
  opacity: 0.5;
  background: #F7FAFC;
  border: 1px solid #4299e1;
}
.list-group-item{
  width: 220px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 2px 10px;
  margin-bottom: 10px;
  background: white;
  border-radius: 6px;
  box-shadow: 1px 1px 7px rgba(83, 83, 83, 0.5);
}
.handle {
  cursor: move;
  float: left;
  padding-top: 8px;
  padding-bottom: 8px;
}
.close {
  float: right;
  padding-top: 8px;
  padding-bottom: 8px;
}
input {
  display: inline-block;
  width: 50%;
}
</style>
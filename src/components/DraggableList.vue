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
    // Removing current element from list and from localStorage
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
            <i class="fa fa-times close" @click="removeAt(index)"><svg xmlns="http://www.w3.org/2000/svg"  viewBox="0 0 50 50" width="24px" height="24px"><path d="M 21 2 C 19.354545 2 18 3.3545455 18 5 L 18 7 L 10.154297 7 A 1.0001 1.0001 0 0 0 9.984375 6.9863281 A 1.0001 1.0001 0 0 0 9.8398438 7 L 8 7 A 1.0001 1.0001 0 1 0 8 9 L 9 9 L 9 45 C 9 46.645455 10.354545 48 12 48 L 38 48 C 39.645455 48 41 46.645455 41 45 L 41 9 L 42 9 A 1.0001 1.0001 0 1 0 42 7 L 40.167969 7 A 1.0001 1.0001 0 0 0 39.841797 7 L 32 7 L 32 5 C 32 3.3545455 30.645455 2 29 2 L 21 2 z M 21 4 L 29 4 C 29.554545 4 30 4.4454545 30 5 L 30 7 L 20 7 L 20 5 C 20 4.4454545 20.445455 4 21 4 z M 11 9 L 18.832031 9 A 1.0001 1.0001 0 0 0 19.158203 9 L 30.832031 9 A 1.0001 1.0001 0 0 0 31.158203 9 L 39 9 L 39 45 C 39 45.554545 38.554545 46 38 46 L 12 46 C 11.445455 46 11 45.554545 11 45 L 11 9 z M 18.984375 13.986328 A 1.0001 1.0001 0 0 0 18 15 L 18 40 A 1.0001 1.0001 0 1 0 20 40 L 20 15 A 1.0001 1.0001 0 0 0 18.984375 13.986328 z M 24.984375 13.986328 A 1.0001 1.0001 0 0 0 24 15 L 24 40 A 1.0001 1.0001 0 1 0 26 40 L 26 15 A 1.0001 1.0001 0 0 0 24.984375 13.986328 z M 30.984375 13.986328 A 1.0001 1.0001 0 0 0 30 15 L 30 40 A 1.0001 1.0001 0 1 0 32 40 L 32 15 A 1.0001 1.0001 0 0 0 30.984375 13.986328 z"/></svg></i>
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
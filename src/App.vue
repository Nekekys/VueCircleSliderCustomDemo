<script setup>
  import CircleSlider from './components/CircleSlider.vue'
  import {ref} from "vue";

  const sliderValue = ref(0)

  const click = () => {
    sliderValue.value++
  }
</script>

<template>
  <div class="container">
    <div class="block">
      <h2>Simple</h2>
      <CircleSlider v-model:value="sliderValue" ></CircleSlider>
    </div>
    <div class="block">
      <h2>With control</h2>
      <CircleSlider v-model:value="sliderValue" ></CircleSlider>
      <input v-model="sliderValue" type="number">
      <button @click="click">+</button>
    </div>
    <div class="block">
      <h2>With slots</h2>
      <div class="row">
        <CircleSlider v-model:value="sliderValue" ><h2>{{ sliderValue }}</h2></CircleSlider>
        <CircleSlider v-model:value="sliderValue" ><h2>Text</h2></CircleSlider>
      </div>
    </div>
    <div class="block">
      <h2>Mirror</h2>
      <div class="row">
        <CircleSlider v-model:value="sliderValue" mirror >{{ sliderValue }}</CircleSlider>
        <CircleSlider v-model:value="sliderValue" >{{ sliderValue }}</CircleSlider>
      </div>
    </div>
  </div>
</template>
<style>
  .container {
    max-width: 1000px;
    margin: 0 auto;
  }
  .block{
    display: flex;
    align-items: center;
    flex-direction: column;
    justify-content: center;
    margin-top: 50px;
  }
  .row{
    display: flex;
    flex-direction: row;
  }
</style>

<template>
  <div id="about">
    <h1> About appstract. </h1>

    <b-container>
      <p>Important stuff</p>

      <p>
        You can unlock the following animals based on the number of points you have:
      </p>

      <b-container fluid class="p-4">
        <b-row>
          <b-col v-for="lev in levels" v-if="lev.img">
            <div >
              <b-img fluid class="pokemon" :src="lev.img_grey" alt="Thumbnail"/>
              <br>
              Level {{lev.level}}
              <br>
              <span>{{lev.min}} points</span>
            </div>
          </b-col>
        </b-row>
      </b-container>

    </b-container>

  </div>

</template>

<style>

.pokemon{
  min-width: 25px;
  min-height: 25px;
  width: 100px;
  height: 100px;
}


#about {
  min-height: 100vh;
}

</style>

<script>
export default {
  name: 'about',
  data() {
    return {
      count: 1,
    };
  },
  props: ['levels'],
};
</script>

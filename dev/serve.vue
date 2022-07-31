<script>
import Vue from 'vue';
// Uncomment import and local "components" registration if library is not registered globally.
// import { FantasySample } from '@/entry.esm';
// import { FantasyTest } from '@/entry';

export default Vue.extend({
  name: 'ServeDev',
  // components: {
  //  FantasySample,
  // }
});
</script>

<template>
  <div id="app">
    <fantasy-test 
      image="https://images.viblo.asia/a28b8126-1298-4ca7-a507-53865b32238d.png"
      :width="100"
      :height="100"
      :column="7"
      :row="4"
      :max="27"
      :frame="20"
      :loop="10"
    />
    <!-- <fantasy-sample /> -->
  </div>
</template>

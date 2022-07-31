<template>
  <div :style="style">
  </div>
</template>

<script>
export default {
  props: {
    image: String,
    height: Number,
    width: Number,
    loop: Number,
    frame: Number,
    max: Number,
    column: Number,
    row: Number,
  },
  mounted() {
    this.play()
  },
  data(){
    return {
      style: {
        width: this.width + 'px',
        height: this.height + 'px',
        background: `url(${this.image})`,
        backgroundSize: `${this.width * this.column}px ${this.height * this.row}px`,
        backgroundPosition: '0px 0px'
      }
    }
  },
  
  methods: {
    play() {
        var _this =this
      let i = 0
      let position = {
        x: 0,
        y: 0,
        loop: 0,
      }
      const playTimer = setInterval(() => {
        i++;
        if(i % this.column) {
          position.x -= this.width; 
        } else {
          position.y -= this.height; 
          position.x = 0;
        }
        if (i == this.max) {
          i = 0
          position.y = 0; 
          position.x = 0;
          position.loop++;
          if(position.loop >= this.loop) {
            clearInterval(playTimer)
          }
        }
        
        _this.$set(this.style, 'backgroundPosition', `${position.x}px ${position.y}px`)
      }, 1000/this.frame);
    }
  }
}
</script>
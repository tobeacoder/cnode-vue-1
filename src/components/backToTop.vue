<template lang="html">
<transition name="fade">
  <div id="back-to-top" v-show="show" @click="backToTop">
      <i :class="type"></i>
  </div>
  </transition>
</template>

<script>
export default {
    data() {
        return {
            type: "el-icon-arrow-up",
            show: false
        }
    },
    computed: {},
    created() {
        var self = this;
        if (window.addEventListener) {
            window.addEventListener("scroll", self.handleScroll, false);
        } else if (window.attachEvent) {
            window.attachEvent("onscroll", self.handleScroll);
        } else {
            window["onscroll"] = self.handleScroll;
        }
    },
    beforeDestory (){
        var self = this;
        if (window.removeEventListener) {
            window.removeEventListener("scroll", self.handleScroll, false);
        } else if (window.detachEvent) {
            window.detachEvent("onscroll", self.handleScroll);
        } else {
            window["onscroll"] = null;
        }
    },
    mounted() {},
    methods: {
        backToTop() {
            window.scrollTo(0, 0);
        },
        handleScroll (){
            this.show = (window.scrollY > 200) && true || false;
        }
    },
    components: {}
}
</script>

<style lang="sass">
#back-to-top {
    position: fixed;
    right: 30px;
    bottom: 30px;
    padding: 10px;
    background-color: #20a0ff;
    color: #fff;
    cursor: pointer;
    border-radius: 50%;
}
.fade-enter-active, .fade-leave-active {
    -webkit-transition: all 0.25s linear;
    -ms-transition: all 0.25s linear;
    -o-transition: all 0.25s linear;
    -moz-transition: all 0.25s linear;
    transition: all 0.25s linear;
}
.fade-enter, .fade-leave-active {
  opacity: 0;
  filter: Alpha(opacity=0);
}
</style>

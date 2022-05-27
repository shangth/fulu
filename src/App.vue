<template>
        <div id="app">
                <hello-world />
        </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";

export default {
        name: "app",
        components: {
                HelloWorld,
        },
};
</script>

<style>
@font-face {
     font-family: 'zf';
     src:  url('./assets/font.TTF');
   }
* {
        padding: 0;
        margin: 0;
}

</style>

<template>
<q-page class="container">
    <div class="acciones"></div>
    <div class="procesos"></div>
    <div class="cpu"></div>
    <div class="memoria"></div>
</q-page>
</template>

<script>

export default {
    name: 'PageIndex',
    setup(){
      document.title = 'Nuestro SO';
    }
}
</script>

<style lang="scss" scoped>
.container {
    width: 100%;
    height: 100vh;
    display: grid;
    grid-template-rows: 0.25fr 1fr 1fr 1fr;

    .procesos {
        background-color: rgb(102, 204, 204);
    }

    .cpu {
        background-color: rgb(254, 224, 102);
    }

    .memoria {
        background-color: rgb(163, 224, 102);
    }
}
</style>

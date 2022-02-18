<template>
<div id="cpu">

    <div class="scheduling">
        <div class="titulo">SCHEDULING</div>
        <div class="info">
            <p><span>Nombre: </span>1</p>
            <p><span>Tmpo Llegada: </span>1</p>
            <p><span>CPU Asignado: </span>1</p>
            <p><span>Envejecimiento: </span>1</p>
            <p><span>CPU Restante: </span>1</p>
            <p><span>Quantum Restante: </span>1</p>
        </div>
    </div>

    <div class="cpu-info">

        <div class="titulo">CPU</div>
        <div class="content">
            <q-select v-model="model" label="Algoritmo" :options="algoritmos" dense filled />
            <q-input v-model.number="quantum" label="Quantum" type="number" dense filled />
            <q-btn color="primary" icon="save" label="GUARDAR"/>
        </div>

    </div>

</div>
</template>

<script>
export default {
    name: 'Cpu',
    props: {
        procesoRunning: Object,
        algoritmos: Array,
        quantum: Number
    },
    data() {
        return {
            model: 0,
            quantum2: 0
        }
    },

}
</script>

<style lang="scss" scoped>
* {
    margin: 0;
    padding: 0;
}

#cpu {
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    gap: 0.7rem;
}

.scheduling, .cpu-info {
    padding: 1rem;
    border-radius: 5px;
    background-color: white;

    .titulo {
        font-size: 1.2rem;
    }

    .content{
      display: flex;
      flex-flow: column nowrap;
      gap:0.35rem;
    }
}

.scheduling{
  grid-column: 1/4;
}
.cpu-info{
  grid-column: 4/7;
}

span {
  text-decoration: underline;
}
</style>

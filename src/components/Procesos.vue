<template>
<div id="procesos">
    <div class="add-proceso">
        <p class="title">NEW</p>
        <div class="content">
            <q-input v-model.number="state.nombre" label="Nombre" dense type="number" filled />
            <q-input v-model.number="state.paginas" label="Páginas" dense type="number" filled />
            <q-input v-model.number="state.ejecTotal" label="Ejec Total" dense type="number" filled />

            <div class="botones">
                <q-btn color="primary" dense icon="add" @click="addProceso" />
                <q-btn color="primary" dense icon="upload" />
            </div>

        </div>
    </div>

    <q-table style="height:220px" title="READY" :rows="state.rows.ready" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense />
    <q-table style="height:220px" title="RUNNING" :rows="state.rows.running" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense />
    <q-table style="height:220px" title="BLOCKED" :rows="state.rows.blocked" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense />
    <q-table style="height:220px" title="FINISHED" :rows="state.rows.finished" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense />

</div>
</template>

<script>
import {
    reactive
} from '@vue/reactivity';
import { onMounted } from '@vue/runtime-core';
export default {
    name: 'Procesos',
    props: {
        procesos: Array,
        algoritmo: Number,
        interrupcion: Number
    },
    setup(props, ctx) {
        document.title = 'Nuestro SO';

        const columns = [{
            name: 'desc',
            required: true,
            label: null,
            align: 'left',
            field: row => row.id,
        }]

        const rows = {
            ready: props.procesos.filter(p => p.estado === 1),
            running: props.procesos.filter(p => p.estado === 2),
            blocked: props.procesos.filter(p => p.estado === 3),
            finished: props.procesos.filter(p => p.estado === 4)
        }

        const state = reactive({
            procesos: props.procesos,
            columns,
            rows,
            nombre: props.procesos.length + 1,
            paginas: 0,
            ejecTotal: 0
        });

        onMounted(() => {
            console.log(state.procesos);
        });

        function addProceso() {
            let proceso = {
                id: state.nombre,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: state.ejecTotal,
                quantum: 0,
                estado: 1
            };

            state.procesos.push(proceso);
            ctx.emit('procesoNuevo', proceso);
        }

        return {
            state,
            addProceso
        }

    },

}
</script>

<style lang="scss" scoped>
#procesos {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 1rem;

    .add-proceso {
        padding: 0.42rem 1rem;
        background-color: white;
        border-radius: 5px;
        max-height: 220px;

        .title {
            margin: 0;
            margin-bottom: 0.175rem;
            font-size: 20px;
        }

        .content {
            display: flex;
            flex-flow: column nowrap;
            gap: 0.35rem;

            .botones {
                display: grid;
                grid-template-columns: repeat(2, 1fr);
                gap: 0.35rem;
            }
        }
    }
}
</style>

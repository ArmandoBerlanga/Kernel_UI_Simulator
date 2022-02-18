<template>
<q-page class="container">
    <div class="acciones">
        <div class="tiempo-actual">
            <p> <span>TIEMPO ACTUAL: </span>{{ state.relojInterno }}</p>
            <q-btn color="primary" icon="play_arrow" label="EJECUTAR"/>
        </div>

       <q-select class="interrupcion" filled dense v-model="state.interrupcion" use-input input-debounce="0" label="Selecciona tu interrupción" :options="interrupciones" />

    </div>

    <div class="sec procesos">
        <div class="nombre">PROCESOS</div>
        <div class="content">
          <Procesos/>
        </div>
    </div>

    <div class="sec cpu">
        <div class="nombre">CPU</div>
        <div class="content">
          <CPU :algoritmos="algoritmos" :quantum="state.tamQuantum"/>
        </div>
    </div>

    <div class="sec memoria">
        <div class="nombre">MEMOIA</div>
        <div class="content"></div>
    </div>
</q-page>
</template>

<script>
import { reactive } from '@vue/reactivity';
import Procesos from 'components/Procesos.vue';
import CPU from 'components/Cpu.vue';

export default {
    name: 'PageIndex',
    components: {
        Procesos,
        CPU
    },
    setup() {
        document.title = 'Nuestro SO';

        const algoritmos = [
            { label: 'FIFO', value: 0 },
            { label: 'RR', value: 1 },
            { label: 'SRT', value: 2 },
            { label: 'HRRN', value: 3}
        ];

        const interrupciones = [
            { label: 'SVC de solicitud de I/O', value: 0 },
            { label: 'SVC de terminación normal', value: 1 },
            { label: 'SVC de solicitud de fecha', value: 2 },
            { label: 'Error de programa', value: 3 },
            { label: 'Externa de quantum expirado', value: 4 },
            { label: 'Dipositivo de I/O', value: 5 }
        ];

        // ESTRCUTURA DE UN PROCESO EN LA LISTA DE PROCESOS
        // 0 NEW, 1 READY, 2 RUNNING, 3 BLOCKED, 4 FINISHED
        // {
            // id: 1,
            // tiempoLlegada: 0,
            // cpuAsignado: 0,
            // envejecimiento: 0,
            // cpuRestante: 0,
            // quantum: 0,
            // estado: 0
        // }

        const state = reactive({
            algoritmo: 0,
            tamQuantum: 5,
            relojInterno: 0,
            interrupcion: null,
            procesos: [{
              id: 1,
              tiempoLlegada: 0,
              cpuAsignado: 10,
              envejecimiento: 11,
              cpuRestante: 2,
              quantum: 5,
              estado: 2
            }]
        });

        // NO APROPIATIVO
        function dispathFIFO() {

            let running = state.procesos.find(p => p.estado === 2);
            let position = state.procesos.findIndex(p => p.estado === 2);

            if(running.cpuRestante !== 0) {
                running.cpuRestante--;
                running.envejecimiento++; // ??
            }
            else {
                running.tiempoLlegada = 0;
                running.cpuAsignado = 0;
                running.envejecimiento = 0;
                running.cpuRestante = 0;
                running.estado = 4;
            }

            state.procesos[position] = running;
            return state.procesos
                    .find(p => p.estado === 1)
                    .sort((a, b) => a.tiempoLlegada - b.tiempoLlegada)[0];

        }

        // APROPIATIVO
        function dispathRR() {
        }

        // APROPIATIVO
        function dispathSRT() {
        }

        // APROPIATIVO
        function dispathHRRN() {
        }

        function dispatch() {
            state.relojInterno++;
            state.procesos.map(p => {
                if(p.estado === 1 || p.estado === 2)
                    p.tiempoLlegada++;
                return p;
            });

            if(state.algoritmo == 0)
                dispathFIFO();
            else if(state.algoritmo == 1)
                dispathRR();
            else if(state.algoritmo == 2)
                dispathSRT();
            else if(state.algoritmo == 3)
                dispathHRRN();
        }

        return {
            state,
            algoritmos,
            interrupciones,
            dispatch
        }
}
}
</script>

<style lang="scss" scoped>

*{
    margin: 0;
}
.container {
    width: 100%;
    height: 100vh;
    display: grid;
    grid-template-rows: 0.25fr auto auto 1fr;

    .acciones{
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0.35rem 1rem;

        .interrupcion{
            width: 25%;
        }
        .tiempo-actual{
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 1rem;
            p{
                font-size: 1.5rem;
                span{
                    font-weight: bold;
                }
            }
        }
    }

    .sec{
        padding: 0.35rem 1rem;
        display: grid;
        gap: 0.7rem;
        grid-template-columns: 0.1fr 1fr;

        .nombre{
            font-size: 1.2rem;
            font-weight: bold;
        }
    }

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

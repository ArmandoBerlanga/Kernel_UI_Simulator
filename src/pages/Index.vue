<template>
<q-page class="container">
    <div class="acciones"></div>
    <div class="procesos"></div>
    <div class="cpu"></div>
    <div class="memoria"></div>
</q-page>
</template>

<script>
import { reactive } from '@vue/reactivity';
export default {
    name: 'PageIndex',
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
        //     id: 1,
        //     tiempoLlegada: 0,
        //     cpuAsignado: 0,
        //     envejecimiento: 0,
        //     cpuRestante: 0,
        //     quantum: 0,
        //     estado: 0
        // }

        const state = reactive({
            algoritmo: 0,
            tamQuantum: 0,
            relojInterno: 0,
            interrupcion: 0,
            procesos: []
        });

        // APROPIATIVO
        function dispathFIFO() {

            let running = state.procesos.find(p => p.estado === 2);
            if(running.cpuRestante !== 0) {
                running.cpuRestante--;
                running.envejecimiento++;
                running.tiempoLlegada++;
                return running;
            }
            else {
                running.tiempoLlegada = null;
                running.cpuAsignado = 0;
                running.envejecimiento = 0;
                running.cpuRestante = 0;
                running.estado = 4;

                return state.procesos
                    .find(p => p.estado === 1)
                    .sort((a, b) => a.tiempoLlegada - b.tiempoLlegada)[0];
            }

        }

        // NO APROPIATIVO
        function dispathRR() {

            let running = state.procesos.find(p => p.estado === 2);
            if(running.quantum !== 0) {
                running.cpuRestante--;
                running.quantum--;
                running.envejecimiento++;
                return running;
            }
            else {

                if(running.cpuRestante !== 0){
                    running.quantum = state.tamQuantum;
                    running.envejecimiento++;
                }
                else {
                    running.tiempoLlegada = null;
                    running.cpuAsignado = 0;
                    running.envejecimiento = 0;
                    running.cpuRestante = 0;
                    running.quantum = 0;
                    running.estado = 4;

                    return state.procesos
                        .find(p => p.estado === 1)
                        .sort((a, b) => a.tiempoLlegada - b.tiempoLlegada)[0];
                }
            }

        }

        // APROPIATIVO
        function dispathSRT() {
        }

        // APROPIATIVO
        function dispathHRRN() {
        }

        function dispatch() {
            state.relojInterno++;
            state.procesos.forEach(p => {
                if(p.estado === 1)
                    p.tiempoLlegada++;
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

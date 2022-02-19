<template>
<q-page class="container">
    <div class="acciones">
        <div class="tiempo-actual">
            <p> <span>TIEMPO ACTUAL: </span>{{ state.relojInterno }}</p>
            <q-btn color="primary" icon="play_arrow" label="EJECUTAR" @click="dispatch" />
        </div>

        <q-select class="interrupcion" filled dense v-model="state.interrupcion" use-input input-debounce="0" label="Selecciona tu interrupción" :options="interrupciones" />

    </div>

    <div class="sec procesos">
        <div class="nombre">PROCESOS</div>
        <div class="content">

            <div id="procesos">
                <div class="add-proceso">
                    <p class="title">NEW</p>
                    <div class="content">
                        <q-input v-model.number="state.nuevo.nombre" label="Nombre" dense type="number" filled />
                        <q-input v-model.number="state.nuevo.paginas" label="Páginas" dense type="number" filled />
                        <q-input v-model.number="state.nuevo.ejecTotal" label="Ejec Total" dense type="number" filled />

                        <div class="botones">
                            <q-btn color="primary" dense icon="add" @click="addProceso" />
                            <q-btn color="primary" dense icon="upload" @click="addProcesos" />
                        </div>

                    </div>
                </div>

                <q-table style="height:220px" title="READY" :rows="state.rows.ready" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" />
                <q-table style="height:220px" title="RUNNING" :rows="state.rows.running" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" />
                <q-table style="height:220px" title="BLOCKED" :rows="state.rows.blocked" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" />
                <q-table style="height:220px" title="FINISHED" :rows="state.rows.finished" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" />

            </div>

        </div>
    </div>

    <div class="sec cpu">
        <div class="nombre">CPU</div>
        <div class="content">

            <div id="cpu">

                <div class="scheduling">
                    <div class="titulo">SCHEDULING</div>
                    <div class="info">
                        <p><span>Nombre: </span>{{ state.procesoRunning.id }}</p>
                        <p><span>Tmpo Llegada: </span>{{ state.procesoRunning.tiempoLlegada }}</p>
                        <p><span>CPU Asignado: </span>{{ state.procesoRunning.cpuAsignado }}</p>
                        <p><span>Envejecimiento: </span>{{ state.procesoRunning.envejecimiento }}</p>
                        <p><span>CPU Restante: </span>{{ state.procesoRunning.cpuRestante }}</p>
                        <p><span>Quantum Restante: </span>{{ state.procesoRunning.quantum }}</p>
                    </div>
                </div>

                <div class="cpu-info">

                    <div class="titulo">CPU</div>
                    <div class="content">
                        <q-select v-model="state.algoritmo" label="Algoritmo" :options="algoritmos" dense filled />
                        <q-input v-model.number="state.tamQuantum" label="Quantum" type="number" dense filled :disable="state.algoritmo.value == 0" />
                        <q-btn color="primary" icon="save" label="GUARDAR" />
                    </div>

                </div>

            </div>
        </div>
    </div>

    <div class="sec memoria">
        <div class="nombre">MEMORIA</div>
        <div class="content"></div>
    </div>
</q-page>
</template>

<script>
import {
    reactive
} from '@vue/reactivity';
import {
    defineComponent,
    onMounted
} from '@vue/runtime-core';

export default defineComponent({
    name: 'PageIndex',

    setup() {
        document.title = 'Nuestro SO';

        const algoritmos = [{
                label: 'FIFO',
                value: 0
            },
            {
                label: 'RR',
                value: 1
            },
            {
                label: 'SRT',
                value: 2
            },
            {
                label: 'HRRN',
                value: 3
            }
        ];

        const interrupciones = [{
                label: 'SVC de solicitud de I/O',
                value: 0
            },
            {
                label: 'SVC de terminación normal',
                value: 1
            },
            {
                label: 'SVC de solicitud de fecha',
                value: 2
            },
            {
                label: 'Error de programa',
                value: 3
            },
            {
                label: 'Externa de quantum expirado',
                value: 4
            },
            {
                label: 'Dipositivo de I/O',
                value: 5
            }
        ];

        const procesos = [{
                id: 1,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 1
            },
            {
                id: 2,
                tiempoLlegada: 7,
                cpuAsignado: 5,
                envejecimiento: 12,
                cpuRestante: 5,
                quantum: 0,
                estado: 2
            },
            {
                id: 3,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 3
            },
            {
                id: 4,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 4
            },
            {
                id: 5,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 1
            },
            {
                id: 6,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 4
            },
            {
                id: 7,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 3
            }
        ];

        let columns = [{
            name: 'desc',
            required: false,
            label: null,
            align: 'left',
            field: row => row.id,
            sortable: false,
        }]

        let rows = [];

        const state = reactive({
            procesoRunning: {
                id: 0,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 2
            },
            procesos: procesos,
            columns: columns,
            rows: rows,
            nuevo: {
                nombre: 0,
                paginas: 0,
                ejecTotal: 0
            },
            algoritmo: {
                value: 0,
                label: 'FIFO'
            },
            tamQuantum: 5,
            relojInterno: 0,
            interrupcion: null
        });

        onMounted(() => {
            state.procesoRunning = state.procesos.find(p => p.estado === 2);

            state.nuevo.nombre = state.procesos.length + 1;
            setProcesos();
        });

        function setProcesos() {
            state.rows = {
                ready: state.procesos.filter(p => p.estado === 1),
                running: state.procesos.filter(p => p.estado === 2),
                blocked: state.procesos.filter(p => p.estado === 3),
                finished: state.procesos.filter(p => p.estado === 4)
            }
        }

        function addProceso() {
            let proceso = {
                id: state.nuevo.nombre++,
                tiempoLlegada: state.relojInterno,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: state.nuevo.ejecTotal,
                quantum: state.tamQuantum,
                estado: 1
            };

            state.procesos.push(proceso);
            state.rows.ready.push(proceso);
        }

        function addProcesos() {
            // txt
        }

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

        // NO APROPIATIVO
        // SEGUN YO YA QUEDO ESTE, NADA MAS FALTA LO DE LOS IH
        function dispathFIFO() {

            let running = state.procesos.find(p => p.estado === 2);

            if (running && running.cpuRestante > 0) {
                running.cpuRestante--;
                running.envejecimiento++;
                running.cpuAsignado++;
            } else {
                running.cpuRestante = 0;
                running.envejecimiento = 0;
                running.cpuAsignado = 0;
                running.estado = 4;
                state.rows.running.pop();
                state.rows.finished.push(running);
            }

            if (state.procesos.find(p => p.estado === 2))
                return;

            let ready = state.procesos.filter(p => p.estado === 1)
                .sort((a, b) => a.tiempoLlegada - b.tiempoLlegada)[0];

            ready.estado = 2;
            state.procesoRunning = ready;
            state.rows.ready.splice(state.rows.ready.indexOf(ready), 1);
            state.rows.running.push(ready);
        }

        // APROPIATIVO
        function dispathRR() {}

        // APROPIATIVO
        function dispathSRT() {}

        // APROPIATIVO
        function dispathHRRN() {}

        function dispatch() {
            state.relojInterno++;
            state.procesos.map(p => {
                if (p.estado === 1 || p.estado === 2)
                    p.tiempoLlegada++;
                return p;
            });

            if (state.algoritmo.value == 0)
                dispathFIFO();
            else if (state.algoritmo.value == 1)
                dispathRR();
            else if (state.algoritmo.value == 2)
                dispathSRT();
            else if (state.algoritmo.value == 3)
                dispathHRRN();
        }

        return {
            state,
            algoritmos,
            interrupciones,
            dispatch,
            addProceso,
            addProcesos,
            pagination: {
                rowsPerPage: 0
            }
        }
    }
})
</script>

<style lang="scss" scoped>
@import 'src/css/procesos.scss';
@import 'src/css/cpu.scss';

* {
    margin: 0;
}

.container {
    width: 100%;
    height: 100vh;
    display: grid;
    grid-template-rows: 0.25fr auto auto 1fr;

    .acciones {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0.35rem 1rem;

        .interrupcion {
            width: 25%;
        }

        .tiempo-actual {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 1rem;

            p {
                font-size: 1.5rem;

                span {
                    font-weight: bold;
                }
            }
        }
    }

    .sec {
        padding: 0.35rem 1rem;
        display: grid;
        gap: 0.7rem;
        grid-template-columns: 0.1fr 1fr;

        .nombre {
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

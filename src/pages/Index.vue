<template>
<q-page class="container">
    <div class="acciones">
        <div class="tiempo-actual">
            <p> <span>TIEMPO ACTUAL: </span>{{ state.relojInterno }}</p>
            <q-btn color="primary" icon="play_arrow" label="EJECUTAR" @click="ejecutar" />
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
                            <q-btn id="btn" color="primary" dense icon="upload" />
                            <input id="input-file" type="file" ref="doc" @change="addProcesos" style="display: none;" />
                        </div>

                    </div>
                </div>

                <q-table style="height:220px" title="READY" :rows="state.rows.ready" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" no-data-label="Sin procesos" />
                <q-table style="height:220px" title="RUNNING" :rows="state.rows.running" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" no-data-label="Sin procesos" />
                <q-table style="height:220px" title="BLOCKED" :rows="state.rows.blocked" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" no-data-label="Sin procesos" />
                <q-table style="height:220px" title="FINISHED" :rows="state.rows.finished" :columns="state.columns" row-key="index" virtual-scroll hide-header hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" no-data-label="Sin procesos" />

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
                        <p><span>Tiempo Llegada: </span>{{ state.procesoRunning.tiempoLlegada }}</p>
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
                        <q-input v-model.number="state.tamQuantum" label="Quantum" type="number" dense filled :disable="state.algoritmo.value != 1" />
                        <q-btn color="primary" icon="save" label="GUARDAR" @click="guardar" />
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
    ref
} from "vue";
import {
    reactive
} from '@vue/reactivity';
import {
    useQuasar
} from "quasar";
import {
    defineComponent,
    onMounted
} from '@vue/runtime-core';

export default defineComponent({
    name: 'PageIndex',

    setup() {
        document.title = 'Nuestro SO';

        const $q = useQuasar();

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
            },
            {
                label: 'SJF',
                value: 4
            }
        ];

        const interrupciones = [{
                label: 'SVC de solicitud de I/O', // manda el proceso a blocked, pasa el sig proceso de ready (empieza contador de espera de blocked)
                value: 0
            },
            {
                label: 'SVC de terminación normal', // lo mando a finished, paso el sig proceso de ready
                value: 1 // APARECE AUTOMATICAMENTE, el usuario no puede solictar esta interrupcion
            },
            {
                label: 'SVC de solicitud de fecha', // lo mando a blocked, pasa el sig proceso de ready (empieza contador de espera de blocked)
                value: 2
            },
            {
                label: 'Error de programa', // lo mando a finished, paso el sig proceso de ready
                value: 3
            },
            {
                label: 'Externa de quantum expirado', // pasa a ready, pasa el sig proceso de ready
                value: 4 // APARECE AUTOMATICAMENTE, el usuario no puede solictar esta interrupcion
            },
            {
                label: 'Dipositivo de I/O', // si hay algo en blocked pasa el de blocked a ready, pasa el sig proceso de ready
                value: 5
            }
        ];

        const procesos = [{
                id: 1,
                tiempoLlegada: 6,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 4,
                quantum: 0,
                tiempoBlocked: 0,
                numPaginas: 20,
                estado: 1
            },
            {
                id: 2,
                tiempoLlegada: 5,
                cpuAsignado: 5,
                envejecimiento: 12,
                cpuRestante: 12,
                quantum: 0,
                tiempoBlocked: 0,
                numPaginas: 17,
                estado: 2
            },
            {
                id: 3,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 2,
                quantum: 0,
                tiempoBlocked: 0,
                numPaginas: 14,
                estado: 3
            },
            {
                id: 4,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 4,
                quantum: 0,
                tiempoBlocked: 0,
                numPaginas: 10,
                estado: 4
            },
            {
                id: 5,
                tiempoLlegada: 10,
                cpuAsignado: 0,
                envejecimiento: 10,
                cpuRestante: 6,
                quantum: 0,
                tiempoBlocked: 0,
                numPaginas: 8,
                estado: 1
            },
            {
                id: 6,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 5,
                quantum: 0,
                tiempoBlocked: 0,
                numPaginas: 3,
                estado: 4
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

        const doc = ref(null);
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
            rows: {
                ready: [],
                running: [],
                blocked: [],
                finished: []
            },
            nuevo: {
                nombre: 0,
                paginas: 0,
                ejecTotal: 0
            },
            algoritmo: {
                value: 0,
                label: 'FIFO'
            },
            tamQuantum: 0,
            relojInterno: 0,
            numPaginas: 0,
            interrupcion: null,
            file: null,
            contenidoFile: null
        });

        onMounted(() => {
            let input = document.getElementById("input-file");
            let btn = document.getElementById("btn");

            btn.onclick = () => input.click();

            state.procesoRunning = state.procesos.find(p => p.estado === 2);
            state.nuevo.nombre = state.procesos.length + 1;

            state.rows = {
                ready: state.procesos.filter(p => p.estado === 1).sort((a, b) => a.tiempoLlegada - b.tiempoLlegada),
                running: state.procesos.filter(p => p.estado === 2).sort((a, b) => a.tiempoLlegada - b.tiempoLlegada),
                blocked: state.procesos.filter(p => p.estado === 3).sort((a, b) => a.tiempoLlegada - b.tiempoLlegada),
                finished: state.procesos.filter(p => p.estado === 4).sort((a, b) => a.tiempoLlegada - b.tiempoLlegada)
            }
        });

        function addProceso() {
            let proceso = {
                id: state.nuevo.nombre++,
                tiempoLlegada: state.relojInterno,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: state.nuevo.ejecTotal,
                quantum: state.tamQuantum,
                tiempoBlocked: 0,
                numPaginas: state.nuevo.paginas,
                estado: 1
            };

            state.rows.ready.sort((a, b) => a.tiempoLlegada - b.tiempoLlegada);

            if (state.algoritmo.value === 2 && (state.procesoRunning.cpuRestante > proceso.cpuRestante)) { // veo si tiene menos cpuRestante

                state.rows.running.map(p => p.estado = 1);
                state.rows.ready.push(state.rows.running.pop());

                proceso.estado = 2;
                state.procesoRunning = proceso;
                state.rows.running.push(proceso);
                state.procesos.push(proceso);

                state.rows.ready.sort((a, b) => a.cpuRestante - b.cpuRestante);

            } else if (state.algoritmo.value === 3) {
                state.procesos.push(proceso);
                state.rows.ready.push(proceso);
                state.rows.ready.sort((a, b) => {
                    let prioridadA = (a.envejecimiento + (a.cpuAsignado + a.cpuRestante)) / (a.cpuAsignado + a.cpuRestante);
                    let prioridadB = (b.envejecimiento + (b.cpuAsignado + b.cpuRestante)) / (b.cpuAsignado + b.cpuRestante);
                    return prioridadB - prioridadA;
                });

            } else if (state.algoritmo.value === 4) {

                state.rows.running.map(p => p.estado = 1);
                state.rows.ready.push(state.rows.running.pop());

                proceso.estado = 2;
                state.procesoRunning = proceso;
                state.rows.running.push(proceso);
                state.procesos.push(proceso);

                state.rows.ready.sort((a, b) => a.numPaginas - b.numPaginas);

            } else {
                state.procesos.push(proceso);
                state.rows.ready.push(proceso);
            }
        }

        async function addProcesos() {
            state.nuevo.nombre = 1;
            state.procesos = [];
            state.rows.ready = [];
            state.rows.running = [];
            state.rows.blocked = [];
            state.rows.finished = [];

            state.file = doc.value.files[0];
            const reader = new FileReader();
            if (state.file.name.includes(".txt")) {
                reader.onload = async (res) => {
                    state.contenidoFile = res.target.result;

                    state.contenidoFile = state.contenidoFile.replaceAll("\n", ",");
                    state.contenidoFile = state.contenidoFile.split(",").map(c => parseInt(c));

                    state.numPaginas = state.contenidoFile[0];
                    state.relojInterno = state.contenidoFile[1];
                    for (let i = 3; i < state.contenidoFile.length; i++) {

                        let proceso = {
                            id: state.nuevo.nombre++,
                            tiempoLlegada: state.contenidoFile[i],
                            cpuAsignado: 0,
                            envejecimiento: 0,
                            cpuRestante: state.contenidoFile[i + 1],
                            quantum: state.tamQuantum,
                            tiempoBlocked: 0,
                            numPaginas: state.contenidoFile[i + 3],
                            estado: state.contenidoFile[i + 2]
                        }

                        state.procesos.push(proceso);

                        switch (proceso.estado) {
                            case 1:
                                state.rows.ready.push(proceso);
                                break;
                            case 2:
                                state.rows.running.push(proceso);
                                state.procesoRunning = proceso;
                                break;
                            case 3:
                                state.rows.blocked.push(proceso);
                                break;
                        }
                        let numPaginas = state.contenidoFile[i + 3];

                        i += (numPaginas * 6 + 3);
                    }

                    switch (state.algoritmo.value) {
                        case 0: // FIFO
                        case 1: // RR
                            state.rows.ready.sort((a, b) => a.tiempoLlegada - b.tiempoLlegada);
                            break;
                        case 2: // SRT
                            state.rows.ready.sort((a, b) => a.cpuRestante - b.cpuRestante);
                            break;
                        case 3: // HRRN
                            state.rows.ready.sort((a, b) => {
                                let prioridadA = (a.envejecimiento + (a.cpuAsignado + a.cpuRestante)) / (a.cpuAsignado + a.cpuRestante);
                                let prioridadB = (b.envejecimiento + (b.cpuAsignado + b.cpuRestante)) / (b.cpuAsignado + b.cpuRestante);
                                return prioridadB - prioridadA;
                            });
                            break;
                        case 4: // SJF
                            state.rows.ready.sort((a, b) => a.numPaginas - b.numPaginas);
                            break;
                    }

                };
                reader.onerror = (err) => console.log(err);
                reader.readAsText(state.file);
            } else {
                state.contenidoFile = "check the console for file output";
                reader.onload = (res) => {};
                reader.onerror = (err) => console.log(err);
                reader.readAsText(state.file);
            }

        }

        function dispatch() {
            let ready = state.rows.ready.filter(p => p.estado === 1)[0];
            if (!ready)
                return;

            ready.estado = 2;
            state.procesoRunning = ready;
            state.rows.ready.splice(state.rows.ready.indexOf(ready), 1);
            state.rows.running.push(ready);
        }

        // NO APROPIATIVO
        function ejecutarFIFO() {

            state.rows.ready.sort((a, b) => a.tiempoLlegada - b.tiempoLlegada);
            let running = state.rows.running.find(p => p.estado === 2);
            if (!running)
                return;

            if (running.cpuRestante > 1) {
                running.cpuRestante--;
                running.cpuAsignado++;
            } else {
                running.cpuRestante = 0;
                running.envejecimiento = 0;
                running.cpuAsignado = 0;
                running.estado = 4;
                state.rows.running.pop();
                state.rows.finished.push(running);
            }

            if (state.rows.running.find(p => p.estado === 2))
                return;

            dispatch();
        }

        // APROPIATIVO
        function ejecutarRR() {

            let running = state.rows.running.find(p => p.estado === 2);
            if (!running)
                return;

            if (running && running.quantum > 1 && running.cpuRestante > 1) {
                running.quantum--;
                running.cpuRestante--;
                running.cpuAsignado++;
            } else {
                if (running.cpuRestante <= 1) {
                    running.cpuRestante = 0;
                    running.quantum = 0;
                    running.envejecimiento = 0;
                    running.cpuAsignado = 0;
                    running.estado = 4;
                    state.rows.running.pop();
                    state.rows.finished.push(running);
                } else {
                    running.cpuRestante--;
                    running.quantum = state.tamQuantum;
                    running.estado = 1;
                    state.rows.running.pop();
                    state.rows.ready.push(running);
                }
            }

            // console.log(running.quantum, running.cpuRestante);
            if (state.rows.running.find(p => p.estado === 2))
                return;

            dispatch();
        }

        // NO APROPIATIVO
        function ejecutarSRT() {

            // console.log(state.rows.ready);
            // ejec total - ejec actual
            state.rows.ready.sort((a, b) => a.cpuRestante - b.cpuRestante);

            let running = state.rows.running.find(p => p.estado === 2);
            if (!running)
                return;

            if (running && running.cpuRestante > 1) {
                running.cpuRestante--;
                running.cpuAsignado++;
            } else {
                running.cpuRestante = 0;
                running.envejecimiento = 0;
                running.cpuAsignado = 0;
                running.estado = 4;
                state.rows.running.pop();
                state.rows.finished.push(running);
                // console.log(state.rows.ready);
            }

            if (state.rows.running.find(p => p.estado === 2))
                return;

            dispatch();
        }

        // NO APROPIATIVO
        function ejecutarHRRN() {

            // prioridad = (tiempoEspera + tiempoServicio / tiempoServicio)
            // prioridad = (envejecimiento + (cpuAsignado + cpuRestante)) / (cpuAsignado + cpuRestante))
            state.rows.ready.sort((a, b) => {
                let prioridadA = (a.envejecimiento + (a.cpuAsignado + a.cpuRestante)) / (a.cpuAsignado + a.cpuRestante);
                let prioridadB = (b.envejecimiento + (b.cpuAsignado + b.cpuRestante)) / (b.cpuAsignado + b.cpuRestante);
                return prioridadB - prioridadA;
            });

            console.log(state.rows.ready.flatMap(p => (p.envejecimiento + (p.cpuAsignado + p.cpuRestante)) / (p.cpuAsignado + p.cpuRestante)));

            let running = state.rows.running.find(p => p.estado === 2);
            if (!running)
                return;

            if (running && running.cpuRestante > 1) {
                running.cpuRestante--;
                running.cpuAsignado++;
            } else {
                running.cpuRestante = 0;
                running.envejecimiento = 0;
                running.cpuAsignado = 0;
                running.estado = 4;
                state.rows.running.pop();
                state.rows.finished.push(running);
            }

            if (state.rows.running.find(p => p.estado === 2))
                return;

            dispatch();
        }

        // NO APROPIATIVO
        function ejecutarSJF() {

            state.rows.ready.sort((a, b) => a.numPaginas - b.numPaginas);

            let running = state.rows.running.find(p => p.estado === 2);
            if (!running)
                return;

            if (running && running.cpuRestante > 1) {
                running.cpuRestante--;
                running.cpuAsignado++;
            } else {
                running.cpuRestante = 0;
                running.envejecimiento = 0;
                running.cpuAsignado = 0;
                running.estado = 4;
                state.rows.running.pop();
                state.rows.finished.push(running);
                // console.log(state.rows.ready);
            }

            if (state.rows.running.find(p => p.estado === 2))
                return;

            dispatch();

        }

        // INTERRUPCIONES

        function intrDispositivoIO(id) {

            let running = state.rows.running.find(p => p.estado === 2);

            if (running && running.cpuRestante > 1) {
                state.rows.running.map(p => p.estado = 1);
                state.rows.ready.push(state.rows.running.pop());
            } else if (running) {
                state.rows.running.map(p => p.estado = 4);
                state.rows.finished.push(state.rows.running.pop());
            }

            state.rows.blocked.map(p => {
                if (p.id == id) {
                    p.estado = 1
                    p.tiempoBlocked = 0;
                }
                return p;
            });

            state.rows.ready.push(state.rows.blocked.shift());

            dispatch();
        }

        function intrSVCdeSolicitudIO() {
            // manda el proceso a blocked, pasa el sig proceso de ready (empieza contador de espera de blocked)
            state.rows.running.map(p => p.estado = 3);
            state.rows.blocked.push(state.rows.running.pop());

            // console.log(ready);

            switch (state.algoritmo.value) {
                case 2:
                    state.rows.ready.sort((a, b) => a.cpuRestante - b.cpuRestante);
                    break;
                case 3:
                    state.rows.ready.sort((a, b) => {
                        let prioridadA = (a.envejecimiento + (a.cpuAsignado + a.cpuRestante)) / (a.cpuAsignado + a.cpuRestante);
                        let prioridadB = (b.envejecimiento + (b.cpuAsignado + b.cpuRestante)) / (b.cpuAsignado + b.cpuRestante);
                        return prioridadB - prioridadA;
                    });
                    break;
            }

            dispatch();
        }

        function intrTerminacionNormal() {

            state.rows.running.map(p => p.estado = 4);
            state.rows.finished.push(state.rows.running.pop());

            // console.log(ready);

            switch (state.algoritmo.value) {
                case 2:
                    state.rows.ready.sort((a, b) => a.cpuRestante - b.cpuRestante);
                    break;
                case 3:
                    state.rows.ready.sort((a, b) => {
                        let prioridadA = (a.envejecimiento + (a.cpuAsignado + a.cpuRestante)) / (a.cpuAsignado + a.cpuRestante);
                        let prioridadB = (b.envejecimiento + (b.cpuAsignado + b.cpuRestante)) / (b.cpuAsignado + b.cpuRestante);
                        return prioridadB - prioridadA;
                    });
                    break;
            }

            dispatch();
        }

        function intrQuantumExpirado() {
            // pasa a ready, pasa el sig proceso de ready

            state.rows.running.map(p => p.estado = 1);
            state.rows.ready.push(state.rows.running.pop());

            // console.log(ready);

            switch (state.algoritmo.value) {
                case 2:
                    state.rows.ready.sort((a, b) => a.cpuRestante - b.cpuRestante);
                    break;
                case 3:
                    state.rows.ready.sort((a, b) => {
                        let prioridadA = (a.envejecimiento + (a.cpuAsignado + a.cpuRestante)) / (a.cpuAsignado + a.cpuRestante);
                        let prioridadB = (b.envejecimiento + (b.cpuAsignado + b.cpuRestante)) / (b.cpuAsignado + b.cpuRestante);
                        return prioridadB - prioridadA;
                    });
                    break;
            }

            dispatch();
        }

        function intrErrorDePrograma() {
            intrTerminacionNormal();

            $q.notify({
                message: 'Error de programa',
                type: 'danger',
                position: 'top',
                color: 'red'
            });
        }

        function intrSVCdeSolicitudDeFecha() {
            intrSVCdeSolicitudIO();
        }

        function ejecutar() {
            state.relojInterno++;
            state.procesos.map(p => {
                if (p.estado === 1) {
                    p.envejecimiento++;
                    p.quantum = state.tamQuantum;
                }
                if (p.estado === 3)
                    p.tiempoBlocked++;
                return p;
            });

            let bool = true;
            for (let i = 0; i < state.rows.blocked.length; i++)
                if (state.rows.blocked[i].tiempoBlocked >= 5) {
                    intrDispositivoIO(state.rows.blocked[i].id);
                    bool = false;
                    break;
                }

            if (bool) {
                if (state.interrupcion != null && (state.rows.running[0] || state.rows.blocked)) {
                    switch (state.interrupcion.value) {
                        case 0:
                            intrSVCdeSolicitudIO(); //DONE
                            break;
                        case 1:
                            intrTerminacionNormal();
                            break;
                        case 2:
                            intrSVCdeSolicitudDeFecha(); //done
                            break;
                        case 3:
                            intrErrorDePrograma();
                            break;
                        case 4:
                            intrQuantumExpirado();
                            break;
                        case 5:
                            intrDispositivoIO(state.rows.blocked[0].id); // DONE
                            break;
                    }

                } else {
                    switch (state.algoritmo.value) {
                        case 0:
                            ejecutarFIFO();
                            break;
                        case 1:
                            ejecutarRR();
                            break;
                        case 2:
                            ejecutarSRT();
                            break;
                        case 3:
                            ejecutarHRRN();
                            break;
                        case 4:
                            ejecutarSJF();
                            break;
                    }
                }
            }
            state.interrupcion = null;
        }

        function guardar() {
            if (state.algoritmo.value == 1)
                state.procesos.map(p => {
                    p.quantum = state.tamQuantum;
                    return p;
                });
        }

        return {
            state,
            algoritmos,
            interrupciones,
            ejecutar,
            guardar,
            addProceso,
            addProcesos,
            doc,
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

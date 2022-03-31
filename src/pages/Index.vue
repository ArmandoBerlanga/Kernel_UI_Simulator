<template>
<q-page class="container">
    <div class="acciones">
        <div class="tiempo-actual">
            <p> <span>TIEMPO ACTUAL: </span>{{ state.relojInterno }}</p>
            <q-select class="interrupcion" filled dense v-model="state.ejecutarPagina" use-input input-debounce="0" label="PÁGINA" :options="state.procesoRunning.paginas.flatMap(p => p.index)" />
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
                            <q-btn color="primary" dense icon="add" @click="addProceso" :disabled="state.procesos.filter(p=>p.estado!=4).length==10" />
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
        <div class="content">

            <div id="memoria">

                <div class="tabla-memoria">
                    <q-table style="height:180px" :rows="state.procesoRunning.paginas" :columns="state.columnsMemoria" row-key="index" virtual-scroll hide-pagination dense :pagination="pagination" :rows-per-page-options="[0]" no-data-label="Sin páginas" />
                </div>

                <div class="memoria-info">

                    <div class="titulo">MEMORIA</div>
                    <div class="content">
                        <q-select v-model="state.algoritmoMemoria" label="Algoritmo" :options="algoritmosMemoria" dense filled />
                        <q-btn color="primary" icon="save" label="GUARDAR" @click="guardarMemoria" />
                        <q-btn color="primary" label="RESET BITS NUR" @click="resetearBits" />
                    </div>

                </div>

            </div>

        </div>
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
        document.title = 'Kernelsito';

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

        const algoritmosMemoria = [{
                label: 'FIFO',
                value: 0
            },
            {
                label: 'LRU',
                value: 1
            },
            {
                label: 'LFU',
                value: 2
            },
            {
                label: 'NUR',
                value: 3
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
                estado: 1,
                paginas: [{
                        index: 0,
                        bitResidencia: 1,
                        llegada: 11,
                        ultAcceso: 8,
                        accesos: 13000,
                        bitLectura: 0,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 0,
                        llegada: 12,
                        ultAcceso: 9,
                        accesos: 10,
                        bitLectura: 1,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 15,
                        ultAcceso: 12,
                        accesos: 4,
                        bitLectura: 0,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 0,
                        llegada: 1,
                        ultAcceso: 1100,
                        accesos: 4,
                        bitLectura: 0,
                        bitModificacion: 0,
                        contAccesos: 0
                    }
                ]
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
                estado: 2,
                paginas: [{
                        index: 0,
                        bitResidencia: 1,
                        llegada: 11,
                        ultAcceso: 8,
                        accesos: 13,
                        bitLectura: 1,
                        bitModificacion: 1,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 12,
                        ultAcceso: 9,
                        accesos: 10,
                        bitLectura: 1,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 10,
                        ultAcceso: 12,
                        accesos: 4,
                        bitLectura: 0,
                        bitModificacion: 1,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 0,
                        llegada: 1,
                        ultAcceso: 8,
                        accesos: 13,
                        bitLectura: 1,
                        bitModificacion: 1,
                        contAccesos: 0
                    },
                ]
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
                estado: 3,
                paginas: [{
                        index: 0,
                        bitResidencia: 1,
                        llegada: 1,
                        ultAcceso: 8,
                        accesos: 13,
                        bitLectura: 1,
                        bitModificacion: 1,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 2,
                        ultAcceso: 9,
                        accesos: 10,
                        bitLectura: 1,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 10,
                        ultAcceso: 12,
                        accesos: 4,
                        bitLectura: 0,
                        bitModificacion: 0,
                        contAccesos: 0
                    }
                ]
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
                estado: 4,
                paginas: [{
                        index: 0,
                        bitResidencia: 1,
                        llegada: 1,
                        ultAcceso: 8,
                        accesos: 13,
                        bitLectura: 1,
                        bitModificacion: 1,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 2,
                        ultAcceso: 9,
                        accesos: 10,
                        bitLectura: 1,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 10,
                        ultAcceso: 12,
                        accesos: 4,
                        bitLectura: 0,
                        bitModificacion: 0,
                        contAccesos: 0
                    }
                ]
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
                estado: 1,
                paginas: [{
                        index: 0,
                        bitResidencia: 1,
                        llegada: 1,
                        ultAcceso: 8,
                        accesos: 13,
                        bitLectura: 1,
                        bitModificacion: 1,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 2,
                        ultAcceso: 9,
                        accesos: 10,
                        bitLectura: 1,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 10,
                        ultAcceso: 12,
                        accesos: 4,
                        bitLectura: 0,
                        bitModificacion: 0,
                        contAccesos: 0
                    }
                ]
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
                estado: 4,
                paginas: [{
                        index: 0,
                        bitResidencia: 1,
                        llegada: 1,
                        ultAcceso: 8,
                        accesos: 13,
                        bitLectura: 1,
                        bitModificacion: 1,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 2,
                        ultAcceso: 9,
                        accesos: 10,
                        bitLectura: 1,
                        bitModificacion: 0,
                        contAccesos: 0
                    },
                    {
                        index: 0,
                        bitResidencia: 1,
                        llegada: 10,
                        ultAcceso: 12,
                        accesos: 4,
                        bitLectura: 0,
                        bitModificacion: 0,
                        contAccesos: 0
                    }
                ]
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

        let columnsMemoria = [{
                name: 'index',
                required: false,
                label: 'num pag',
                align: 'center',
                field: row => row.index,
                sortable: false,
            },
            {
                name: 'bit r',
                required: false,
                label: 'bit r',
                align: 'center',
                field: row => row.bitResidencia,
                sortable: false,
            },
            {
                name: 'llegada',
                required: false,
                label: 'tmpo llegada',
                align: 'center',
                field: row => row.llegada,
                sortable: false,
            },
            {
                name: 'ult acceso',
                required: false,
                label: 'ult acceso',
                align: 'center',
                field: row => row.ultAcceso,
                sortable: false,
            },
            {
                name: 'accesos',
                required: false,
                label: 'accesos',
                align: 'center',
                field: row => row.accesos,
                sortable: false,
            },
            {
                name: 'NUR',
                required: false,
                label: 'NUR',
                align: 'center',
                field: row => row.bitLectura + ' ' + row.bitModificacion,
                sortable: false,
            }
        ]

        const doc = ref(null);
        const state = reactive({
            procesoRunning: {
                id: 0,
                tiempoLlegada: 0,
                cpuAsignado: 0,
                envejecimiento: 0,
                cpuRestante: 0,
                quantum: 0,
                estado: 2,
                paginas: []
            },
            ejecutarPagina: 0,
            procesos: procesos,
            columns: columns,
            columnsMemoria: columnsMemoria,
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
            algoritmoMemoria: {
                value: 0,
                label: 'FIFO'
            },
            tamQuantum: 0,
            relojInterno: 0,
            numPaginas: 3,
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

            state.procesos.forEach(p => p.paginas.forEach(pag => pag.index = p.paginas.indexOf(pag)))
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
                estado: 1,
                paginas: []
            };

            for (let i = 0; i < state.nuevo.paginas; i++)
                proceso.paginas.push({
                    index: i,
                    bitResidencia: 0,
                    llegada: state.relojInterno,
                    ultAcceso: 0,
                    accesos: 0,
                    bitLectura: 0,
                    bitModificacion: 0,
                    contAccesos: 0
                })

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
                            cpuRestante: state.contenidoFile[++i],
                            quantum: state.tamQuantum,
                            tiempoBlocked: 0,
                            estado: state.contenidoFile[++i],
                            numPaginas: state.contenidoFile[++i],
                            paginas: []
                        }

                        for (let j = 0; j < proceso.numPaginas; j++)
                            proceso.paginas.push({
                                index: j,
                                bitResidencia: state.contenidoFile[++i],
                                llegada: state.contenidoFile[++i],
                                ultAcceso: state.contenidoFile[++i],
                                accesos: state.contenidoFile[++i],
                                bitLectura: state.contenidoFile[++i],
                                bitModificacion: state.contenidoFile[++i],
                                contAccesos: 0
                            })

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

                    // console.log(state.procesoRunning);

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

        function cambiarAtributosPaginas(index) {
            //AL PROCESO QUE SALE
            state.procesoRunning.paginas.find(p => p.index == index).bitResidencia = 0;

            //AL PROCESO QUE ENTRA
            state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitResidencia = 1;
            //Poner nuevo tiempo de llegada
            state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).llegada = state.relojInterno;
            //Poner el resto a 0
            state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).accesos = 0;
            state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).ultAcceso = 0;
            state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitLectura = 0;
            state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitModificacion = 0;
        }



        // ALGORITMOS DE MEMORIA

        function ejecutarMemoriaFIFO() {
            let paginas = state.procesoRunning.paginas
                .filter(p => p.bitResidencia == 1)
                .sort((a, b) => a.llegada - b.llegada);

            cambiarAtributosPaginas(paginas[0].index);
        }

        function ejecutarMemoriaLRU() {
            let paginas = state.procesoRunning.paginas
                .filter(p => p.bitResidencia == 1)
                .sort((a, b) => a.ultAcceso - b.ultAcceso);

            cambiarAtributosPaginas(paginas[0].index);
        }

        function ejecutarMemoriaLFU() {
            let paginas = state.procesoRunning.paginas
                .filter(p => p.bitResidencia == 1)
                .sort((a, b) => a.accesos - b.accesos);

            cambiarAtributosPaginas(paginas[0].index);
        }

        function ejecutarMemoriaNUR() {

            let paginas = state.procesoRunning.paginas
                .filter(p => p.bitResidencia == 1)
                .sort((a, b) => {
                    let sumaA = a.bitLectura + a.bitModificacion * 2;
                    let sumaB = b.bitLectura + b.bitModificacion * 2;
                    return sumaA - sumaB;
                });

            cambiarAtributosPaginas(paginas[0].index);
        }

        function ejecutarPagina() {
            let paginasUsadas = state.procesoRunning.paginas.filter(p => p.bitResidencia == 1).length;
            let paginaActiva = state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitResidencia == 0;
            let falloPagina = state.numPaginas == paginasUsadas && paginaActiva;
            if (falloPagina)
                switch (state.algoritmoMemoria.value) {
                    case 0:
                        ejecutarMemoriaFIFO();
                        break;
                    case 1:
                        ejecutarMemoriaLRU();
                        break;
                    case 2:
                        ejecutarMemoriaLFU();
                        break;
                    case 3:
                        ejecutarMemoriaNUR();
                        break;
                }
            else
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitResidencia = 1;

            //Incrementar número de accesos
            //Actualizar último acceso
            //Actualizar bit de lectura
            if(!falloPagina) {
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).accesos++;
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).ultAcceso = state.relojInterno;
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitLectura = 1;
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).contAccesos++;
            }

            //Actualizar bit de escritura cuando el número de accesos llegue a 5
            if (state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).contAccesos == 5) {
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitModificacion = 1;
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).contAccesos = 0;
            }

            if (!falloPagina && paginaActiva) {
                //AL PROCESO QUE ENTRA
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitResidencia = 1;
                //Poner nuevo tiempo de llegada
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).llegada = state.relojInterno;
                //Poner el resto a 0
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).accesos = 0;
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).ultAcceso = 0;
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitLectura = 0;
                state.procesoRunning.paginas.find(p => p.index == state.ejecutarPagina).bitModificacion = 0;
            }

            if (paginaActiva) {
                $q.notify({
                    message: 'Fallo de página',
                    type: 'warning',
                    position: 'top',
                    color: 'warning'
                });
                intrSVCdeSolicitudIO();
            }

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

                    ejecutarPagina();
                }
            }
            state.interrupcion = null;
        }

        function resetearBits() {
            state.procesos.forEach(proceso => {
                proceso.paginas.map(p => {
                    p.bitModificacion = 0;
                    p.bitLectura = 0;
                    return p;
                });
            });
        }

        function guardar() {
            if (state.algoritmo.value == 1)
                state.procesos.map(p => {
                    p.quantum = state.tamQuantum;
                    return p;
                });
        }

        function guardarMemoria() {
            console.log(state.algoritmoMemoria.value);
        }

        return {
            state,
            algoritmos,
            algoritmosMemoria,
            interrupciones,
            ejecutar,
            resetearBits,
            guardar,
            guardarMemoria,
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
@import 'src/css/memoria.scss';

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

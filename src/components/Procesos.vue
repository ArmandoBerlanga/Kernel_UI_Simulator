<template>
<div id="procesos">
    <div class="add-proceso">
        <p class="title">NEW</p>
        <div class="content">
            <q-input v-model.number="model" label="Nombre" dense type="number" filled />
            <q-input v-model.number="model" label="Páginas" dense type="number" filled />
            <q-input v-model.number="model" label="Ejec Total" dense type="number" filled />

            <div class="botones">
                <q-btn color="primary" dense icon="add" />
                <q-btn color="primary" dense icon="upload" />
            </div>
        </div>
    </div>

    <q-table style="height:220px" title="READY" :rows="rows" :columns="columns" row-key="index" virtual-scroll hide-header hide-pagination dense v-model:pagination="pagination" :rows-per-page-options="[0]" />
    <q-table style="height:220px" title="RUNNING" :rows="rows" :columns="columns" row-key="index" virtual-scroll hide-header hide-pagination dense v-model:pagination="pagination" :rows-per-page-options="[0]" />
    <q-table style="height:220px" title="BLOCKED" :rows="rows" :columns="columns" row-key="index" virtual-scroll hide-header hide-pagination dense v-model:pagination="pagination" :rows-per-page-options="[0]" />
    <q-table style="height:220px" title="FINISHED" :rows="rows" :columns="columns" row-key="index" virtual-scroll hide-header hide-pagination dense v-model:pagination="pagination" :rows-per-page-options="[0]" />

</div>
</template>

<script>
export default {
    name: 'Procesos',
    props: {
        procesos: Array,
        algoritmo: Number,
        interrupcion: Number
    },
    setup() {
        document.title = 'Nuestro SO';

        const columns = [{
            name: 'desc',
            required: true,
            label: null,
            align: 'left',
            field: row => row.name,
        }]

        const rows = [{
                name: 1
            }, {
                name: 2
            }, {
                name: 3
            }, {
                name: 4
            }, {
                name: 5
            }

        ]

        return {
            columns,
            rows
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

<script setup lang="ts">
import type { Serie } from '../../models/serie'
import http from '../../plugins/axios'
import Button from 'primevue/button'
import Dialog from 'primevue/dialog'
import InputText from 'primevue/inputtext'
import InputNumber from 'primevue/inputnumber'
import DatePicker from 'primevue/datepicker';
import { computed, ref, watch } from 'vue'
import Select from 'primevue/select'

const ENDPOINT = 'series'
const props = defineProps({
    mostrar: Boolean,
    serie: {
        type: Object as () => Serie,
        default: () => ({}) as Serie
    },
    modoEdicion: Boolean
})
const emit = defineEmits(['guardar', 'close'])

const dialogVisible = computed({
    get: () => props.mostrar,
    set: (value) => {
        if (!value) emit('close')
    }
})


const clasificaciones = [
    { label: 'A: Todo público', value: 'A: Todo público' },
    { label: 'B: Para niños', value: 'B: Para niños' },
    { label: 'B12: Público mayor a 12 años', value: 'B12: Público mayor a 12 años' },
    { label: 'B15: Público mayor a 15 años', value: 'B15: Público mayor a 15 años' },
    { label: 'C: Público mayor a 18 años', value: 'C: Público mayor a 18 años' }
]

const serie = ref<Serie>({ ...props.serie })
watch(
    () => props.serie,
    (newVal) => {
        serie.value = { ...newVal }
    }
)

async function handleSave() {
    try {
        const body = {
            titulo: serie.value.titulo,
            sinopsis: serie.value.sinopsis,
            tipoClasificacion: serie.value.tipoClasificacion,
            director: serie.value.director,
            temporadas: serie.value.temporadas,
            fechaEstreno: serie.value.fechaEstreno,
        }
        if (props.modoEdicion) {
            await http.patch(`${ENDPOINT}/${serie.value.id}`, body)
        } else {
            await http.post(ENDPOINT, body)
        }
        emit('guardar')
        serie.value = {} as Serie
        dialogVisible.value = false
    } catch (error: any) {
        alert(error?.response?.data?.message)
    }
}
</script>

<template>
    <div class="card flex justify-center">
        <Dialog v-model:visible="dialogVisible" :header="props.modoEdicion ? 'Editar' : 'Crear'" style="width: 25rem">
            <div class="flex items-center gap-4 mb-4">
                <label for="titulo" class="font-semibold w-24">Título</label>
                <InputText id="titulo" v-model="serie.titulo" class="flex-auto" autocomplete="off" autofocus />
            </div>
            <div class="flex items-center gap-4 mb-4">
                <label for="sinopsis" class="font-semibold w-24">Sinopsis</label>
                <InputText id="sinopsis" v-model="serie.sinopsis" class="flex-auto" autocomplete="off" />
            </div>
            <div class="flex items-center gap-4 mb-4">
                <label for="tipoClasificaciones" class="font-semibold w-24">Clasificaciones</label>
                <Select id="tipoClasificaciones" v-model="serie.tipoClasificacion" :options="clasificaciones"
                    optionLabel="label" optionValue="value" class="flex-auto" />
            </div>
            <div class="flex items-center gap-4 mb-4">
                <label for="director" class="font-semibold w-24">Director</label>
                <InputText id="director" v-model="serie.director" class="flex-auto" autocomplete="off" />
            </div>
            <div class="flex items-center gap-4 mb-4">
                <label for="temporadas" class="font-semibold w-24">Temporadas</label>
                <InputNumber id="temporadas" v-model="serie.temporadas" class="flex-auto" autocomplete="off" />
            </div>
            <div class="flex items-center gap-4 mb-4">
                <label for="fechaEstreno" class="font-semibold w-24">Fecha de Estreno</label>
                <DatePicker id="fechaEstreno" v-model="serie.fechaEstreno" class="flex-auto" autocomplete="off"
                    :showIcon="true" />
            </div>
            <div class="flex justify-end gap-2">
                <Button type="button" label="Cancelar" icon="pi pi-times" severity="secondary"
                    @click="dialogVisible = false"></Button>
                <Button type="button" label="Guardar" icon="pi pi-save" @click="handleSave"></Button>
            </div>
        </Dialog>
    </div>
</template>

<style scoped></style>
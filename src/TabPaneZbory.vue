<template>
    <div class="tab-pane fade">
        <div class="container">
            <div class=" my-3 alert alert-info">
                <FontAwesomeIcon :icon="faCircleExclamation" class="me-2"/>
                Limity zborów określają ile identyfikatorów parkingowych może użyć każdy zbór, żeby umożliwić wjazd na parking dla niepełnosprawnych
            </div>

            <div v-if="show_error" class=" mb-3 alert alert-danger">
                <FontAwesomeIcon :icon="faCircleExclamation" class="me-2"/>
                Zapis nowej wartości limitu zakończył się niepowodzeniem, spróbuj ponownie.
            </div>

            <div class="mb-3 d-flex justify-content-center">
                <div class="form-check me-3">
                    <input class="form-check-input" type="radio" value="ALL" id="zb_tura_all" v-model="tura">
                    <label class="form-check-label" for="zb_tura_all">
                        Wszystko
                    </label>
                </div>
                <div v-for="t in props.tury" :key="t.tid" class="form-check me-3" :title="t.name">
                    <input class="form-check-input" type="radio" :value="t.tid" :id="`zb_tura_${t.tid}`" v-model="tura">
                    <label class="form-check-label" :for="`zb_tura_${t.tid}`">
                        {{ t.shortcut }}
                    </label>
                </div>
            </div>

            <table class="table table-striped table-hover">
                <thead>
                    <tr>
                        <th scope="col">Nazwa zboru</th>
                        <th scope="col">Limit identyfikatorów</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(zbor, index) in filtered_zbory" :key="index">
                        <td colspan="2">
                            <div class="d-flex align-items-center">
                                <span class="flex-grow-1">{{ zbor.name }}</span>
                                <input
                                    class="form-control text-center w-auto"
                                    type="number"
                                    :maxlength="2"
                                    :min="0"
                                    :max="99"
                                    v-model="zbor.plimit"
                                >
                                <button
                                    class="btn btn-outline-primary ms-2"
                                    @click="on_Save(zbor)"
                                >
                                    <FontAwesomeIcon :icon="faSave" />
                                </button>
                            </div>
                            <div v-if="zbor.limitRequest" class="mt-2 p-2 alert alert-warning mb-0 d-flex flex-wrap column-gap-4">
                                <div><strong>Nowa wartość limitu:</strong> {{ zbor.limitRequest.plimit }}</div>
                                <div><strong>Uzasadnienie:</strong> {{ zbor.limitRequest.reason }}</div>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faCircleExclamation, faSave } from '@fortawesome/free-solid-svg-icons';

const props = defineProps(['tury'])
const show_error = ref(false)
const zbory = ref([])
const tura = ref('ALL')
const filtered_zbory = computed(() => zbory.value.filter((elem) => tura.value === 'ALL' || elem.tura === tura.value))

onMounted(loadLimits)

async function loadLimits() {
    const resp = await fetch("/api/limits/zbory")
    if(resp.status === 200) {
        let tmp = await resp.json()
        tmp.sort((a,b) => a.name.localeCompare(b.name))
        zbory.value = tmp
    }
}

async function on_Save(zbor) {
    const resp = await fetch("/api/limits/zbory/setlimit", {
        method: "POST",
        headers: { "Content-Type": "application/json", },
        body: JSON.stringify({
            congregation_id: zbor.id,
            plimit: zbor.plimit,
        })
    })
    if(resp.status !== 200) {
        show_error.value = true
        let timer = setTimeout(() => {
            clearTimeout(timer)
            show_error.value = false
        }, 5000)
    }
    await loadLimits()
}
</script>
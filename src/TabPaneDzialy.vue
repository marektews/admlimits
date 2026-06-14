<template>
    <div class="tab-pane fade">
        <div class="container">
            <div class=" my-3 alert alert-info">
                <FontAwesomeIcon :icon="faCircleExclamation" class="me-2"/>
                Limity działów określają ile identyfikatorów parkingowych może użyć każdy z działów kongresowy
            </div>

            <div v-if="show_error" class=" mb-3 alert alert-danger">
                <FontAwesomeIcon :icon="faCircleExclamation" class="me-2"/>
                Zapis nowej wartości limitu zakończył się niepowodzeniem, spróbuj ponownie.
            </div>

            <div class="mb-3 d-flex justify-content-center">
                <div class="form-check me-3">
                    <input class="form-check-input" type="radio" value="ALL" id="dz_tura_all" v-model="tura">
                    <label class="form-check-label" for="dz_tura_all">
                        Wszystko
                    </label>
                </div>
                <div v-for="t in props.tury" :key="t.tid" class="form-check me-3" :title="t.name">
                    <input class="form-check-input" type="radio" :value="t.tid" :id="`dz_tura_${t.tid}`" v-model="tura">
                    <label class="form-check-label" :for="`dz_tura_${t.tid}`">
                        {{ t.shortcut }}
                    </label>
                </div>
            </div>

            <table class="table table-striped table-hover">
                <thead>
                    <tr>
                        <th scope="col">Nazwa działu</th>
                        <th scope="col">Limit identyfikatorów</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(dzial, index) in filtered_dzialy" :key="index">
                        <td>{{ dzial.name }}</td>
                        <td class="d-flex">
                            <input 
                                class="form-control text-center" 
                                type="number" 
                                :maxlength="2" 
                                :min="0" 
                                :max="99"
                                v-model="dzial.plimit"
                            >
                            <button 
                                class="btn btn-outline-primary ms-2"
                                @click="on_Save(dzial)"
                            >
                                <FontAwesomeIcon :icon="faSave" />
                            </button>
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
const dzialy = ref([])
const tura = ref('ALL')
const filtered_dzialy = computed(() => dzialy.value.filter((elem) => tura.value === 'ALL' || elem.tura_id === tura.value))

onMounted(loadLimits)

async function loadLimits() {
    const resp = await fetch("/api/limits/dzialy")
    if(resp.status === 200) {
        let tmp = await resp.json()
        tmp.sort((a,b) => a.name.localeCompare(b.name))
        dzialy.value = tmp
    }
}

async function on_Save(dzial) {
    const resp = await fetch("/api/limits/dzialy/setlimit", {
        method: "POST",
        headers: { "Content-Type": "application/json", },
        body: JSON.stringify({
            dzial_id: dzial.id,
            plimit: dzial.plimit,
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
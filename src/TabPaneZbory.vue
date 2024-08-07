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
                    <input class="form-check-input" type="radio" :value="2" id="w2" v-model="tura">
                    <label class="form-check-label" for="w2">
                        W2
                    </label>
                </div>
                <div class="form-check">
                    <input class="form-check-input" type="radio" :value="3" id="w3" v-model="tura">
                    <label class="form-check-label" for="w3">
                        W3
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
                    <tr v-for="(zbor, index) in filtered_zbory" :key="index">
                        <td>{{ zbor.name }}</td>
                        <td class="d-flex">
                            <input 
                                class="form-control text-center" 
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

const show_error = ref(false)
const zbory = ref([])
const tura = ref(3)
const filtered_zbory = computed(() => zbory.value.filter((elem) => elem.tura === tura.value))

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
            zbor_id: zbor.id,
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
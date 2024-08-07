<template>
    <div class="container">
        <h4>Limity - tryb moderatora</h4>

        <ul class="mt-4 nav nav-tabs" role="tablist">
            <button class="nav-link active" data-bs-toggle="tab" data-bs-target="#dzialy">
                Działy
            </button>
            <button class="nav-link" data-bs-toggle="tab" data-bs-target="#zbory">
                Zbory
            </button>
        </ul>

        <div class="tab-content" id="tabContent">
            <TabPaneDzialy class="show active" id="dzialy" />
            <TabPaneZbory id="zbory" />
        </div>
    </div>
</template>

<script setup>
import TabPaneDzialy from './TabPaneDzialy.vue';
import TabPaneZbory from './TabPaneZbory.vue';

</script>

<style>
#app {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    margin-top: 16pt;
}
</style>
